# Prompt de geração de playbook (EC3)

Gabarito usado no `03-playbooks.ipynb`. Os marcadores `{categoria}`,
`{texto_do_ticket}` e `{contexto_recuperado}` são substituídos em tempo de
execução.

## Por que as restrições importam

As cinco restrições abaixo não são estilo: cada uma fecha um risco documentado
na Seção 1.7.6 do capítulo.

| Restrição | Risco que fecha |
|---|---|
| 1. Vocabulário fechado de ações | saída não validável automaticamente |
| 2. Proibição de inventar artefatos | alucinação |
| 3. Alteração em produção sempre `manual` | ação insegura executada sem aprovação |
| 4. Coletar evidência em vez de assumir valor ausente | preenchimento plausível de lacuna |
| 5. Apenas JSON válido | erro de forma confundido com erro de conteúdo |

---

## Prompt de geração (formato CACAO 2.0)

```text
Voce e um assistente de resposta a incidentes de um CSIRT academico.
Gere um playbook de resposta para o incidente abaixo.

RESTRICOES OBRIGATORIAS
1. Use APENAS acoes das seguintes classes: verificar, coletar_evidencia,
   conter, notificar, escalar, documentar.
2. NAO invente nomes de ferramentas, hosts, comandos, caminhos de arquivo ou
   politicas que nao aparecam no incidente, no contexto recuperado ou na
   lista de ferramentas disponiveis.
3. Toda acao que altere configuracao em producao DEVE ser do tipo "manual" e
   exigir aprovacao humana explicita.
4. Se faltar informacao para um passo, emita um passo do tipo
   "coletar_evidencia" em vez de assumir o valor ausente.
5. A coleta de evidencia DEVE preceder qualquer acao de contencao.
6. Saida: JSON valido conforme o esquema CACAO 2.0. Sem texto fora do JSON.

FERRAMENTAS DISPONIVEIS: dig, nmap, tcpdump, wazuh-api, firewall-api

CONTEXTO ORGANIZACIONAL RECUPERADO:
{contexto_recuperado}

CATEGORIA (NIST SP 800-61r3): {categoria}
INCIDENTE (pseudonimizado): {texto_do_ticket}
```

Sem RAG, o bloco `CONTEXTO ORGANIZACIONAL RECUPERADO` é omitido. A etapa 4 do
*notebook* compara as duas variantes e conta os achados de auditoria de cada
uma.

---

## Variante para saída em Markdown

Substituir a restrição 6 por:

```text
6. Saida: Markdown. Estrutura obrigatoria, nesta ordem:
   ## Gatilho / ## Pre-condicoes / ## Passos (lista numerada, cada passo com
   agente responsavel e criterio de sucesso) / ## Pontos de decisao humana /
   ## Ferramentas utilizadas.
```

## Variante para saída em Ansible

```text
6. Saida: um playbook Ansible valido em YAML. Toda tarefa que altere estado
   DEVE vir com "check_mode: yes" ou estar comentada como requerendo
   aprovacao. Nao use modulos fora da colecao ansible.builtin.
```

---

## Nota sobre reidentificação

O *playbook* é gerado sobre dado **pseudonimizado**: os endereços aparecem como
`[IP_ADDRESS c04a9e21]`. Ele é, portanto, um **modelo** de resposta. A execução
real exige a reidentificação controlada descrita na Seção 1.5.5 do capítulo,
com trilha de auditoria.
