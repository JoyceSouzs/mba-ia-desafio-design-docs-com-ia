# ADR-002: Worker em processo separado, lendo a outbox por polling de 2 segundos

- **Status:** Aceito
- **Data da decisão:** reunião técnica de quinta-feira, [09:10] Larissa e [09:11] Diego
- **Decisores:** Larissa (Tech Lead), Diego (Plataforma), Bruno (Pedidos)
- **Relacionados:** [ADR-001](ADR-001-outbox-no-mysql.md), [ADR-003](ADR-003-retry-backoff-e-dlq.md)

## Contexto

Com a outbox definida no [ADR-001](ADR-001-outbox-no-mysql.md), falta decidir **como** e **onde** os eventos são consumidos.

- Requisito de latência dos clientes: "qualquer coisa abaixo de 10 segundos já é tempo real" ([09:02] Marcos).
- O MySQL não oferece um mecanismo nativo de notificação para processos externos, como o `NOTIFY/LISTEN` do Postgres ([09:09] Diego).
- Hoje a aplicação tem um único entry-point HTTP, `src/server.ts`, que cria o app Express, usa o `prisma` de `src/config/database.ts` e trata `SIGINT`/`SIGTERM` com `prisma.$disconnect()`.

## Decisão

1. **Polling em loop a cada 2 segundos**: o worker busca os eventos pendentes mais antigos em lote pequeno, processa e marca o resultado ([09:08] Diego, [09:09] Diego, [09:10] Larissa).
2. **Processo separado da API**: novo entry-point `src/worker.ts` e script `npm run worker` em `package.json` ([09:11] Larissa). Se a API reinicia, o worker não morre junto ([09:11] Diego).
3. **Mesmo banco e mesma stack, PrismaClient próprio**: mesma `DATABASE_URL`, mas uma instância nova de `PrismaClient` porque é outro processo Node ([09:11] Bruno, [09:30] Bruno). A instância deve ser criada com `createPrismaClient()` de `src/config/database.ts`.
4. **Lógica de processamento dentro do módulo**: `src/modules/webhooks/webhook.worker.ts` (ou `webhook.processor.ts`); `src/worker.ts` só faz o bootstrap ([09:28] Bruno).
5. **Uma única instância do worker** nesta fase. Ordering garantido apenas **por `order_id`**, pela ordem de `created_at` da outbox, e somente enquanto houver um worker ([09:12] Diego, [09:13] Larissa).

## Alternativas Consideradas

| Alternativa | Por que foi descartada | Fonte |
|---|---|---|
| Trigger no banco para reagir à inserção | Trigger do MySQL só executa SQL e não notifica processo externo; avisar o worker exigiria improviso (escrever em arquivo, bater em endpoint) | [09:09] Bruno, [09:09] Diego |
| Rodar o worker dentro do processo da API | Se a API reinicia, o worker morre junto | [09:11] Diego |
| Compartilhar o mesmo PrismaClient da API | PrismaClient é por processo; o worker é outro processo Node | [09:29] Diego, [09:30] Bruno |
| Múltiplos workers em paralelo | Perde a garantia de ordem por pedido; escalar (particionar por `order_id` ou lock pessimista) é "problema do futuro" | [09:12] Diego, [09:13] Diego |

## Consequências

**Positivas**
- Atende o requisito de menos de 10 s com folga: no pior caso o evento espera ~2 s para ser lido ([09:09] Diego, [09:10] Larissa).
- Isolamento de falhas entre API e worker; deploy e restart independentes.
- Nenhum mecanismo novo de infraestrutura.

**Negativas / trade-offs**
- Latência mínima de até 2 s aceita explicitamente ([09:10] Larissa).
- Consultas periódicas ao banco mesmo sem eventos (mitigado pelos índices de `status` e `created_at`, [09:08] Diego).
- **Limitação conhecida:** sem ordering global e sem escala horizontal nesta fase ([09:13] Larissa). Os clientes não pediram ordering global ([09:14] Marcos).
- Mais um processo para operar e monitorar em produção.

## Referências

- Transcrição: [09:02] Marcos, [09:08] Diego, [09:09] Diego, [09:09] Bruno, [09:10] Larissa, [09:11] Diego, [09:11] Larissa, [09:11] Bruno, [09:12] Diego, [09:13] Diego, [09:13] Larissa, [09:14] Marcos, [09:28] Bruno, [09:30] Bruno
- Código: `src/server.ts`, `src/config/database.ts`, `package.json`
