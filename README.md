# Da Reunião ao Documento: Webhooks de Notificação de Pedidos

Pacote de design docs do **Sistema de Webhooks de Notificação de Pedidos** do OMS, produzido com IA a partir da transcrição de uma reunião técnica ([TRANSCRICAO.md](TRANSCRICAO.md)) e do código desta aplicação.

> O enunciado original do desafio está no histórico do git (commit `e7f6311`, arquivo `README.md`).

---

## Sobre o desafio

Uma empresa que opera um Order Management System em produção decidiu, em uma call de ~55 minutos entre tech lead, PM, dois engenheiros e uma engenheira de segurança, construir webhooks para avisar clientes B2B quando o status dos pedidos deles muda. Nada foi registrado além da transcrição. O desafio é transformar essa conversa, junto com o código Node.js/TypeScript/Prisma existente, em um pacote de documentação acionável: PRD, RFC, FDD, ADRs e um tracker que prova a origem de cada item.

A parte difícil não é escrever, e sim **filtrar**. A reunião mistura decisões fechadas, ideias descartadas (Redis, trigger de banco, exactly-once), pontos adiados (email, dashboard, rate limiting) e detalhes que ninguém decidiu. Todo o pacote foi construído com a regra de que nada entra sem um `[hh:mm] Nome` ou um caminho de arquivo real. O que não tem origem aparece marcado como **[HIPÓTESE]**, nunca como fato.

---

## Ferramentas de IA utilizadas

- **Claude Code (modelo Claude Opus)**: ferramenta principal. Leu o repositório e a transcrição diretamente no terminal, montou o plano de trabalho em modo de planejamento, conduziu as entrevistas de PRD e FDD respondendo pela transcrição, escreveu os documentos e rodou os scripts de verificação.
- **Prompts de entrevista do curso (PRD e FDD)**: usados como template de estrutura e de perguntas, adaptados para que as respostas viessem da transcrição e não de um usuário humano.
- **Shell (`grep`, `test -e`)**: verificação automática de timestamps e caminhos citados nos documentos.

---

## Workflow adotado

1. **Contextualização.** O Claude Code leu `TRANSCRICAO.md` inteira e os arquivos de integração (`order.service.ts`, `order.status.ts`, `shared/errors/*`, `middlewares/*`, `logger`, `schema.prisma`, `app.ts`, `server.ts`, `package.json`).
2. **Inventário antes de escrever.** Em modo de planejamento, extraí da transcrição cinco listas: decisões fechadas, alternativas descartadas, itens fora de escopo/adiados, questões em aberto e pontos de integração com o código. Esse inventário virou o plano aprovado e é a base de tudo.
3. **ADRs primeiro** (7 arquivos em `docs/adrs/`), um por decisão, no formato MADR.
4. **RFC**: proposta em nível de arquitetura, linkando os ADRs, com as alternativas e as questões em aberto do inventário.
5. **FDD**: entrevista do prompt de FDD respondida pela transcrição (registro em `docs/processo/entrevista-fdd.md`), depois o documento no esqueleto do prompt, com a seção extra "Integração com o sistema existente".
6. **PRD**: entrevista do prompt de PRD respondida pela transcrição (registro em `docs/processo/entrevista-prd.md`), depois o documento no esqueleto exato do prompt.
7. **Tracker**: varredura dos documentos prontos, uma linha por item, com uma seção separada para as hipóteses.
8. **Verificação automática e revisão** (ver "Iterações e ajustes").
9. **README** por último.

---

## Prompts customizados

### 1. Regra de entrevista respondida pela transcrição

Aplicada aos prompts de entrevista de PRD e FDD do curso, que originalmente pedem respostas a um usuário humano:

```text
Execute o prompt de entrevista abaixo, mas você faz os dois papéis.
Para cada pergunta do roteiro:
- responda SOMENTE com o que está em TRANSCRICAO.md ou no código do repositório;
- cite a origem de cada resposta como [hh:mm] Nome ou como caminho de arquivo;
- se a transcrição não responder, NÃO invente: escreva [HIPÓTESE] com a justificativa
  ou [EM ABERTO] se foi levantado e não decidido na reunião;
- mantenha o resumo de cada etapa e registre a entrevista em docs/processo/.
Depois gere o documento final exatamente no esqueleto do prompt, sem travessões "—".
```

### 2. Filtragem do que entra e do que não entra

Usado no início, antes de escrever qualquer documento:

```text
Leia TRANSCRICAO.md inteira e classifique CADA item técnico ou de produto em uma
destas categorias, com timestamp e falante:
1. Decisão fechada (alguém disse "decidido", "anotado" ou houve concordância explícita)
2. Alternativa descartada (e o motivo do descarte, com quem argumentou)
3. Fora de escopo ou adiado para fase futura
4. Questão em aberto (levantada e não decidida)
5. Requisito funcional ou não funcional
6. Ponto de integração com o código (confirme que o arquivo existe em src/)
Não promova nada das categorias 2, 3 ou 4 a requisito. Se um item estiver ambíguo,
coloque em 4 e explique a ambiguidade.
```

### 3. Verificação de rastreabilidade

```bash
# todo [hh:mm] Nome citado precisa existir na transcrição com aquele falante
grep -ohE '\[[0-9]{2}:[0-9]{2}\] (Larissa|Marcos|Bruno|Diego|Sofia)' -r docs README.md | sort -u \
  | while read -r ts; do grep -qF "$ts:" TRANSCRICAO.md || echo "MISSING: $ts"; done

# todo caminho de código citado precisa existir (exceto os arquivos novos propostos)
grep -ohE '(src|prisma|tests)/[A-Za-z0-9_./-]+\.(ts|prisma)|package\.json' -r docs | sort -u \
  | while read -r p; do test -e "$p" || echo "NAO EXISTE: $p"; done
```

---

## Iterações e ajustes

Foram **5 iterações principais** depois da primeira geração.

1. **"5 tentativas" não fechava com "quase 15 horas".** A primeira versão do ADR-003 dizia "5 tentativas" e parava aí. Ao montar a tabela de retry do FDD, a conta não fechava: se a 1ª das 5 tentativas é o envio inicial, sobram só 4 intervalos (1m+5m+30m+2h ≈ 2h36m), e não os "quase 15 horas" que o Diego citou em [09:17]. A única leitura coerente é 1 envio inicial + 5 retentativas (14h36m). Registrei essa interpretação explicitamente no ADR-003 e no FDD, marcada para confirmação, em vez de escolher em silêncio.

2. **O código contradizia o desenho de erros.** O primeiro rascunho da matriz de erros previa `WEBHOOK_INVALID_URL` como código de topo da resposta e `WebhookNotFoundError extends NotFoundError`. Lendo `src/middlewares/validate.middleware.ts` e `src/shared/errors/http-errors.ts`, vi que o `validate()` converte todo `ZodError` em `VALIDATION_ERROR` e que `NotFoundError` fixa o código `NOT_FOUND`. O FDD foi corrigido: o código `WEBHOOK_*` de validação aparece em `details`, e os 404 do módulo estendem `AppError` diretamente. Assim, nenhuma alteração em middleware compartilhado é necessária.

3. **Observabilidade inventada foi rebaixada a hipótese.** O FDD exige métricas, logs e tracing, mas a reunião só fala de Pino e de auditoria do replay, e o projeto não tem biblioteca de métricas nem de tracing. Em vez de "adicionar OpenTelemetry e Prometheus", que contraria o ADR-006 (reuso, nada novo), a seção ficou com logs reais (Pino, `requestId` existente), métricas derivadas das tabelas e correlação por `event_id`, tudo marcado como **[HIPÓTESE]** e listado no tracker.

4. **Inconsistência de modelo encontrada na revisão.** O FDD propunha DELETE como remoção lógica via `active = false`, o mesmo campo do `PATCH active=false`. Com isso, um webhook pausado e um removido ficariam indistinguíveis. Foi adicionado `deletedAt`, com a semântica de cada operação descrita.

5. **Auditoria contra os critérios de aceite e as dicas.** Pedi à IA para conferir a entrega item por item e ela encontrou problemas que a primeira versão escondia:
   - **Hipóteses dispensáveis:** havia 99,9% de disponibilidade no PRD (default do prompt, não da reunião), uma contingência inventada para a Atlas, dois códigos de erro sem origem (`WEBHOOK_ROTATION_IN_PROGRESS`, `WEBHOOK_DEAD_LETTER_ALREADY_REPLAYED`) e um limite de 4 KB arbitrário. Tudo foi removido ou trocado por comportamento derivado da transcrição. Exemplo: o replay só vale para eventos em `FAILED`, e uma nova falha faz upsert na DLQ.
   - **Riscos duplicados:** os mesmos riscos apareciam no PRD, no RFC e no FDD. Cada documento ficou com os da sua altura: negócio no PRD, arquitetura no RFC, implementação no FDD.
   - **Títulos implícitos:** seções obrigatórias que estavam como texto simples ("Metadados" no RFC; "Problema e motivação" e "Público-alvo e cenários de uso" no PRD) viraram títulos.
   - **Exemplos vazios:** os exemplos de request `{}` dos GET e DELETE viraram requisições HTTP reais.
   - **Tracker:** vínculos fracos foram trocados por fontes melhores. `WEBHOOK_CUSTOMER_NOT_FOUND`, por exemplo, passou a apontar para a checagem que já existe em `order.service.ts`. A tabela de hipóteses passou a usar o mesmo formato de 6 colunas.

Ajustes menores: o script de verificação confirmou que todos os timestamps citados existem com o falante correto. Também mostrou que os únicos caminhos "inexistentes" eram os arquivos novos propostos (`src/worker.ts`, `src/modules/webhooks/*`), que passaram a aparecer rotulados como "a criar". O RFC ficou sem schemas, payloads e matriz de erros, que ficam só no FDD, para não duplicar níveis.

---

## Como navegar a entrega

Ordem sugerida de leitura:

1. **[docs/PRD.md](docs/PRD.md)**: por que e o quê. Problema, público, métricas, escopo e fora de escopo, 10 requisitos funcionais.
2. **[docs/RFC.md](docs/RFC.md)**: a proposta técnica para revisão. Visão geral, 5 alternativas descartadas, 6 questões em aberto.
3. **[docs/adrs/](docs/adrs/)**: uma decisão por arquivo.
   - [ADR-001](docs/adrs/ADR-001-outbox-no-mysql.md) Outbox no MySQL
   - [ADR-002](docs/adrs/ADR-002-worker-separado-em-polling.md) Worker separado em polling
   - [ADR-003](docs/adrs/ADR-003-retry-backoff-e-dlq.md) Retry com backoff e DLQ
   - [ADR-004](docs/adrs/ADR-004-hmac-sha256-secret-por-endpoint.md) HMAC-SHA256 com secret por endpoint
   - [ADR-005](docs/adrs/ADR-005-at-least-once-com-x-event-id.md) At-least-once com X-Event-Id
   - [ADR-006](docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md) Reuso dos padrões do projeto
   - [ADR-007](docs/adrs/ADR-007-payload-snapshot-e-filtro-na-insercao.md) Payload snapshot e filtro na inserção
4. **[docs/FDD.md](docs/FDD.md)**: como construir. Modelo de dados, fluxos, 8 contratos com exemplos, matriz `WEBHOOK_*`, observabilidade e a seção 11, "Integração com o sistema existente".
5. **[docs/TRACKER.md](docs/TRACKER.md)**: a origem de cada item, mais a lista de hipóteses explícitas.
6. **[docs/processo/](docs/processo/)**: registros das entrevistas de PRD e FDD respondidas pela transcrição.

```
.
├── README.md                 # este arquivo
├── TRANSCRICAO.md            # fonte (inalterada)
└── docs/
    ├── PRD.md
    ├── RFC.md
    ├── FDD.md
    ├── TRACKER.md
    ├── adrs/
    │   ├── README.md         # índice dos ADRs
    │   └── ADR-001 … ADR-007
    └── processo/
        ├── entrevista-prd.md
        └── entrevista-fdd.md
```

O código da aplicação (`src/`, `prisma/`, `tests/`) não foi alterado: a entrega é só documentação.
