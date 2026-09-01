# Material prático do minicurso "Inteligência Artificial aplicada à Resposta a Incidentes"

Repositório complementar do capítulo homônimo dos **Minicursos do SBSeg 2026**.

Aqui ficam os *notebooks*, os dados sintéticos, os *prompts* e os procedimentos
operacionais de referência que o capítulo cita mas não reproduz. O capítulo
explica o *porquê*; este repositório mostra o *como*, de forma executável.

| Notebook | Estudo de caso | Duração | Entrada | Saída |
|---|---|---|---|---|
| [`01-anonimizacao.ipynb`](notebooks/01-anonimizacao.ipynb) | EC1: anonimização de *tickets* | ~15 min | `dados/tickets_sinteticos.csv` | `saida/tickets_pseudonimizados.csv` |
| [`02-classificacao.ipynb`](notebooks/02-classificacao.ipynb) | EC2: classificação de incidentes | ~15 min | `tickets_sinteticos.csv`, mais uma chave de API | `saida/tickets_classificados.csv` |
| [`02-classificacao_offline.ipynb`](notebooks/02-classificacao_offline.ipynb) | EC2 sem rede, resultados embutidos | ~10 min | `tickets_sinteticos.csv` | métricas e gráficos na tela |
| [`02-resultados_classificacao_tickets_reais.ipynb`](notebooks/02-resultados_classificacao_tickets_reais.ipynb) | EC2 sobre a base real | ~5 min | `dados/resultados_reais.csv` | métricas e gráficos na tela |
| [`03-playbooks.ipynb`](notebooks/03-playbooks.ipynb) | EC3: geração de *playbooks* | ~12 min | saída do NB2 | `saida/playbooks/` |

O 02 e o 03 formam um **encadeamento**: a saída do primeiro é a entrada do
segundo, o que reproduz o trecho AutoClass-LFI → Módulo PoP do ciclo descrito no
capítulo. O 01 demonstra a etapa que, em produção, antecede os dois, e aqui ele
opera sobre os mesmos *tickets* sintéticos, e não sobre a base real.

Cada *notebook* também roda isolado. O 03 procura os *tickets* classificados em
`saida/`, tanto na raiz quanto em `notebooks/`, e recorre à cópia pré-computada
em `dados/tickets_classificados.csv` quando você ainda não executou o 02.

Dois *notebooks* cobrem o mesmo Estudo de Caso 2 e chegam aos mesmos números.
Use o `02-classificacao.ipynb` se tiver chave de API e quiser ver cada
classificação acontecendo; use o `02-classificacao_offline.ipynb` se não tiver
rede, chave ou cota. O terceiro, o de resultados reais, é só de leitura: ele
mostra o desempenho sobre os 182 incidentes reais a partir de um CSV que contém
apenas identificadores e categorias, sem texto de *ticket* algum, e por isso pode
ser compartilhado livremente.

## Três compromissos de projeto

1. **Auto-contidos.** Nenhum *notebook* depende das ferramentas internas do
   grupo proponente (AnonShield, SecLINC, FrameworkPE, AutoClass-LFI). Eles
   reimplementam o mecanismo essencial em poucas dezenas de linhas, para que o
   comportamento observado na demonstração das ferramentas se torne
   inspecionável. Não substituem as ferramentas de produção: os repositórios
   delas estão listados no fim deste arquivo.
2. **Há sempre um caminho sem GPU, sem rede e sem credenciais.** O 01 é Python
   puro. O 02 tem duas versões equivalentes, uma que chama a API do Gemini e
   outra com os resultados de uma execução real embutidos, que dispensa rede. O
   03 traz um provedor `simulado`, determinístico e local, com os provedores
   reais (Ollama local e APIs compatíveis com OpenAI) plugáveis por uma
   variável. Nenhuma etapa exige GPU.
3. **Sem dado real.** Os `dados/tickets_sinteticos.csv` foram escritos para o
   minicurso. Nomes, e-mails, organizações e domínios são fictícios, e os
   endereços IP vêm exclusivamente das faixas de documentação reservadas pela
   RFC 5737 (`192.0.2.0/24`, `198.51.100.0/24`, `203.0.113.0/24`) e pela
   RFC 3849 (`2001:db8::/32`). Os *tickets* reais que sustentam os resultados
   publicados **não** estão aqui, e não estarão: são compartilhados sob acordo
   com os CSIRTs cedentes, conforme a política de dados descrita no capítulo.

## Instalação

Requer **Python 3.10 ou superior**. As dependências mínimas são as da
biblioteca padrão; tudo o mais é opcional e o código degrada com elegância se
faltar.

### Passo 1: Python

Confira se você já tem, com `python3 --version`. Se o comando não existir ou a
versão for anterior à 3.10, instale:

```bash
# Ubuntu, Debian, Linux Mint
sudo apt update && sudo apt install -y python3 python3-venv python3-pip git

# Fedora
sudo dnf install -y python3 python3-pip git

# macOS (com Homebrew; instale-o em https://brew.sh se ainda não tiver)
brew install python git

# Windows (PowerShell)
winget install --id Python.Python.3.12 -e
winget install --id Git.Git -e
```

No Windows também funciona baixar o instalador em <https://python.org/downloads>.
Marque **"Add python.exe to PATH"** na primeira tela; sem isso, o comando
`python` não é encontrado no terminal.

### Passo 2: o repositório e o ambiente virtual

O ambiente virtual isola estas dependências das do resto da sua máquina, e
permite apagar tudo depois removendo uma única pasta.

```bash
git clone https://github.com/sbseg26-mc1/minicurso.git
cd minicurso

python3 -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
```

O prompt do terminal passa a exibir `(.venv)`. Se ele não aparecer, o ambiente
não foi ativado e os pacotes irão para a instalação global.

### Passo 3: Jupyter e as demais dependências

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

Se quiser apenas o essencial, o Jupyter com `pandas` e `matplotlib` cobre os
caminhos que não usam rede:

```bash
pip install jupyterlab pandas matplotlib
```

### Passo 4: abrir os notebooks

```bash
jupyter lab                        # ou: jupyter notebook
```

O comando abre o navegador em `http://localhost:8888`. Vá até `notebooks/` e
comece pelo `01-anonimizacao.ipynb`: os três são encadeados e cada um consome a
saída do anterior.

Para encerrar, `Ctrl+C` no terminal e depois `deactivate` para sair do ambiente
virtual.

### Sem instalar nada

Cada *notebook* também abre no [Google Colab](https://colab.research.google.com)
pelo menu `Arquivo > Abrir notebook > GitHub`. Nesse caso, execute
`!pip install -q jupyterlab` apenas se algum import falhar, e lembre-se de que o
Colab envia o conteúdo executado para a infraestrutura do Google, o que é
aceitável aqui porque todos os dados deste repositório são sintéticos.

### O que cada dependência opcional habilita

| Pacote | Habilita | Se faltar |
|---|---|---|
| `transformers`, `torch` | NER por *transformer* no NB1 (etapa 3) | cai para um reconhecedor léxico simplificado; a lição de cobertura permanece observável |
| `pandas` | tabelas formatadas, em todos os *notebooks* | as saídas viram listas de dicionários impressas |
| `matplotlib` | gráficos de acurácia e matriz de confusão nos *notebooks* do EC2 | os mesmos números continuam saindo em texto |
| `google-genai` | chamada à API do Gemini no `02-classificacao.ipynb` | use o `02-classificacao_offline.ipynb` |
| `requests` | provedores `ollama` e `openai` no NB3 | apenas o provedor `simulado` fica disponível |

## Modelos: a chave do EC2 e o provedor do EC3

O `02-classificacao.ipynb` chama a API do Gemini, no modelo
`gemini-3.1-flash-lite`, que é o mesmo que produziu os resultados publicados no
capítulo. Registramos o identificador exato de propósito: provedores comerciais
atualizam *endpoints* sem aviso, e reproduzir os números depende dessa
informação. Gere uma chave gratuita no
[Google AI Studio](https://aistudio.google.com/) e exporte-a antes de abrir o
Jupyter:

```bash
export API_KEY="sua-chave"          # Windows PowerShell: $env:API_KEY="sua-chave"
```

Sem chave, sem rede ou sem cota, use o `02-classificacao_offline.ipynb`: ele
percorre a mesma sequência de experimentos e chega aos mesmos números.

O `03-playbooks.ipynb` tem uma única célula de configuração no alto:

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
  ticket_injecao.csv         1 ticket com instrução injetada no corpo do relato,
                             para o exercício de injeção de prompt (seção 6.5
                             do NB2)
  tickets_classificados.csv  saída pré-computada do NB2, para que o NB3 rode
                             isolado
  resultados_sinteticos.csv  resultados das três estratégias sobre os sintéticos
  resultados_reais.csv       resultados sobre os 182 incidentes reais, só com
                             identificadores e categorias, sem texto de ticket
  pops/                      5 procedimentos operacionais padrão, base de
                             recuperação do NB3
prompts/
  ec2-classificacao.md       prompts de classificação (livre, taxonomia,
                             one-shot, PHP)
  ec3-playbook.md            prompt de geração de playbook com restrições
notebooks/                   os cinco notebooks
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
