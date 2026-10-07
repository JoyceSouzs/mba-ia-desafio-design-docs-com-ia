# ADR-007: Payload como snapshot e filtro de eventos na inserção da outbox

- **Status:** Aceito
- **Data da decisão:** reunião técnica de quinta-feira, [09:34] Bruno e [09:52] Bruno
- **Decisores:** Larissa (Tech Lead), Diego (Plataforma), Bruno (Pedidos), Marcos (PM)
- **Relacionados:** [ADR-001](ADR-001-outbox-no-mysql.md)

## Contexto

Duas decisões secundárias afetam o que é gravado na outbox:

1. Cada webhook escolhe quais status quer receber, por exemplo "só SHIPPED e DELIVERED" ([09:33] Marcos). Esse filtro pode ser aplicado na inserção da outbox ou no envio ([09:34] Diego).
2. O evento pode guardar o payload já pronto ou só o `order_id`, montando o payload no envio ([09:51] Bruno).

## Decisão

1. **Filtrar na inserção**: em `publishWebhookEvent`, só se grava evento para webhooks ativos do customer cujo filtro inclui o `toStatus`. Se nenhum webhook quer aquele status, nada é inserido ([09:34] Bruno, [09:34] Diego).
2. **Payload renderizado na inserção (snapshot)**: o JSON completo é gravado na linha da outbox, refletindo o estado do pedido no momento da mudança de status ([09:52] Larissa, [09:52] Diego, [09:52] Bruno).
3. **Payload enxuto**: `event_id`, `event_type` (`order.status_changed`), `timestamp` ISO 8601, `order_id`, `order_number`, `from_status`, `to_status`, `customer_id` e campos básicos como `total_cents`. **Sem `items`**; quem quiser detalhe consulta `GET /orders/:id` ([09:43] Diego, [09:44] Bruno).
4. **Limite de 64 KB**: payload maior gera erro, nunca truncamento ([09:23] Sofia, [09:24] Diego, [09:24] Larissa).

## Alternativas Consideradas

| Alternativa | Por que foi descartada | Fonte |
|---|---|---|
| Filtrar no momento do envio | Grava linhas que nunca serão enviadas; filtrar antes "economiza linha na tabela" | [09:34] Bruno |
| Guardar só `order_id` e renderizar no envio | Se o pedido mudar depois, o evento não reflete o estado da mudança; gera "caso esquisito" | [09:52] Larissa |
| Incluir `items` no payload | Infla o payload sem necessidade | [09:43] Diego |
| Truncar payload acima do limite | Payload desse tamanho indica erro; preferível falhar | [09:23] Sofia |

## Consequências

**Positivas**
- Evento imutável e fiel ao momento da transição; replay da DLQ reenvia exatamente o mesmo conteúdo.
- Outbox menor, contendo só o que será entregue.
- Worker não precisa consultar `orders` no envio.

**Negativas / trade-offs**
- Alterar o filtro de status de um webhook não afeta eventos que já estão na outbox.
- O payload é duplicado por webhook quando um customer tem vários endpoints interessados no mesmo status.
- A inserção na outbox passa a ler a configuração de webhooks dentro da transação de `changeStatus`.
- **Trade-off explícito:** mais trabalho na escrita em troca de um envio simples, determinístico e sem leituras extras.

## Referências

- Transcrição: [09:23] Sofia, [09:24] Diego, [09:24] Larissa, [09:33] Marcos, [09:34] Diego, [09:34] Bruno, [09:43] Diego, [09:44] Bruno, [09:51] Bruno, [09:52] Larissa, [09:52] Diego, [09:52] Bruno
- Código: `src/modules/orders/order.service.ts`
