---
id: POP-CAT3-DDOS-01
categorias: [CAT3]
titulo: Negação de serviço distribuída volumétrica ou aplicacional
palavras_chave: [ddos, dos, negacao de servico, volumetrico, amplificacao, ntp, memcached, blackhole, saturacao, banda]
---

# POP: negação de serviço distribuída

## Gatilho
Saturação de enlace, degradação de latência ou indisponibilidade de serviço
com padrão de tráfego anômalo, detectada por telemetria de fluxo, balanceador
ou reclamação do cliente.

## Pré-condições
- Prefixo ou serviço alvo identificado, com cliente ou área responsável.
- Baseline de tráfego disponível para comparação.
- Canal de acionamento do provedor de trânsito conhecido.

## Passos

1. **Caracterizar o vetor.** Determinar se o ataque é volumétrico (satura
   banda) ou aplicacional (satura CPU ou conexões), a partir de amostras de
   fluxo: protocolo, portas de origem e destino, taxa em bps e em pps.
   Critério de sucesso: vetor classificado. Agente: analista N2.
2. **Coletar evidência.** Preservar amostras de fluxo, gráficos de utilização e
   registros do balanceador ou do servidor de aplicação. Agente: analista N1.
3. **Medir o impacto real.** Confirmar se há indisponibilidade total, parcial
   ou apenas degradação, e quais serviços e clientes estão afetados. Critério
   de sucesso: escopo de impacto documentado. Agente: analista N2.
4. **Aplicar mitigação proporcional.** Para vetor volumétrico com refletores,
   preferir filtro de taxa por porta de origem no roteador de borda. Recorrer
   a anúncio de rota `blackhole` apenas quando a saturação impedir qualquer
   outra ação, porque essa medida completa a indisponibilidade do alvo.
   **Passo manual: exige aprovação, por afetar disponibilidade.** Agente:
   administrador de rede.
5. **Mitigar o vetor aplicacional, quando for o caso.** Limitação de taxa por
   endereço, cache agressivo do endpoint atacado e desafio de cliente.
   **Passo manual.** Agente: administrador de aplicação.
6. **Acionar o provedor de trânsito.** Quando o volume exceder a capacidade de
   mitigação local. Agente: analista N2.
7. **Notificar o cliente ou a área afetada.** Comunicar o vetor, a mitigação
   aplicada e a expectativa de normalização. Agente: analista N1.
8. **Documentar e encerrar.** Registrar pico, duração, vetor, mitigação e
   lições aprendidas. Agente: analista N1.

## Pontos de decisão humana
- Passos 4 e 5: toda mitigação que afete disponibilidade.
- Passo 6: acionamento externo com implicações contratuais.

## Ferramentas autorizadas
`tcpdump`, `nmap`, `firewall-api`, `wazuh-api`

## Erros comuns
- Aplicar `blackhole` como primeira medida: entrega ao atacante exatamente o
  objetivo dele.
- Tratar exaustão de CPU no backend como problema de banda, e dimensionar a
  resposta pelo indicador errado.
