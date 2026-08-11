# Material prático do minicurso "Inteligência Artificial aplicada à Resposta a Incidentes"

Repositório complementar do capítulo homônimo dos **Minicursos do SBSeg 2026**.

Aqui ficam os *notebooks*, os dados sintéticos, os *prompts* e os procedimentos
operacionais de referência que o capítulo cita mas não reproduz. O capítulo
explica o *porquê*; este repositório mostra o *como*, de forma executável.

| Notebook | Estudo de caso | Duração | Entrada | Saída |
|---|---|---|---|---|
| [`01-anonimizacao.ipynb`](notebooks/01-anonimizacao.ipynb) | EC1: anonimização de *tickets* | ~15 min | `dados/tickets_sinteticos.csv` | `saida/tickets_pseudonimizados.csv` |
| [`02-classificacao.ipynb`](notebooks/02-classificacao.ipynb) | EC2: classificação de incidentes | ~15 min | saída do NB1 | `saida/incidentes_classificados.csv` |
| [`03-playbooks.ipynb`](notebooks/03-playbooks.ipynb) | EC3: geração de *playbooks* | ~12 min | saída do NB2 | `saida/playbooks/` |

Os três formam um **encadeamento**: a saída de um é a entrada do seguinte, o que
reproduz o ciclo ANON-LFI → AutoClass-LFI → Módulo PoP descrito no capítulo.
Cada um também roda isolado, porque regenera a entrada faltante se ela não
existir.

## Três compromissos de projeto

1. **Auto-contidos.** Nenhum *notebook* depende das ferramentas internas do
   grupo proponente (AnonShield, SecLINC, FrameworkPE, AutoClass-LFI). Eles
   reimplementam o mecanismo essencial em poucas dezenas de linhas, para que o
   comportamento observado na demonstração das ferramentas se torne
   inspecionável. Não substituem as ferramentas de produção: os repositórios
   delas estão listados no fim deste arquivo.
2. **Executam sem GPU, sem rede e sem credenciais.** Onde há uma chamada a
   modelo de linguagem, existe um provedor `simulado`, determinístico e local,
   que permite completar a atividade inteira offline. Provedores reais (Ollama
   local e APIs compatíveis com OpenAI) são plugáveis trocando uma variável.
3. **Sem dado real.** Os `dados/tickets_sinteticos.csv` foram escritos para o
   minicurso. Nomes, e-mails, organizações e domínios são fictícios, e os
   endereços IP vêm exclusivamente das faixas de documentação reservadas pela
   RFC 5737 (`192.0.2.0/24`, `198.51.100.0/24`, `203.0.113.0/24`) e pela
   RFC 3849 (`2001:db8::/32`). Os *tickets* reais que sustentam os resultados
   publicados **não** estão aqui, e não estarão: são compartilhados sob acordo
   com os CSIRTs cedentes, conforme a política de dados descrita no capítulo.

## Instalação

Requer **Python 3.10+**. As dependências mínimas são as da biblioteca padrão;
tudo o mais é opcional e o código degrada com elegância se faltar.

```bash
git clone https://github.com/sbseg26-mc1/minicurso.git
cd minicurso
python3 -m venv .venv && source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter lab            # ou: jupyter notebook
```

Instalação mínima, se você quiser apenas rodar tudo com o provedor `simulado`:

```bash
pip install jupyterlab
```

### O que cada dependência opcional habilita

| Pacote | Habilita | Se faltar |
|---|---|---|
| `transformers`, `torch` | NER por *transformer* no NB1 (etapa 3) | cai para um reconhecedor léxico simplificado; a lição de cobertura permanece observável |
| `pandas` | tabelas formatadas | as saídas viram listas de dicionários impressas |
| `scikit-learn` | relatório de classificação do NB2 | usa a implementação em Python puro que acompanha o *notebook* |
| `requests` | provedores `ollama` e `openai` | apenas o provedor `simulado` fica disponível |

## Escolhendo o provedor de modelo (NB2 e NB3)

No alto de cada *notebook* há uma única célula de configuração:

```python
PROVEDOR = "simulado"     # "simulado" | "ollama" | "openai"
MODELO   = "qwen3:14b"    # usado por "ollama" e "openai"
```

- **`simulado`** (padrão): classificador e gerador determinísticos, baseados em
  regras e em recuperação por similaridade, escritos para esta atividade. **Não
  é um LLM.** Serve para que a mecânica do *pipeline*, das métricas e da
  auditoria seja exercitada mesmo sem infraestrutura. Os números que ele produz
  não devem ser lidos como desempenho de modelo de linguagem.
- **`ollama`**: fala com um [Ollama](https://ollama.com) em
  `http://localhost:11434`. É o caminho que materializa o argumento de
  privacidade do capítulo, porque o dado não sai da máquina.
  Sugestão: `ollama pull qwen3:14b` (ou `phi3:14b`, `deepseek-r1:14b`).
- **`openai`**: qualquer endpoint compatível com a API de *chat completions*.
  Configure `OPENAI_API_KEY` e, se necessário, `OPENAI_BASE_URL` no ambiente.
  Lembre-se de que isso envia o conteúdo dos *tickets* a um terceiro, e por isso
  só deve ser usado sobre dados já pseudonimizados.

## Estrutura

```
dados/
  tickets_sinteticos.csv     24 tickets sintéticos rotulados (12 categorias NIST)
  pops/                      5 procedimentos operacionais padrão, base de
                             recuperação do NB3
prompts/
  ec2-classificacao.md       prompts de classificação (livre, taxonomia,
                             one-shot, PHP)
  ec3-playbook.md            prompt de geração de playbook com restrições
notebooks/                   os três notebooks
saida/                       criado na execução; não versionado
```

## Taxonomia adotada

As doze categorias funcionais derivadas do NIST SP 800-61r3, conforme a
Tabela 1.2 do capítulo: `CAT1` comprometimento de conta, `CAT2` *malware*,
`CAT3` negação de serviço, `CAT4` exfiltração ou vazamento, `CAT5` exploração de
vulnerabilidade, `CAT6` abuso interno, `CAT7` engenharia social, `CAT8`
incidente físico, `CAT9` alteração não autorizada, `CAT10` uso indevido de
recursos, `CAT11` problema de terceiro, `CAT12` tentativa de intrusão.

## Ferramentas de produção do grupo proponente

Os *notebooks* são didáticos. Para uso real, vá às ferramentas:

- **AnonShield** (pseudonimização em escala): <https://github.com/AnonShield/anonshield>
- **ANON-LFI** (anonimização de incidentes, GT-LFI): <https://github.com/gt-rnp-lfi/anon-lfi>
- **AutoClass-LFI** (classificação e geração de *playbooks*, GT-LFI): <https://github.com/gt-rnp-lfi/lfi-autoclass>
- **SecLINC** (engenharia de *prompts* para categorização): <https://github.com/AI-Horizon-Labs/SecLINC>
- **FrameworkPE** (cinco técnicas de *prompting*, com execução local): <https://github.com/AILabs4All/FrameworkPE>

## Licença

Código sob licença MIT (ver [`LICENSE`](LICENSE)). Textos, dados sintéticos e
material didático sob CC BY 4.0. Ao reutilizar, cite o capítulo dos Minicursos
do SBSeg 2026.

## Agradecimentos

CAPES (código 001), RNP (Programa Hackers do Bem e GT-LFI), FAPERGS (termos
24/2551-0001368-7 e 24/2551-0000726-1), CNPq (409743/2025-9) e ANATEL (Termo de
Execução Descentralizada com o ITA).
