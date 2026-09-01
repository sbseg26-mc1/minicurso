# Prompts de classificação de incidentes (EC2)

Gabaritos usados no `02-classificacao.ipynb`. Os marcadores `{texto_do_ticket}`,
`{categorias}` e `{exemplos}` são substituídos em tempo de execução pelas
funções `montar_prompt_livre`, `montar_prompt_zero_shot` e
`montar_prompt_com_exemplos`.

A taxonomia é a das doze categorias funcionais derivadas do NIST SP 800-61r3,
apresentada na Tabela 1.2 do capítulo e definida em código na variável
`TAXONOMIA`.

Todos os três prompts pedem resposta em **JSON**, com as chaves `categoria` e
`justificativa`. A extração usa regex sobre o texto retornado
(`extrair_categoria` / `extrair_categoria_livre`), então o formato de saída
não é imposto por *structured output* da API — é convenção de prompt.

---

## A. Categorização livre (linha de base)

Serve para medir a **dispersão** de rótulos: quantos nomes distintos o modelo
inventa para o mesmo fenômeno, sem taxonomia. É o argumento empírico a favor
de vocabulário controlado (ver Seção 6.3 do *notebook*, "Dispersão
semântica").

```text
Você é um analista de SOC. Leia o ticket abaixo e diga, com suas próprias
palavras, que tipo de incidente de segurança é esse — não fornecemos nenhuma
lista de categorias.

Ticket:
"""
{texto_do_ticket}
"""

Responda em formato JSON, apenas com as chaves "categoria" (um rótulo curto,
2 a 4 palavras) e "justificativa" (uma frase curta).
```

O rótulo livre é extraído do JSON e normalizado (minúsculas, sem espaços nas
pontas) apenas para a contagem de rótulos distintos; o valor bruto é mantido
na coluna `rotulo_livre` do CSV de resultados.

---

## B. Orientada por taxonomia, zero-shot

```text
Você é um analista de SOC especialista em categorização de incidentes de
segurança.

Classifique o ticket abaixo em UMA das categorias a seguir, segundo o NIST SP
800-61 Rev. 3:

- CAT1: Comprometimento de Conta — acesso não autorizado a contas de usuários
  ou administradores
- CAT2: Malware — infecção por código malicioso que compromete dispositivos
  ou dados
- CAT3: Negação de Serviço (DoS/DDoS) — indisponibilidade de sistemas ou
  redes
- CAT4: Exfiltração/Vazamento de Dados — acesso, cópia ou divulgação não
  autorizada de dados sensíveis
- CAT5: Exploração de Vulnerabilidade — uso de falhas conhecidas ou
  desconhecidas para comprometer ativos
- CAT6: Abuso Interno — ações intencionais ou negligentes de usuários
  internos
- CAT7: Engenharia Social — engano de pessoas para obter acesso ou
  informações
- CAT8: Incidente Físico ou de Infraestrutura — violação física que impacta
  ativos computacionais
- CAT9: Alteração Não Autorizada — modificação não autorizada em sistemas,
  dados ou configurações
- CAT10: Uso Indevido de Recursos — uso não autorizado de sistemas para
  outros fins
- CAT11: Problema de Fornecedor/Terceiro — incidente originado por falha de
  segurança de terceiros
- CAT12: Tentativa de Intrusão — tentativas hostis de invasão ainda não
  confirmadas como bem-sucedidas

Ticket:
"""
{texto_do_ticket}
"""

Responda em formato JSON, apenas com as chaves "categoria" (o código, ex:
CAT1) e "justificativa" (uma frase curta).
```

A lista de categorias é gerada a partir do dicionário `TAXONOMIA` (uma linha
`- CODIGO: descrição` por entrada); o bloco acima reflete o texto exato que
`montar_prompt_zero_shot` produz.

---

## C. Orientada por taxonomia, one-shot

Mesmo cabeçalho e lista de categorias do prompt B, com um bloco de exemplos
inserido antes do ticket a classificar.

**Cuidado metodológico obrigatório:** os *tickets* usados como exemplo
precisam ser removidos do conjunto de avaliação, sob pena de contaminação. O
*notebook* faz essa exclusão explicitamente na Seção 2, com quatro exemplos
fixos (um por categoria escolhida com propósito didático: `T017`→CAT5,
`T023`→CAT3, `T010`→CAT12, `T022`→CAT4) e não uma amostragem aleatória.

```text
Você é um analista de SOC especialista em categorização de incidentes de
segurança.

Classifique o ticket em UMA das categorias a seguir, segundo o NIST SP
800-61 Rev. 3:

[... mesma lista de categorias do prompt B ...]

Veja alguns exemplos já classificados:

{exemplos}

Agora classifique o novo ticket:
"""
{texto_do_ticket}
"""

Responda em formato JSON, apenas com as chaves "categoria" (o código, ex:
CAT1) e "justificativa" (uma frase curta).
```

Formato de cada exemplo (os primeiros 400 caracteres do ticket, seguidos de
reticências, e a resposta no mesmo formato JSON exigido do modelo):

```text
Ticket: "<primeiros 400 caracteres do ticket>..."
Resposta: {"categoria": "CAT5"}
```

---

## Nota sobre parâmetros de inferência

O *notebook* chama `client.models.generate_content(model=MODEL_NAME,
contents=prompt)` sem definir `temperature` — a chamada usa o valor padrão da
API do Gemini, não `temperature = 0`. Isso é uma mudança em relação a
versões anteriores deste material: se a reprodutibilidade entre execuções for
importante para a aula, considere fixar a temperatura explicitamente na
função `classificar`.

O modelo usado é o **Gemini** (`gemini-3.1-flash-lite` por padrão, via
`google-genai`), configurável pela variável de ambiente
`GOOGLE_GENERATIVE_MODEL`. Diferente do EC1 (que roda localmente e sem rede),
o EC2 depende de uma API key (`GOOGLE_GENERATIVE_AI_API_KEY`) e de acesso à
internet — vale registrar essa diferença de pressuposto operacional ao
apresentar os dois estudos de caso em sequência.
