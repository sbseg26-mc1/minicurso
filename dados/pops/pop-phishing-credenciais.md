---
id: POP-CAT7-PH-01
categorias: [CAT7, CAT1]
titulo: Phishing e comprometimento de credenciais institucionais
palavras_chave: [phishing, engenharia social, credencial, senha, conta, encaminhamento, mfa, dominio falso, fraude]
---

# POP: phishing e comprometimento de credenciais

## Gatilho
Relato de usuário sobre mensagem suspeita, detecção de domínio semelhante ao
institucional, ou autenticação com padrão anômalo de origem ou horário.

## Pré-condições
- Amostra da mensagem com cabeçalhos completos, quando houver.
- Acesso aos registros de autenticação do provedor de identidade.

## Passos

1. **Preservar a amostra.** Obter a mensagem original com cabeçalhos, a URL de
   destino e uma captura da página, sem submeter credenciais. Agente:
   analista N1.
2. **Determinar o alcance.** Levantar quantos usuários receberam a mensagem e
   quantos clicaram, a partir dos registros do gateway de e-mail e do proxy.
   Critério de sucesso: lista de destinatários e de cliques. Agente:
   analista N1.
3. **Identificar credenciais efetivamente comprometidas.** Cruzar a lista de
   cliques com autenticações bem-sucedidas de origem anômala. Critério de
   sucesso: lista de contas suspeitas. Agente: analista N2.
4. **Conter as contas comprometidas.** Redefinir senha, revogar sessões e
   tokens ativos, e habilitar segundo fator obrigatório. **Passo manual para
   contas de gestores ou de serviço, pelo impacto operacional.** Agente:
   administrador de identidade.
5. **Procurar persistência na caixa postal.** Verificar regras de
   encaminhamento automático, delegações e aplicativos autorizados criados no
   período. Remover o que for indevido. Agente: administrador de identidade.
6. **Bloquear a infraestrutura do atacante.** Bloquear o domínio e a URL no
   proxy e no gateway de e-mail, e solicitar remoção ao registrador ou
   provedor de hospedagem. Agente: analista N1.
7. **Comunicar a comunidade.** Alerta preventivo descrevendo o golpe, sem
   expor os usuários atingidos. Agente: analista N1.
8. **Documentar e encerrar.** Agente: analista N1.

## Pontos de decisão humana
- Passo 4, para contas de gestores ou de serviço.
- Passo 7, quanto ao teor e ao alcance da comunicação.

## Ferramentas autorizadas
`wazuh-api`, `firewall-api`

## Erros comuns
- Redefinir a senha sem revogar as sessões ativas, o que mantém o acesso do
  atacante.
- Encerrar o caso sem verificar regras de encaminhamento: é o mecanismo de
  persistência mais comum e o menos procurado.
- Divulgar o alerta identificando quem caiu no golpe.
