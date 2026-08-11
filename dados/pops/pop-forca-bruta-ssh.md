---
id: POP-CAT12-SSH-01
categorias: [CAT12, CAT1]
titulo: Tentativa de intrusão por força bruta em SSH
palavras_chave: [ssh, forca bruta, autenticacao, failed password, bastion, porta 22, credencial]
---

# POP: força bruta em SSH

## Gatilho
Volume anômalo de tentativas de autenticação SSH malsucedidas contra um ou
mais hosts, detectado por IDS, SIEM ou análise de `auth.log`.

## Pré-condições
- Host alvo identificado no inventário e classificado quanto à criticidade.
- Acesso de leitura aos registros de autenticação do host.

## Passos

1. **Determinar se houve autenticação bem-sucedida.** Filtrar os registros por
   `Accepted password` e `Accepted publickey` provenientes dos endereços de
   origem suspeitos, na janela do ataque. Critério de sucesso: resposta
   binária, com evidência. Agente: analista N1.
2. **Coletar evidência.** Preservar `auth.log`, `wtmp`, `lastlog` e a lista de
   processos e conexões ativas. **Este passo precede qualquer contenção**,
   sob pena de destruir a evidência da investigação. Agente: analista N2.
3. **Caracterizar a origem.** Consultar reputação, ASN e histórico dos
   endereços de origem. Critério de sucesso: origens classificadas como
   automatizadas ou direcionadas. Agente: analista N1.
4. **Bloquear as origens.** Aplicar bloqueio no firewall de borda ou no
   `fail2ban` local para os endereços confirmados. **Passo manual quando o
   bloqueio for de faixa (`/24` ou maior), pelo risco de afetar tráfego
   legítimo.** Agente: administrador de rede.
5. **Endurecer o serviço.** Verificar e, se necessário, ajustar
   `PermitRootLogin no`, autenticação por chave, `MaxAuthTries` e exposição do
   serviço à Internet. **Passo manual.** Agente: administrador de sistemas.
6. **Escalar em caso de sucesso do atacante.** Se o passo 1 confirmou
   autenticação bem-sucedida, tratar como comprometimento de conta (CAT1):
   revogar credenciais e sessões, e acionar o procedimento de resposta a
   comprometimento. Agente: analista N2.
7. **Documentar e encerrar.** Agente: analista N1.

## Pontos de decisão humana
- Passo 4, quando o bloqueio for de faixa.
- Passo 5, por alterar configuração de serviço em produção.
- Passo 6, decisão de escalar.

## Ferramentas autorizadas
`nmap`, `tcpdump`, `wazuh-api`, `firewall-api`

## Erros comuns
- Bloquear a origem antes de coletar evidência.
- Concluir que não houve comprometimento apenas porque o volume de falhas é
  alto: um ataque bem-sucedido produz poucas linhas no meio do ruído.
