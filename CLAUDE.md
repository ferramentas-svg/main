# Memória operacional — SendFlow MCP

Regras permanentes para agendar e disparar mensagens via SendFlow Pro.

## Formatação de texto no WhatsApp

**A pontuação fica SEMPRE fora dos marcadores de formatação.**

O WhatsApp não aplica negrito nem itálico quando o marcador encosta em
pontuação (aspas, ponto, vírgula, interrogação, exclamação, dois-pontos).
Nesses casos ele exibe o `*` ou o `_` cru, no meio do texto.

| Errado | Certo |
| --- | --- |
| `_"Como eu queria ter conhecido isso"_` | `"_Como eu queria ter conhecido isso_."` |
| `*Os horários fecham hoje.*` | `*Os horários fecham hoje*.` |
| `*Restam 12 vagas!*` | `*Restam 12 vagas*!` |

Checklist antes de agendar qualquer mensagem:

1. Todo `*` e `_` abre e fecha colado em letra ou número — nunca em espaço
   nem em sinal de pontuação.
2. Cada marcador tem o seu par. Marcador ímpar aparece cru na mensagem.
3. Negrito aninhado dentro de itálico (`_texto *negrito* texto_`) é válido.

## Escopo de envio

- Por padrão, todo disparo vai para **todos os grupos** e **todas as contas**
  da campanha. Buscar os `accountIds` em `get-release` e passar a lista
  completa.
- **Só marcar participantes quando solicitado explicitamente**
  (`options.mentionAllParticipants`). Sem pedido, não marca.

## Verificação de links

- Todo link próprio deve conter a sigla da campanha de destino
  (`check-in-wcm` na WCM, `check-in-wcd` na WCD, `...-wfp` na WFP).
- Conferir a sigla contra a descrição fixa dos grupos em `get-release`.
- Links raiz de terceiros (YouTube, por exemplo) são exceção e não levam sigla.
- Sigla divergente é erro de copy: avisar antes de agendar, não agendar.

## Agendamento

- Horários informados estão em horário de Brasília (UTC-3).
  `scheduledTo` é ISO 8601 em UTC — somar 3 horas. Ex.: 19h BRT = `T22:00:00Z`.
- Depois de criar, confirmar com `get-action` que voltou
  `scheduled: true` e `error: null`.
- Para corrigir uma ação já agendada: `cancel-actions` e recriar.
  `get-action` devolve payload mínimo e **não** retorna o texto da mensagem,
  então ações criadas pela interface não podem ser reconstruídas pela API sem
  o texto original em mãos.

## IDs de campanha usados com frequência

| Campanha | Release ID |
| --- | --- |
| WCM Anterior | `ly4HcYeRc7RQmibxpkxl` |
| WCM Atual - Domingo - Aplicação | `LahJb7ltS90576gYeu4c` |
| WFP - Anterior | `Nx0SFwVXlOpFQySy7RXU` |
| WCD - Atual | `pqZ1quppMxTMPRfzn6SU` |
