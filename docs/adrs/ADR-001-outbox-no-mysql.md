# ADR-001: Padrão Outbox no MySQL existente

- **Status:** Aceito
- **Data da decisão:** reunião técnica de quinta-feira, [09:08] Larissa
- **Decisores:** Larissa (Tech Lead), Diego (Plataforma), Bruno (Pedidos)
- **Relacionados:** [ADR-002](ADR-002-worker-separado-em-polling.md), [ADR-005](ADR-005-at-least-once-com-x-event-id.md), [ADR-007](ADR-007-payload-snapshot-e-filtro-na-insercao.md)

## Contexto

Três clientes B2B pediram para ser notificados quando o status dos seus pedidos muda ([09:00] Marcos). A mudança de status acontece em `OrderService.changeStatus` (`src/modules/orders/order.service.ts`), que já executa dentro de um único `prisma.$transaction`:

1. lê o pedido com os itens;
2. valida a transição com `canTransition` (`src/modules/orders/order.status.ts`);
3. debita ou repõe estoque (`debitStock` / `replenishStock`);
4. atualiza `orders.status`;
5. insere uma linha em `order_status_history`.

Bruno lembrou que essa transação "já é pesada" e que uma chamada HTTP no meio dela faria qualquer cliente lento travar a mudança de status de outros pedidos ([09:04] Bruno). Também não há como desfazer a mudança de status se o cliente estiver fora do ar ([09:04] Bruno).

O projeto não tem nenhuma infraestrutura de mensageria: o único armazenamento é o MySQL acessado via Prisma (`prisma/schema.prisma`, `src/config/database.ts`).

## Decisão

Adotar o **padrão Transactional Outbox no MySQL existente** ([09:06] Diego, [09:08] Larissa):

- Criar a tabela `webhook_outbox`. Dentro da **mesma transação** de `changeStatus`, depois da inserção em `order_status_history`, inserir uma linha por evento a ser entregue ([09:06] Diego, [09:40] Bruno).
- Se a inserção na outbox falhar, a transação inteira faz rollback: "não pode ter caso de status mudar e evento não sair" ([09:40] Bruno, [09:41] Diego).
- A tabela tem índice em `status` (pendente, processando, falhou, entregue) e em `created_at` ([09:08] Diego).
- Identificadores em UUID, seguindo o padrão `@id @default(uuid()) @db.Char(36)` usado em todos os models de `prisma/schema.prisma` ([09:51] Larissa).
- Um worker separado lê a outbox e faz as chamadas HTTP (ver [ADR-002](ADR-002-worker-separado-em-polling.md)).

## Alternativas Consideradas

| Alternativa | Por que foi descartada | Fonte |
|---|---|---|
| Disparar o HTTP de forma síncrona dentro de `changeStatus` | Cliente lento trava a transação e as mudanças de status de outros pedidos; cliente fora do ar forçaria um rollback de status que não faz sentido | [09:04] Bruno, [09:06] Diego |
| Redis Streams (ou fila similar) | Exige subir infraestrutura nova; para um time pequeno, "subir Redis Cluster pra isso é overengineering" | [09:07] Larissa, [09:07] Diego |

## Consequências

**Positivas**
- Atomicidade: se a transação de status commitou, o evento está registrado; se fez rollback, o evento some junto. "Não tem inconsistência possível" ([09:06] Diego).
- Nenhuma infraestrutura nova: mesmo banco, mesma stack, mesmo Prisma ([09:07] Diego, [09:11] Diego).
- A mudança de status deixa de depender da disponibilidade do cliente externo.

**Negativas / trade-offs**
- A transação de `changeStatus` ganha mais uma escrita (custo pequeno, mas cresce com o número de webhooks do customer).
- A tabela cresce continuamente. O arquivamento de linhas entregues ("depois de 30 dias ou assim") ficou **fora do escopo** desta feature ([09:08] Diego).
- A entrega passa a ser assíncrona, com latência mínima igual ao intervalo de polling do worker ([09:10] Larissa).
- **Trade-off explícito:** trocamos throughput e reatividade de um broker dedicado por consistência transacional e zero infraestrutura adicional.

## Referências

- Transcrição: [09:03] Larissa, [09:04] Bruno, [09:06] Diego, [09:07] Larissa, [09:07] Diego, [09:08] Diego, [09:08] Larissa, [09:40] Bruno, [09:41] Diego, [09:51] Larissa
- Código: `src/modules/orders/order.service.ts` (`changeStatus`), `src/modules/orders/order.status.ts`, `prisma/schema.prisma`
