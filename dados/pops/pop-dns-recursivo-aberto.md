---
id: POP-CAT5-DNS-01
categorias: [CAT5]
titulo: Resolvedor DNS recursivo aberto passível de amplificação
palavras_chave: [dns, recursivo, resolvedor, amplificacao, udp, porta 53, monlist, refletor]
---

# POP: resolvedor DNS recursivo aberto

## Gatilho
Notificação de terceiro (CERT.br, PoP ou parceiro) ou varredura interna
indicando que um host da organização responde a consultas DNS recursivas
originadas de redes externas.

## Pré-condições
- Endereço IP e nome do host confirmados no inventário de ativos.
- Responsável técnico pelo ativo identificado.
- Janela de manutenção conhecida, caso a mudança exija reinício do serviço.

## Passos

1. **Verificar a recursividade externa.** Executar, de fora da rede da
   organização, uma consulta de teste ao resolvedor. Ferramenta: `dig`.
   Critério de sucesso: a resposta indica se a recursão está de fato aberta.
   Agente: analista N1.
2. **Coletar evidência antes de qualquer alteração.** Salvar a saída da
   consulta, o trecho de configuração vigente e as métricas de tráfego da
   porta 53/UDP nas últimas 24 horas. Critério de sucesso: evidência
   arquivada no caso. Agente: analista N1.
3. **Identificar o escopo legítimo de atendimento.** Levantar quais redes
   precisam de recursão neste resolvedor. Critério de sucesso: lista de
   prefixos autorizados validada com o responsável pelo ativo. Agente:
   administrador de rede.
4. **Restringir a recursão.** Aplicar `allow-recursion` (BIND) ou equivalente
   restringindo às redes internas identificadas. **Passo manual: exige
   aprovação humana explícita, por alterar configuração em produção.**
   Agente: administrador de rede.
5. **Revalidar externamente.** Repetir o passo 1 e confirmar que a consulta
   externa já não obtém resposta recursiva. Critério de sucesso: consulta
   externa recusada. Agente: analista N1.
6. **Notificar o responsável e a origem da notificação.** Comunicar a
   correção a quem reportou e ao responsável pelo ativo. Agente: analista N1.
7. **Documentar e encerrar.** Registrar causa, ação e evidência de
   revalidação. Agente: analista N1.

## Pontos de decisão humana
- Passo 4 (alteração de configuração em produção).
- Decisão de bloquear a porta 53/UDP na borda, caso o serviço não deva ser
  público: exige aprovação da coordenação de redes.

## Ferramentas autorizadas
`dig`, `nmap`, `tcpdump`, `firewall-api`

## Erros comuns
- Bloquear a porta 53 na borda sem verificar se o host também é autoritativo
  para zonas públicas, o que derruba a resolução legítima do domínio.
- Encerrar o caso sem revalidação externa: a configuração pode não ter sido
  recarregada.
