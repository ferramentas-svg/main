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

## Espaçamento e blocos

**Parágrafos são separados por linha em branco.** Copy recebida com as linhas
coladas (só quebra simples entre parágrafos) vira um bloco denso no WhatsApp e
afunda a leitura. Reescrever com uma linha em branco entre cada bloco de ideia
antes de agendar.

Continuam coladas, sem linha em branco no meio:

- A chamada e o link que ela apresenta
  (`⚠️ As vagas estão sendo preenchidas:` + `🔗 https://...`).
- Itens de uma mesma lista (as linhas de `✅`, por exemplo).
- Versos de uma mesma frase quebrada de propósito.

Emoji que abre a linha leva espaço antes do texto: `▶️ Assiste`, não
`▶️Assiste`. Espaço solto no fim da linha e linha em branco dupla saem.

## Escopo de envio

- Por padrão, todo disparo vai para **todos os grupos** e **todas as contas**
  da campanha. Buscar os `accountIds` em `get-release` e passar a lista
  completa.
- **Só marcar participantes quando solicitado explicitamente**
  (`options.mentionAllParticipants`). Sem pedido, não marca.
- **Janela de menção: 8h às 22h.** Quando a marcação está valendo, ela se
  aplica apenas às mensagens agendadas entre 08:00 e 22:00 (horário de
  Brasília), inclusive. Fora dessa faixa — 23h, 23h30, madrugada — a mensagem
  vai sem menção, para não acordar a base.

## Verificação de links

- Todo link próprio deve conter a sigla da campanha de destino
  (`check-in-wcm` na WCM, `check-in-wcd` na WCD, `...-wfp` na WFP).
- Conferir a sigla contra a descrição fixa dos grupos em `get-release`.
- Links raiz de terceiros (YouTube, por exemplo) são exceção e não levam sigla.
- Sigla divergente é erro de copy: avisar antes de agendar, não agendar.

## Mídia (imagem, áudio, vídeo)

- As tools de envio aceitam **apenas URL pública**. Não há upload de arquivo
  local pelo MCP — o `generate-media-id` serve só para posts do Instagram.
- **A URL precisa terminar em extensão de arquivo** (`.png`, `.jpg`, …). O
  SendFlow valida isso na criação da ação e recusa com
  "Formato de arquivo inválido!" quando não há extensão.
- Link do Google Drive **não serve em nenhuma forma**, nem quando o arquivo
  está público (`role: reader, type: anyone`). O link `/view` devolve HTML, e
  as URLs de serviço direto (`lh3.googleusercontent.com/d/<id>`,
  `drive.usercontent.google.com/download?id=…`, `drive.google.com/uc?export=view`)
  não têm extensão, então caem na validação acima. Testado em 26/09.
- Caminhos que funcionam: subir pela interface do SendFlow (hospeda no
  `storage.sendflow.pro`) ou qualquer URL pública terminada em extensão — a
  CDN do ActiveCampaign (`content.app-us1.com/.../arquivo.png`) passa na
  validação.
- Se a copy **referencia a mídia** ("a mensagem aí em cima", "aperta o play no
  áudio", "no print acima"), não agendar o texto sem o arquivo — sozinho ele
  fica quebrado.

### Imagem + texto num disparo só

`send-messages-action` envia mídia e texto na mesma ação, como a interface faz.
O formato do item de mídia **não** é o que o schema do MCP sugere — a URL vai
em `message.image.url`, aninhada, e não em `message.url` nem no topo do item.
Formato confirmado pelo suporte do SendFlow em 26/09:

```json
{
  "releaseId": "<releaseId>",
  "accountsFrom": "accounts",
  "accounts": ["<accountId>", "..."],
  "to": { "type": "release", "ids": ["<releaseId>"] },
  "data": { "messages": [
    { "type": "imageMessage",
      "message": { "image": { "url": "https://..." }, "caption": "" } },
    { "type": "extendedTextMessage",
      "message": { "text": "..." } }
  ]},
  "scheduledTo": "2026-09-27T20:00:00.000Z"
}
```

Com `to.type: "release"`, o `to.ids` recebe o **próprio releaseId** — não pode
ficar vazio e não precisa dos GIDs dos grupos. A campanha inteira é atingida.
Erros que indicam formato errado: "Campo message é obrigatório em cada item"
(falta o wrapper `message`) e "URL do arquivo não informado!" (falta o nível
`image`).

## Agendamento

- Horários informados estão em horário de Brasília (UTC-3).
  `scheduledTo` é ISO 8601 em UTC — somar 3 horas. Ex.: 19h BRT = `T22:00:00Z`.
- Depois de criar, confirmar com `get-action` que voltou
  `scheduled: true` e `error: null`. **`success: true` na criação não garante
  que a ação existe**: em 24/09 um `send-image-action` devolveu
  `{"success": true, "actionId": "..."}` e a ação nunca foi persistida —
  `get-action` respondeu 404 e ela não constava em `list-actions`. Conferir
  sempre, e em lote usar `list-actions` para checar todas de uma vez.
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
| WFP - Atual | `9CCFiGon3VC7zuNnBzlg` |
| WEPSET26 - ATUAL | `xJBmgZnzAErbv9gYDMRM` |
| DQCOUT26 - Meteórico | `29a22nrE5uv5g0WQ9TBB` |
| Desafios Antigos | `O8a5yFUR94rA0U1a6Xov` |
