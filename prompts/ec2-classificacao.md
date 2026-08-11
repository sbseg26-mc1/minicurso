# Prompts de classificação de incidentes (EC2)

Gabaritos usados no `02-classificacao.ipynb`. Os marcadores `{texto_do_ticket}`
e `{exemplos}` são substituídos em tempo de execução.

A taxonomia é a das doze categorias funcionais derivadas do NIST SP 800-61r3,
apresentada na Tabela 1.2 do capítulo.

---

## A. Categorização livre (linha de base)

Serve para medir a **dispersão** de rótulos: quantos nomes distintos o modelo
inventa para o mesmo fenômeno. É o argumento empírico a favor de vocabulário
controlado.

```text
Voce e um classificador de incidentes de ciberseguranca.
Analise o relato abaixo e responda APENAS com a categoria do incidente.

INCIDENTE:
{texto_do_ticket}
```

---

## B. Orientada por taxonomia, zero-shot

```text
Voce e um classificador de incidentes de ciberseguranca.
Classifique o relato abaixo em EXATAMENTE UMA das categorias:

CAT1  Comprometimento de Conta       CAT7  Engenharia Social
CAT2  Malware                        CAT8  Incidente Fisico
CAT3  Negacao de Servico (DoS/DDoS)  CAT9  Alteracao Nao Autorizada
CAT4  Exfiltracao ou Vazamento       CAT10 Uso Indevido de Recursos
CAT5  Exploracao de Vulnerabilidade  CAT11 Problema de Terceiro
CAT6  Abuso Interno                  CAT12 Tentativa de Intrusao

Responda APENAS com o codigo da categoria (ex.: CAT5).
Se houver mais de uma categoria plausivel, escolha a de maior prioridade.

INCIDENTE:
{texto_do_ticket}
```

---

## C. Orientada por taxonomia, one-shot

Idêntica à anterior, com um bloco de exemplos inserido antes do incidente a
classificar. **Cuidado metodológico obrigatório:** os *tickets* usados como
exemplo precisam ser removidos do conjunto de avaliação, sob pena de
contaminação. O *notebook* faz essa exclusão explicitamente.

```text
[... mesmo cabeçalho e lista de categorias do prompt B ...]

EXEMPLOS ROTULADOS:
{exemplos}

Agora classifique o incidente abaixo.

INCIDENTE:
{texto_do_ticket}
```

Formato de cada exemplo:

```text
--- Exemplo ---
INCIDENTE: <trecho do ticket>
CATEGORIA: CAT5
```

---

## D. Progressive-Hint Prompting (PHP)

Heurística iterativa avaliada em Severo et al. (2025). O ciclo encerra quando
o contador de dicas se esgota ou quando duas respostas sucessivas coincidem.
Na implementação de referência (SecLINC), a estabilização é medida por ROUGE
com limiar 0,9; no *notebook*, por simplicidade didática, a condição de parada
é a igualdade do código de categoria em duas rodadas consecutivas.

Primeira rodada: prompt B.

Rodadas seguintes:

```text
[... mesmo cabeçalho e lista de categorias do prompt B ...]

DICA: uma analise anterior deste mesmo incidente sugeriu a categoria
{resposta_anterior}. Reavalie criticamente. Se a dica estiver correta,
confirme-a; se estiver errada, corrija-a.

Responda APENAS com o codigo da categoria.

INCIDENTE:
{texto_do_ticket}
```

---

## Nota sobre parâmetros de inferência

Para classificação, use `temperature = 0`. A evidência de Pohlmann et al.
(2025) indica que a temperatura tem influência limitada sobre a precisão nesta
tarefa, mas valores baixos reduzem a variabilidade entre execuções, que é o
problema real em ambiente operacional.
