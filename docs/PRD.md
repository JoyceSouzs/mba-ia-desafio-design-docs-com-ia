### PRD: Order Management System (OMS) Sistema de Webhooks de Notificação de Pedidos

Versão: 1.0
Data: 2026-10-07
Responsável: Marcos (Product Manager)

> Documentos ligados: [RFC](RFC.md) (proposta técnica e riscos de arquitetura), [ADRs](adrs/) (decisões), [FDD](FDD.md) (implementação e riscos técnicos), [Tracker](TRACKER.md) (origem de cada item). Itens marcados **[HIPÓTESE]** não foram definidos na reunião e estão explícitos para validação.

---

### Resumo

Clientes B2B do OMS precisam saber quando o status dos seus pedidos muda sem ter que consultar a API repetidamente. Esta feature envia uma notificação HTTP (webhook) para um endpoint cadastrado pelo cliente sempre que um pedido dele muda de status, em menos de 10 segundos, de forma segura e verificável. O cliente escolhe quais status quer receber, acompanha o histórico de entregas e pode rotacionar sua credencial. A entrega é resiliente a indisponibilidades do cliente e nunca atrasa nem bloqueia a operação de pedidos.

---

### Contexto e problema

#### Contexto da feature

Onde essa feature será implantada
- No OMS existente (Node.js, TypeScript, Express, Prisma, MySQL), como novo módulo `src/modules/webhooks` seguindo o padrão dos demais ([09:27] Bruno) e um novo processo worker que roda separado da API, no mesmo banco ([09:11] Diego, [09:11] Larissa)
- Hoje a aplicação não tem eventos, filas nem webhooks; a mudança de status de pedidos acontece em `src/modules/orders/order.service.ts`
- A notificação é só de saída: a plataforma envia, o cliente recebe ([09:02] Marcos, [09:03] Sofia)

#### Problema e motivação

Problemas priorizados
- Clientes fazem polling em `GET /orders` para descobrir mudanças de status. Impacto: integração lenta e cara para o cliente. Prioridade: alta ([09:00] Marcos)
- Risco comercial: a Atlas Comercial indicou que pode migrar para um concorrente se a funcionalidade não for entregue até o fim do trimestre, com pedido de entrega até o fim de novembro. Impacto: perda de cliente. Prioridade: alta ([09:00] Marcos, [09:45] Marcos)
- O OMS não possui nenhum mecanismo de eventos ou notificações. Impacto: nenhuma alternativa disponível ao polling. Prioridade: alta (`src/app.ts`)

#### Público-alvo e cenários de uso

Público-alvo
- Clientes B2B que integram com o OMS via API, começando por Atlas Comercial, MaxDistribuição e Nova Cargo ([09:00] Marcos)
- Usuários do OMS que representam esses clientes e gerenciam a integração pela API, autenticados com JWT ([09:32] Marcos)
- Administradores (role ADMIN) responsáveis por reprocessar notificações que falharam definitivamente ([09:36] Sofia)

Cenários de uso chave
- O cliente cadastra a URL do seu sistema e escolhe ser avisado só quando o pedido vira SHIPPED e DELIVERED ([09:33] Marcos)
- Um operador muda o pedido para SHIPPED e o sistema do cliente é notificado em poucos segundos, sem polling ([09:02] Marcos)
- O sistema do cliente fica fora do ar por 2 horas em manutenção planejada e recebe as notificações atrasadas quando volta ([09:16] Diego)
- O cliente suspeita que vazou sua secret, pede uma nova e tem 24 horas para atualizar seus sistemas ([09:21] Sofia, [09:22] Diego)
- O cliente consulta os últimos 100 envios para investigar uma notificação que não processou ([09:34] Marcos)
- Um ADMIN reprocessa uma notificação que esgotou as tentativas depois que o cliente corrigiu seu endpoint ([09:18] Diego)

---

### Objetivos e métricas de sucesso

| Objetivo | Métrica | Meta |
| --- | --- | --- |
| Notificar o cliente em "tempo real" | Tempo entre a mudança de status e a primeira tentativa de entrega (p95) | Menor que 10 segundos ([09:02] Marcos) |
| Substituir o polling dos clientes que pediram a feature | Clientes solicitantes recebendo webhooks em produção | 3 de 3: Atlas, MaxDistribuição e Nova Cargo ([09:00] Marcos) |
| Entregar no prazo pedido pela Atlas | Sprints até a liberação em produção, incluindo revisão de segurança | 3 sprints, até o fim de novembro ([09:45] Marcos, [09:47] Larissa) |
| Nenhuma mudança de status sem notificação correspondente | Mudanças de status com webhook interessado e sem evento registrado | 0 ([09:40] Bruno, [09:41] Diego) |
| Resiliência a indisponibilidade do cliente | Janela coberta por novas tentativas automáticas antes de exigir ação manual | Aproximadamente 15 horas ([09:17] Diego) |

---

### Escopo (incluso e fora de escopo)

Incluso
- Cadastro, listagem, edição e remoção de webhooks por cliente via API ([09:31] Marcos, [09:33] Bruno)
- Escolha, por webhook, de quais status de pedido disparam notificação ([09:33] Marcos)
- Notificação automática a cada mudança de status de pedido ([09:40] Bruno)
- Assinatura de cada notificação para o cliente verificar origem e integridade ([09:20] Sofia)
- Rotação da secret com 24 horas de convivência entre a antiga e a nova ([09:21] Sofia)
- Novas tentativas automáticas e fila de falhas definitivas (DLQ) ([09:17] Larissa)
- Reprocessamento manual de falhas definitivas por ADMIN ([09:35] Diego, [09:36] Sofia)
- Histórico das últimas 100 entregas por webhook ([09:34] Marcos)
- Documentação de integração no portal de desenvolvedor ([09:26] Marcos, [09:40] Marcos)

Fora de escopo
- Email para o cliente quando o webhook falha repetidamente: adiado para a próxima fase, depois de medir o impacto ([09:37] Marcos, [09:37] Larissa)
- Dashboard ou painel visual para o cliente acompanhar seus webhooks: projeto separado do time de frontend ([09:39] Marcos, [09:40] Larissa)
- Rate limiting de envio por cliente: observar e decidir depois ([09:38] Diego, [09:39] Larissa)
- Arquivamento de notificações entregues após cerca de 30 dias ([09:08] Diego)
- Webhooks de entrada (cliente enviando para o OMS): só saída ([09:02] Marcos)
- Garantia de ordem global entre pedidos e processamento paralelo com múltiplos workers ([09:13] Diego, [09:13] Larissa, [09:14] Marcos)
- Entrega exactly-once: a garantia é at-least-once ([09:25] Diego)
- Itens do pedido no conteúdo da notificação: o cliente consulta `GET /orders/:id` se precisar ([09:43] Diego)

---

### Requisitos funcionais

#### FR-001 Cadastrar webhook
O usuário autenticado cadastra um endpoint de notificação para um cliente, informando a URL e os status que deseja receber; a plataforma gera e devolve a secret.

**Fluxo principal**
- O usuário envia URL, lista de status e o cliente ao qual o webhook pertence ([09:31] Marcos, [09:32] Larissa)
- O sistema valida que a URL usa https ([09:23] Sofia)
- O sistema gera uma secret única para esse endpoint ([09:21] Sofia, [09:31] Marcos)
- O sistema responde com o webhook criado e a secret

**Fluxos alternativos e exceções**
- Um mesmo cliente pode ter vários webhooks; cada envio identifica de qual cadastro veio ([09:44] Sofia)
- O cliente é informado no body ou no path, não é extraído do JWT; a forma final está em aberto ([09:32] Larissa)

**Erros previstos**
- URL sem https é recusada com erro de validação ([09:23] Sofia)
- Cliente inexistente é recusado
- Requisição sem autenticação é recusada

**Prioridade:** alta

---

#### FR-002 Listar webhooks de um cliente
O usuário consulta os webhooks cadastrados de um cliente.

**Fluxo principal**
- O usuário informa o cliente ([09:33] Bruno)
- O sistema retorna os webhooks com URL, status escolhidos e estado ativo, sem expor a secret

**Fluxos alternativos e exceções**
- Cliente sem webhooks retorna lista vazia

**Erros previstos**
- Cliente não informado ou inválido é recusado

**Prioridade:** alta

---

#### FR-003 Editar webhook
O usuário altera a URL, os status escolhidos ou ativa e desativa um webhook.

**Fluxo principal**
- O usuário envia os campos a alterar ([09:33] Bruno)
- O sistema valida e salva

**Fluxos alternativos e exceções**
- A mudança de status escolhidos vale para as próximas mudanças de pedido; notificações já geradas são enviadas como estavam ([09:34] Bruno)

**Erros previstos**
- Webhook inexistente retorna erro `WEBHOOK_NOT_FOUND` ([09:28] Bruno)
- URL sem https é recusada ([09:23] Sofia)

**Prioridade:** alta

---

#### FR-004 Remover webhook
O usuário remove um webhook e ele deixa de receber notificações.

**Fluxo principal**
- O usuário pede a remoção ([09:33] Bruno)
- O sistema remove o webhook e para de gerar notificações para ele

**Fluxos alternativos e exceções**
- Para só pausar o envio sem remover, o usuário desativa o webhook pela edição (FR-003), usando o estado ativo do cadastro ([09:21] Bruno)

**Erros previstos**
- Webhook inexistente retorna erro `WEBHOOK_NOT_FOUND`

**Prioridade:** alta

---

#### FR-005 Rotacionar secret
O cliente pede uma nova secret pela API, e a antiga continua válida por 24 horas para dar tempo de migrar.

**Fluxo principal**
- O usuário pede nova secret para um webhook ([09:21] Sofia)
- O sistema gera a nova secret e a devolve
- Por 24 horas, as notificações continuam verificáveis com a secret antiga ([09:21] Sofia)
- Depois de 24 horas, só a nova vale

**Fluxos alternativos e exceções**
- Uso típico: secret vazada em log da aplicação do cliente ([09:22] Diego)

**Erros previstos**
- Webhook inexistente retorna erro `WEBHOOK_NOT_FOUND`

**Prioridade:** alta

---

#### FR-006 Gerar notificação na mudança de status do pedido
Toda mudança de status de pedido gera notificação para os webhooks do cliente que escolheram aquele status, de forma inseparável da própria mudança.

**Fluxo principal**
- Um operador muda o status do pedido
- O sistema identifica os webhooks ativos do cliente interessados no novo status ([09:34] Bruno)
- O sistema registra a notificação junto com a mudança de status: ou as duas acontecem, ou nenhuma ([09:40] Bruno, [09:41] Diego)
- O conteúdo da notificação reflete o pedido no momento da mudança ([09:52] Larissa)

**Fluxos alternativos e exceções**
- Nenhum webhook interessado: nada é registrado ([09:34] Bruno)
- A mudança de status nunca espera a resposta do cliente ([09:04] Bruno)

**Erros previstos**
- Falha ao registrar a notificação cancela a mudança de status ([09:40] Bruno)

**Prioridade:** alta

---

#### FR-007 Entregar notificação assinada
A plataforma envia a notificação ao endpoint do cliente com identificação única e assinatura verificável.

**Fluxo principal**
- O sistema verifica novas notificações a cada 2 segundos ([09:10] Larissa)
- Envia para a URL do cliente o conteúdo enxuto do pedido: identificador do evento, tipo, data e hora, pedido, número, status anterior, novo status, cliente e total ([09:43] Diego)
- Envia identificador do evento, identificador do webhook, horário de envio e assinatura ([09:44] Diego, [09:44] Sofia)
- Resposta de sucesso do cliente encerra a entrega

**Fluxos alternativos e exceções**
- O mesmo evento pode chegar mais de uma vez; o cliente descarta repetidos pelo identificador do evento ([09:24] Diego, [09:25] Diego)
- Eventos do mesmo pedido chegam em ordem; não há ordem garantida entre pedidos diferentes ([09:13] Larissa)

**Erros previstos**
- Cliente sem resposta em 10 segundos conta como falha ([09:42] Diego)
- Notificação acima de 64 KB não é enviada e é tratada como erro ([09:24] Larissa)

**Prioridade:** alta

---

#### FR-008 Retentar entregas com falha
Quando a entrega falha, o sistema tenta de novo com intervalos crescentes antes de desistir.

**Fluxo principal**
- Após falha, nova tentativa em 1 minuto, depois 5 minutos, 30 minutos, 2 horas e 12 horas ([09:17] Diego)
- Sucesso em qualquer tentativa encerra a entrega
- Esgotadas as 5 tentativas, a notificação vai para a fila de falhas definitivas (DLQ) ([09:15] Diego)

**Fluxos alternativos e exceções**
- Cliente fora do ar por mais de cerca de 15 horas exige reprocessamento manual ([09:17] Marcos)

**Erros previstos**
- Falha definitiva registrada com motivo e horário ([09:18] Diego)

**Prioridade:** alta

---

#### FR-009 Reprocessar falhas definitivas (replay da DLQ)
Um ADMIN recoloca uma notificação da DLQ para nova entrega.

**Fluxo principal**
- O ADMIN pede o reprocessamento de um item da DLQ ([09:35] Diego)
- O sistema recoloca a notificação como pendente, com o mesmo identificador de evento ([09:18] Diego)
- O sistema registra quem fez o reprocessamento ([09:36] Sofia)

**Fluxos alternativos e exceções**
- Se o cliente continuar fora do ar, a notificação reprocessada volta ao ciclo normal de tentativas, porque o replay a recoloca como pendente ([09:18] Diego)

**Erros previstos**
- Usuário sem role ADMIN é recusado ([09:36] Sofia, [09:36] Larissa)
- Item inexistente na DLQ retorna erro

**Prioridade:** alta

---

#### FR-010 Consultar histórico de entregas
O cliente vê as últimas entregas de um webhook para investigar problemas.

**Fluxo principal**
- O usuário consulta o histórico de um webhook ([09:34] Marcos)
- O sistema retorna as últimas 100 tentativas com resultado (sucesso ou falha), conteúdo enviado, resposta recebida e tempo de resposta ([09:34] Marcos)

**Fluxos alternativos e exceções**
- Webhook sem envios retorna lista vazia

**Erros previstos**
- Webhook inexistente retorna erro `WEBHOOK_NOT_FOUND`

**Prioridade:** media

---

### Requisitos não funcionais

Performance
- Notificação em menos de 10 segundos (p95) após a mudança de status ([09:02] Marcos)
- Espera máxima de 2 segundos até o sistema perceber a nova notificação ([09:10] Larissa)
- Timeout de 10 segundos por tentativa de entrega ([09:42] Diego)
- Notificação limitada a 64 KB ([09:24] Larissa)
- A mudança de status de pedidos não pode ficar mais lenta por causa de clientes lentos ([09:04] Bruno)

Disponibilidade
- Indisponibilidade de um cliente não afeta a mudança de status nem os demais clientes ([09:04] Bruno)
- O processo de envio roda separado da API e não cai quando a API reinicia ([09:11] Diego)
- Nenhuma meta percentual de uptime foi definida na reunião; a garantia acordada é que eventos não se perdem enquanto o worker estiver parado, porque ficam na outbox ([09:06] Diego)

Segurança e autorização
- Assinatura HMAC-SHA256 do conteúdo enviado ([09:20] Sofia, [09:22] Sofia)
- Secret única por webhook, nunca global ([09:21] Sofia)
- Rotação de secret com 24 horas de convivência ([09:21] Sofia)
- Somente URLs https ([09:23] Sofia)
- Gestão de webhooks para qualquer usuário autenticado nesta fase ([09:37] Sofia)
- Reprocessamento da DLQ restrito a ADMIN ([09:36] Sofia)
- Revisão de segurança antes do deploy ([09:46] Sofia)

Observabilidade
- Histórico de entregas consultável pelo cliente ([09:34] Marcos)
- Logs estruturados no logger já usado pelo projeto (Pino) ([09:29] Bruno)
- Registro de quem reprocessou cada item da DLQ ([09:36] Sofia)

Confiabilidade e integridade de dados
- Mudança de status e registro da notificação são atômicos ([09:06] Diego, [09:40] Bruno)
- Entrega pelo menos uma vez (at-least-once), com identificador único por evento para descarte de repetidos ([09:24] Diego, [09:25] Diego)
- Ordem garantida por pedido enquanto houver um único processo de envio ([09:13] Larissa)
- Conteúdo da notificação congelado no momento da mudança ([09:52] Larissa)

Compatibilidade e portabilidade
- API REST JSON no prefixo `/api/v1` já existente (`src/app.ts`)
- Mesma stack e mesmo banco, sem infraestrutura nova ([09:07] Diego, [09:11] Diego)
- Nenhum contrato existente da API muda

Compliance
- Secret nunca aparece em logs ([09:22] Diego)
- Trilha de auditoria do reprocessamento manual ([09:36] Sofia)

Acessibilidade no frontend consumidor
- Não se aplica nesta fase: não há interface visual; o painel está fora de escopo ([09:40] Larissa)

---

### Arquitetura e abordagem

Abordagem
- Padrão outbox: a notificação é gravada em uma tabela do MySQL dentro da mesma transação da mudança de status, e um worker separado lê essa tabela e faz as entregas ([09:06] Diego, [09:08] Larissa)

Componentes
- Módulo `src/modules/webhooks` na API, com o padrão de controller, service, repository, routes e schemas ([09:27] Bruno)
- Tabelas de configuração de webhooks, outbox, histórico de entregas e DLQ no MySQL existente ([09:08] Diego, [09:18] Diego, [09:34] Marcos)
- Worker em processo separado (`src/worker.ts`, `npm run worker`) com polling de 2 segundos ([09:11] Larissa, [09:10] Larissa)

Integrações
- `OrderService.changeStatus` registra a notificação na mesma transação ([09:40] Bruno, [09:41] Bruno)
- Endpoints HTTPS dos clientes recebem as notificações ([09:23] Sofia)
- Portal de desenvolvedor documenta assinatura, deduplicação e contrato ([09:26] Marcos)

### Decisões e trade-offs principais

#### Decisão: outbox no MySQL existente ([ADR-001](adrs/ADR-001-outbox-no-mysql.md))
- **Justificativa:** garante que mudança de status e notificação andem juntas, sem subir infraestrutura nova para um time pequeno ([09:06] Diego, [09:07] Diego)
- **Trade-off:** entrega assíncrona e tabela que cresce com o tempo; arquivamento fica para depois ([09:08] Diego)

#### Decisão: worker separado com polling de 2 segundos ([ADR-002](adrs/ADR-002-worker-separado-em-polling.md))
- **Justificativa:** atende o limite de 10 segundos com folga e isola o envio da API ([09:09] Diego, [09:11] Diego)
- **Trade-off:** até 2 segundos de espera e ordem garantida só por pedido ([09:10] Larissa, [09:13] Larissa)

#### Decisão: 5 tentativas com backoff e DLQ ([ADR-003](adrs/ADR-003-retry-backoff-e-dlq.md))
- **Justificativa:** cobre indisponibilidades de até cerca de 15 horas sem deixar eventos pendurados para sempre ([09:15] Diego, [09:16] Diego)
- **Trade-off:** indisponibilidades maiores exigem reprocessamento manual ([09:17] Marcos)

#### Decisão: HMAC-SHA256 com secret por webhook ([ADR-004](adrs/ADR-004-hmac-sha256-secret-por-endpoint.md))
- **Justificativa:** padrão de mercado; um vazamento afeta só um webhook ([09:20] Sofia, [09:21] Sofia)
- **Trade-off:** o cliente precisa implementar a verificação e gerenciar a rotação

#### Decisão: at-least-once com `X-Event-Id` ([ADR-005](adrs/ADR-005-at-least-once-com-x-event-id.md))
- **Justificativa:** exactly-once seria muito mais complexo; padrão usado por Stripe e GitHub ([09:25] Diego)
- **Trade-off:** o cliente precisa descartar repetidos ([09:25] Sofia)

#### Decisão: reuso dos padrões do projeto ([ADR-006](adrs/ADR-006-reuso-dos-padroes-do-projeto.md))
- **Justificativa:** consistência e menos superfície nova ([09:30] Larissa)
- **Trade-off:** herda limitações atuais, como a ausência de métricas e tracing no projeto

#### Decisão: conteúdo congelado e filtro na origem ([ADR-007](adrs/ADR-007-payload-snapshot-e-filtro-na-insercao.md))
- **Justificativa:** a notificação reflete o pedido no momento da mudança e não se gravam eventos que ninguém quer ([09:52] Larissa, [09:34] Bruno)
- **Trade-off:** mudar o filtro não afeta notificações já geradas

---

### Dependências

#### Organizacional: revisão de segurança
Sofia precisa de pelo menos 2 dias úteis para revisar o código de HMAC e de geração de secret antes do deploy; a revisão está incluída na estimativa de 3 sprints ([09:46] Sofia, [09:47] Larissa).

#### Organizacional: documentação no portal de desenvolvedor
Marcos documenta no portal como integrar via API, com destaque para a deduplicação por `X-Event-Id`, antes da liberação para os clientes ([09:26] Marcos, [09:40] Marcos).

#### Externa: alinhamento de prazo com a Atlas
Marcos confirma com a Atlas o prazo de fim de novembro e atualiza os três clientes ([09:47] Marcos, [09:49] Marcos).

#### Externa: endpoints dos clientes
Cada cliente precisa expor um endpoint https, validar a assinatura e descartar eventos repetidos ([09:23] Sofia, [09:25] Diego).

#### Técnica: execução do worker em produção
O ambiente precisa rodar um segundo processo (`npm run worker`), separado da API, com acesso ao mesmo banco ([09:11] Diego, [09:11] Larissa).

#### Técnica: revisão de design
Sessão de revisão do desenho com Bruno e Diego antes do início da codificação ([09:50] Larissa).

---

### Riscos e mitigação

#### Atraso na entrega leva à perda da Atlas Comercial
- **Probabilidade:** media
- **Impacto:** perda de um cliente B2B que sinalizou migração para concorrente ([09:00] Marcos)
- **Mitigação:**
  - Escopo enxuto, com email, dashboard e rate limiting fora desta fase ([09:37] Larissa, [09:40] Larissa, [09:39] Larissa)
  - Estimativa de 3 sprints com a revisão de segurança já incluída ([09:46] Larissa, [09:47] Larissa)
  - Marcos confirma o prazo com a Atlas ([09:47] Marcos)
- **Plano de contingência:** Marcos, que é o ponto de contato com os clientes, comunica qualquer desvio de prazo à Atlas assim que ele for identificado ([09:47] Marcos, [09:49] Marcos)

#### Clientes não se adaptam à entrega duplicada e processam a mesma mudança duas vezes
- **Probabilidade:** media
- **Impacto:** ação duplicada no sistema do cliente e percepção de que a integração não é confiável ([09:25] Sofia)
- **Mitigação:**
  - Explicação em destaque no portal de desenvolvedor ([09:26] Marcos)
  - Identificador único por evento, padrão que os clientes já conhecem de Stripe e GitHub ([09:25] Diego)
- **Plano de contingência:** usar o histórico de entregas para mostrar ao cliente quais eventos foram reenviados ([09:34] Marcos)

#### Cliente fica fora do ar por mais de 15 horas
- **Probabilidade:** baixa
- **Impacto:** notificações vão para a DLQ e o cliente não é avisado automaticamente ([09:17] Marcos, [09:37] Larissa)
- **Mitigação:**
  - Janela de cerca de 15 horas de novas tentativas ([09:17] Diego)
  - Histórico de entregas visível ao cliente ([09:34] Marcos)
- **Plano de contingência:** ADMIN reprocessa a DLQ depois que o cliente se recupera ([09:18] Diego)

---

### Critérios de aceitação
Checklist objetivo que define se a feature está pronta.

- Um usuário autenticado cadastra um webhook com URL https e lista de status e recebe a secret na resposta
- Cadastro com URL http é recusado com erro de validação
- Listagem, edição e remoção funcionam e nenhuma resposta além do cadastro e da rotação expõe a secret
- Uma mudança de status de pedido para um status escolhido gera notificação recebida pelo cliente em menos de 10 segundos (p95)
- Uma mudança para um status não escolhido não gera notificação
- Se o registro da notificação falhar, a mudança de status é desfeita
- Toda notificação traz `X-Event-Id`, `X-Webhook-Id`, `X-Timestamp` e `X-Signature`, e a assinatura confere com HMAC-SHA256 do corpo usando a secret do webhook
- Um endpoint do cliente indisponível recebe novas tentativas em 1m, 5m, 30m, 2h e 12h; depois disso a notificação aparece na DLQ com motivo e horário
- Um ADMIN reprocessa um item da DLQ e o evento é reenviado com o mesmo identificador; um usuário sem ADMIN é recusado; o log registra quem reprocessou
- Após rotação, notificações validam com a secret antiga e com a nova por 24 horas e só com a nova depois disso
- O histórico de entregas mostra as últimas 100 tentativas com resultado, conteúdo, resposta e tempo de resposta
- Reiniciar a API não interrompe o envio de notificações
- Revisão de segurança da Sofia concluída antes do deploy
- Portal de desenvolvedor publicado com assinatura, deduplicação e contrato da notificação

---

### Estratégia de testes e validação

Tipos de teste obrigatórios
- Testes unitários para assinatura HMAC, cálculo dos intervalos de retry, filtro de status e limite de 64 KB
- Testes de integração para o fluxo completo de mudança de status até o registro da notificação, incluindo o cancelamento da mudança quando o registro falha ([09:46] Larissa)
- Testes de integração da API de webhooks: cadastro, listagem, edição, remoção, rotação, histórico e permissão de ADMIN no reprocessamento
- Testes do worker contra um endpoint simulado com sucesso, erro e demora acima de 10 segundos
- Revisão de segurança manual do código de HMAC e de geração de secret ([09:46] Sofia)

Estratégia de validação
- Revisão do desenho com Bruno e Diego antes de começar a codar ([09:50] Larissa)
- Testes automatizados com Vitest e Supertest, no padrão já usado em `tests/orders.test.ts`
- Testes ponta a ponta da integração com `order.service` dentro da estimativa de 3 sprints ([09:46] Larissa)
- Liberação para Atlas, MaxDistribuição e Nova Cargo com acompanhamento do histórico de entregas e da DLQ
