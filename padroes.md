Índice
------

- [Histórico de Revisões](#histórico-de-revisões)
- [**1. Introdução**](#1-introdução)
    - [**1.1 Propósito e Objetivo**](#11-propósito-e-objetivo)
    - [**1.2 Público-Alvo**](#12-público-alvo)
    - [**1.3 Serviços Abrangidos**](#13-serviços-abrangidos)
    - [**1.4 Âmbito**](#14-âmbito)
- [**2. Dependências e Referências**](#2-dependências-e-referências)
    - [2.1 Dependências Técnicas](#21-dependências-técnicas)
- [**3. Regras de Utilização**](#3-regras-de-utilização)
- [**4. Arquitetura da Tipologia**](#4-arquitetura-da-tipologia)
    - [**4.1 Contexto**](#41-contexto)
    - [**4.2 Diagrama de Arquitetura Lógica**](#42-diagrama-de-arquitetura-lógica)
    - [**4.3 Diagramas de Sequência Detalhados**](#43-diagramas-de-sequência-detalhados)
        - [4.3.1 Scenario 1 — Legacy 1 (pull/polling direto pelo Job Worker)](#431-scenario-1-legacy-1-pullpolling-direto-pelo-job-worker)
        - [4.3.2 Scenario 2 — Ideal (KEDA escala pods; worker faz polling)](#432-scenario-2-ideal-keda-escala-pods-worker-faz-polling)
        - [4.3.3 Scenario 3 — Legacy 2 (Event Gateway faz "push" para endpoint HTTP do worker)](#433-scenario-3-legacy-2-event-gateway-faz-push-para-endpoint-http-do-worker)
    - [**4.4 Princípios Arquiteturais**](#44-princípios-arquiteturais)
    - [**4.5 Casos de Uso**](#45-casos-de-uso)
    - [**4.6. Estratégias de Teste**](#46-estratégias-de-teste)
    - [**4.7 Estratégias de Ingestão e Persistência**](#47-estratégias-de-ingestão-e-persistência)
        - [4.7.1 Processamento Event-Driven (Streaming)](#471-processamento-event-driven-streaming)
        - [4.7.2 Processamento de eventos em batch (Batch de eventos / Windowing)](#472-processamento-de-eventos-em-batch-batch-de-eventos-windowing)
        - [4.7.3 Scheduling/trigger de processamento de eventos em batch](#473-schedulingtrigger-de-processamento-de-eventos-em-batch)
        - [4.7.4 Scheduling/trigger de processamento de dados persistidos em outros meios](#474-schedulingtrigger-de-processamento-de-dados-persistidos-em-outros-meios)
        - [4.7.5 Granularidade de ingestão: linha-a-linha vs batch/bulk](#475-granularidade-de-ingestão-linha-a-linha-vs-batchbulk)
    - [4.8 Controle de Carga, Throttling e Escalabilidade](#48-controle-de-carga-throttling-e-escalabilidade)
        - [4.8.1 Throttling baseado em mensageria](#481-throttling-baseado-em-mensageria)
        - [Mecanismo de Backpressure](#mecanismo-de-backpressure)
            - [Controle de prefetch / pull-based consumption](#controle-de-prefetch-pull-based-consumption)
            - [Acknowledgement como válvula de pressão](#acknowledgement-como-válvula-de-pressão)
            - [Consumer Lag como indicador de saturação](#consumer-lag-como-indicador-de-saturação)
            - [Limites físicos e lógicos do broker](#limites-físicos-e-lógicos-do-broker)
            - [Estratégias de throttling usando backpressure](#estratégias-de-throttling-usando-backpressure)
        - [4.8.2 Escalabilidade horizontal](#482-escalabilidade-horizontal)
        - [4.8.3 Proteção de sistemas downstream](#483-proteção-de-sistemas-downstream)
        - [4.8.4. Quando usar Hangfire](#484-quando-usar-hangfire)
- [**5. Estrutura do Projeto**](#5-estrutura-do-projeto)
- [**6. Tecnologias Utilizadas**](#6-tecnologias-utilizadas)
    - [6.1. Uso de NoSQL com Hangfire](#61-uso-de-nosql-com-hangfire)
- [**7. Segurança**](#7-segurança)
    - [7.1 Requisitos Mínimos Obrigatórios](#71-requisitos-mínimos-obrigatórios)
- [**8. Infraestrutura**](#8-infraestrutura)
    - [**8.2. CI/CD e DevOps**](#82-cicd-e-devops)
- [**9. Padrões e Princípios Arquiteturais**](#9-padrões-e-princípios-arquiteturais)
- [9.1. Referências dos Princípios (R1–R7)](#91-referências-dos-princípios-r1r7)
- [**10. Observabilidade**](#10-observabilidade)
    - [10.1 Requisitos Mínimos Obrigatórios](#101-requisitos-mínimos-obrigatórios)
    - [10.2 Métricas, Correlação e Alertas](#102-métricas-correlação-e-alertas)
    - [10.3 Monitorização de SLA por Integração](#103-monitorização-de-sla-por-integração)

# DAS Batch Job Worker — Diretrizes Técnicas e Arquiteturais

> **Detalhe:** Middleware entre sistemas, transformação de dados, sync jobs.

---

## Histórico de Revisões

*(A preencher com datas e autores)*

---

## **1. Introdução**

### **1.1 Propósito e Objetivo**

Este documento estabelece diretrizes técnicas e arquiteturais para desenvolvimento, manutenção e documentação de sistemas de processamento assíncrono em batch, incluindo ETL, agendamento com Hangfire, cron jobs e workers distribuídos, aplicações containerizadas em Kubernetes, cloud-native.

O objetivo é mitigar problemas críticos como alto acoplamento, volume elevado de dados, falta de paginação e baixa observabilidade, garantindo soluções escaláveis, resilientes e seguras.

---

### **1.2 Público-Alvo**

- Engenharia
- Suporte
- Arquitetura

---

### **1.3 Serviços Abrangidos**

- Agendamento e execução periódica de jobs
- Pipelines de ETL (Extract, Transform, Load)
- Processamentos distribuídos de alto volume
- Integrações com sistemas externos
- Processamentos baseados em filas/mensageria
- Estratégias para lidar com APIs não paginadas

---

### **1.4 Âmbito**

Os seguintes objetivos visam endereçar os problemas relatados na etapa de Assessment.

| Objetivo Arquitetural | Problema Endereçado | Solução Proposta |
|---|---|---|
| **Escalabilidade horizontal com orquestração** | Grandes volumes de dados e limitação de processamento em picos de carga. | Adoção de orquestração de containers com escalabilidade horizontal automática, permitindo distribuição de carga e processamento paralelo. |
| **Uso de mensageria para desacoplamento** | Alto acoplamento entre serviços e dependência direta de base de dados. | Implementação de mensageria assíncrona (pub/sub) para desacoplar produtores e consumidores, reduzindo impacto entre sistemas. |
| **Abstrações para APIs com paginação** | Falta de paginação e consumo excessivo de dados em chamadas de API. | Padronização de contratos de API com paginação, filtros e limites, garantindo performance e previsibilidade. |
| **Consumidores múltiplos de um mesmo dado** | Necessidade de atender múltiplos consumidores com o mesmo dado. | Publicação de eventos de domínio, permitindo que diferentes consumidores utilizem o mesmo dado de forma independente. |
| **Retry com backoff** | Falhas intermitentes em integrações externas. | Aplicação de políticas de retry com backoff exponencial e controle de tentativas para aumentar resiliência. |
| **Observabilidade padronizada** | Baixa visibilidade operacional e dificuldade de diagnóstico. | Implementação de logs estruturados, métricas e tracing distribuído, com correlação de requisições ponta a ponta. |

---

## **2. Dependências e Referências**

### 2.1 Dependências Técnicas

> **Nota:** Os links desta secção apontam para documentos internos de referência. Caso algum link esteja inacessível, consultar o repositório central de arquitetura ou contactar a equipa de Arquitetura.

- **Orquestradores:** Hangfire
- **Mensageria:** Solace
- **Execução:** .NET Worker, Python (para ETL)
- **Infraestrutura:** Docker/Kubernetes
- **Monitorização:** Grafana, Prometheus, OpenTelemetry

---

## **3. Regras de Utilização**

Cada tipologia terá abordados aspetos específicos da tipologia, incluindo, sempre que aplicável, referências a diretrizes, padrões gerais e boas práticas.

Este documento não é um conjunto de regras inflexíveis, mas sim um guia de fortes recomendações. Desvios são permitidos, mas devem ser justificados, documentados em ADR com a devida aprovação do Comité de Arquitetura.

---

## **4. Arquitetura da Tipologia**

### **4.1 Contexto**

A arquitetura de referência adota:

- **Scheduler** para disparo de jobs (não deve executar processamento pesado, apenas agendar/publicar — ex.: para o broker ou para um Job/Pod)
- **Mensageria** para desacoplamento
- **Workers escaláveis** para paralelismo
- **Observabilidade centralizada**

> **Nota:** Observabilidade e Resiliência são *cross-cutting concerns* transversais a todas as camadas desta arquitetura, e não constituem uma camada física isolada no fluxo de processamento.

### **4.2 Diagrama de Arquitetura Lógica**

*(Diagrama de arquitetura lógica — ver imagem original no Confluence: contentId-5235737001)*

---

### **4.3 Diagramas de Sequência Detalhados**

> [!warning]
> **Para os cenários abaixo segue-se a Semântica de Entrega:**
> - *at-least-once* (normal em mensageria) + idempotência obrigatória no worker.

> **Garantias de Entrega e Idempotência:** A semântica *at-least-once* implica que um mesmo evento pode ser processado mais de uma vez. Por isso, **idempotência é obrigatória em todos os workers**: cada unidade de trabalho deve ser identificável por um `eventId`/`correlationId` único, com registo de deduplicação, garantindo que reprocessamentos não produzam efeitos duplicados.

#### 4.3.1 Scenario 1 — Legacy 1 (pull/polling direto pelo Job Worker)

**Quando usar:**

- Tens um worker que já existe e só sabe funcionar como long-running consumer.
- Não tens Kubernetes/KEDA, ou não queres mexer em autoscaling.
- Volume constante ou suficientemente frequente para justificar worker "sempre ligado".

**Trade-offs:**

- Simples, mas custa mais (worker sempre ativo).
- Pode ter idle alto e menor eficiência.

*(Diagrama de sequência Scenario 1 — ver imagem original no Confluence: contentId-5235737001)*

---

#### 4.3.2 Scenario 2 — Ideal (KEDA escala pods; worker faz polling)

**Quando usar:**

- Kubernetes disponível.
- Queres scale-to-zero, custo otimizado e elasticidade.
- Tens métricas confiáveis (depth/lag) e queres autoscaling orientado à fila.

> [!warning]
> **Nota importante:**
> O worker normalmente continua a fazer poll ao broker. KEDA não entrega a mensagem, decide escalar com base em métricas.

*(Diagrama de sequência Scenario 2 — ver imagem original no Confluence: contentId-5235737001)*

---

#### 4.3.3 Scenario 3 — Legacy 2 (Event Gateway faz "push" para endpoint HTTP do worker)

**Quando usar:**

- O "worker" não pode consumir fila (só arranca via HTTP/webhook).
- Estás a integrar com runtime legado (ex.: app antiga, scheduler externo, algo que só reage a HTTP).
- Precisas de um componente intermediário para:
  - autenticar,
  - fazer *rate-limit*,
  - controlar retries,
  - mapear mensagens → chamadas HTTP.

**Trade-offs:**

- Mais peças = mais operação.
- Tens de desenhar bem *idempotency* e *retry semantics* no endpoint.

*(Diagrama de sequência Scenario 3 — ver imagem original no Confluence: contentId-5235737001)*

---

### **4.4 Princípios Arquiteturais**

- Processamento desacoplado por eventos
- Escalabilidade baseada em carga
- Segurança por design
- Resiliência com retries e timeouts
- Observabilidade como requisito

### **4.5 Casos de Uso**

- Processamento de eventos assíncronos em batch

### **4.6. Estratégias de Teste**

- **Unitários**: Funções e transformações
- **Integração**: APIs, filas, DB
- **Carga**: Avaliar limites de volume
- **Resiliência**: Simular timeouts/falhas
- **End-to-End**: Orquestração completa

### **4.7 Estratégias de Ingestão e Persistência**

Os processamentos em batch e job workers devem adotar estratégias de ingestão e persistência compatíveis com a origem do dado (eventos vs dados persistidos), o modo de disparo (push vs scheduling/pull) e a granularidade do processamento (event-driven vs batch).

De forma genérica, os cenários se organizam em quatro casos:

1. Processamento event-driven (consumo contínuo - pull/consumer)
2. Processamento de eventos em batch
3. Scheduling/trigger de processamento de eventos em batch
4. Scheduling/trigger de processamento de dados persistidos em outros meios

Em todos os casos, são obrigatórios: idempotência, checkpoint/controle de progresso, observabilidade, e proteção de downstream (throttling/retry/backoff/circuit breaker quando aplicável).

#### 4.7.1 Processamento Event-Driven (Streaming)

**Descrição:** cada evento recebido dispara o processamento de uma unidade de trabalho (near real-time).

**Indicado para:**

- Necessidade de baixa latência
- Eventos independentes (ou com ordenação bem definida)
- Escalabilidade por número de consumers

**Boas práticas:**

- Idempotência por evento (eventId/correlationId + registro de deduplicação)
- Sem transações longas; confirmar (ack) somente após persistência/efeito concluído
- DLQ para falhas não recuperáveis e replay controlado
- Garantir contratos de evento versionados (compatibilidade retroativa — ver Secção 9.7)

**Persistência recomendada:**

- Persistência do resultado + log/estado mínimo para reprocessamento (audit trail)

*(Diagrama de fluxo event-driven — ver imagem original no Confluence: contentId-5235737001)*

#### 4.7.2 Processamento de eventos em batch (Batch de eventos / Windowing)

**Descrição:** eventos são agregados e processados em blocos (por janela de tempo, tamanho, ou critério de negócio).

**Indicado para:**

- Alto volume com custo por evento alto
- Necessidade de consolidação/agrupamento
- Otimização de I/O (DB, storage, APIs)

**Boas práticas:**

- Definir política de "janela" (tempo/tamanho) e regra de fechamento
- Checkpoint por batch (marcar até onde foi processado)
- Idempotência por batch (batchId + hash da janela, ou watermark)
- Evitar "batchs gigantes": usar chunks e checkpoints intermediários

**Persistência recomendada:**

- Staging table/armazenamento temporário para eventos do batch (quando necessário)
- Resultado persistido com trilha de execução do batch

*(Diagrama de fluxo batch/windowing — ver imagem original no Confluence: contentId-5235737001)*

#### 4.7.3 Scheduling/trigger de processamento de eventos em batch

**Descrição:** o scheduler dispara um job que varre/seleciona eventos acumulados (fila, tópico, storage intermediário) e processa por batch.

**Indicado para:**

- Processamentos periódicos (ex.: a cada 5 min, hora, dia)
- Quando a organização quer "janelas fixas" e previsibilidade operacional
- Quando o consumo contínuo não é desejado

**Boas práticas:**

- Job deve ser reentrante (rodar de novo sem duplicar efeitos)
- Persistir estado do trigger (last_run, watermark, batchId)
- Proteger concorrência (ex.: lock distribuído ou single-run por chave)
- Garantir paginação/chunking na leitura dos eventos

**Persistência recomendada:**

- Tabela/registro de controle do scheduler (execução, watermark, status)
- Logs e métricas por execução e por batch processado

*(Diagrama de fluxo scheduling/batch — ver imagem original no Confluence: contentId-5235737001)*

#### 4.7.4 Scheduling/trigger de processamento de dados persistidos em outros meios

**Descrição:** o job não parte de eventos, mas sim de dados já persistidos (DB, Data Lake, arquivos, object storage, etc.), fazendo leitura incremental e processamento.

**Indicado para:**

- ETLs, sincronizações completas/parciais
- Integrações em que não há eventos disponíveis
- Reconciliação/correção (reprocessar base existente)

**Boas práticas:**

- Preferir incremental (watermark por data/ID, CDC, ou cursor) em vez de full scan
- Chunking obrigatório (500–2000 padrão; máx 5000 conforme Secção 9)
- Checkpoint em cada chunk (último ID/data processado)
- Se a fonte for compartilhada, aplicar throttling para não degradar o sistema
- Evitar transações longas; commit por chunk

**Persistência recomendada:**

- Tabela de controle (cursor/watermark + status + retries)
- Staging/landing zone quando houver transformação pesada

#### 4.7.5 Granularidade de ingestão: linha-a-linha vs batch/bulk

A escolha entre tratar unidade-a-unidade ou em blocos é uma decisão transversal:

**(a) Linha-a-linha** – indicada para validação individual, rastreabilidade fina, baixo/médio volume.
**(b) Batch/Bulk** – indicada para alto volume e SLA apertado, com chunks e checkpoints.

**Critérios de escolha (obrigatório documentar quando divergir do padrão):**

- Volume total e frequência
- SLA de processamento
- Capacidade do downstream
- Custo por item (CPU/IO)
- Impacto em ambientes compartilhados

### 4.8 Controle de Carga, Throttling e Escalabilidade

A arquitetura deve prever mecanismos explícitos para controle de carga, evitando sobrecarga em sistemas consumidores e garantindo estabilidade operacional.

#### 4.8.1 Throttling baseado em mensageria

Throttling baseado em mensageria é a prática de controlar a taxa de processamento a partir do consumidor, e não do produtor, usando o próprio broker como elemento regulador do fluxo. Diferente de throttling tradicional (rate limit em API), aqui o sistema aceita o pico, mas processa no ritmo seguro, desacoplando ingestão de execução. Isso é essencial em arquiteturas orientadas a eventos, integração assíncrona, pipelines de dados e workloads elásticos.

Em vez de rejeitar requisições, o sistema acumula mensagens no broker e regula:

- Quantas mensagens são entregues
- Quando são entregues
- Quando novos produtores devem desacelerar

Esse modelo preserva disponibilidade, evita cascatas de falha e mantém o sistema estável sob carga.

- Consumo regulado pelo número de consumers
- Backpressure automático quando consumidores ficam lentos
- Absorção de picos sem falha do job principal

O Job Worker não deve tentar controlar carga via lógica interna complexa. Deve privilegiar controles simples e configuráveis (concurrency/prefetch), evitando mecanismos ad-hoc difíceis de operar.

#### Mecanismo de Backpressure

Backpressure é o mecanismo que permite ao consumidor sinalizar que não consegue acompanhar o ritmo de produção. Em Mensageria, isso não é um "sinal explícito", mas o efeito combinado de vários controles técnicos.

> [!info]
> **Como funciona?**

##### Controle de prefetch / pull-based consumption

O consumidor define quantas mensagens pode receber de cada vez:

- Prefetch baixo → menor pressão
- Prefetch alto → maior throughput, maior risco

##### Acknowledgement como válvula de pressão

- Mensagens só são removidas da fila após ACK.
- Processamento lento → ACK demora
- Broker entende que o consumidor está ocupado
- Entrega desacelera automaticamente

Sem ACK, o broker não avança o offset (Kafka) ou não libera novas mensagens (queues tradicionais).

##### Consumer Lag como indicador de saturação

Quando o consumo é mais lento que a produção:

- O lag cresce
- Métricas disparam alertas
- Escala horizontal pode ser acionada

O lag não é erro, é o buffer funcionando como projetado.

##### Limites físicos e lógicos do broker

O broker impõe limites que reforçam o backpressure:

- Tamanho máximo da fila
- Retention baseada em tempo ou espaço
- Throughput por partição
- Conexões simultâneas

Quando esses limites se aproximam:

- Produtores bloqueiam
- Publicações falham temporariamente
- Retry com backoff entra em ação

##### Estratégias de throttling usando backpressure

> [!info]
> **No Consumidor:**

- Limitar concorrência interna (workers)
- Controlar prefetch dinamicamente
- Pausar consumo quando CPU/memória passam do limite

> [!info]
> **No Broker:**

- Particionar corretamente
- Configurar retenção adequada
- Isolar tópicos críticos

> [!info]
> **No Produtor:**

- Retry com exponential backoff
- Circuit breaker para publicação
- Fallback assíncrono (buffer local)

#### 4.8.2 Escalabilidade horizontal

Os workers devem ser projetados para escalar horizontalmente:

- Múltiplas instâncias do mesmo consumer
- Escala independente por tipo de processamento
- Ajuste dinâmico conforme volume e SLA

A escalabilidade deve ocorrer sem necessidade de alteração de código.

#### 4.8.3 Proteção de sistemas downstream

Para proteger APIs, bancos e outros destinos:

- Limitar paralelismo por consumer
- Aplicar retry com backoff exponencial
- Utilizar circuit breaker em dependências instáveis
- Priorizar filas separadas quando necessário

Essas medidas evitam falhas em cascata e indisponibilidade sistêmica.

#### 4.8.4. Quando usar Hangfire

Oficialmente suportado com SQL Server 2008 R2+, Redis 2.6.12+ e In-Memory. (Funciona com outras DBs como PostgreSQL e MongoDB, extra-oficialmente).

O uso de Hangfire é válido e recomendado quando:

- É necessário agendar execuções recorrentes (cron, diário, horário específico)
- É importante ter um dashboard para acompanhar jobs, fila, retries e falhas
- Queremos evitar manter threads abertas no contexto da API (fire-and-forget real, persistido)

> [!info]
> Jobs devem ser idempotentes e passíveis de retry, porque Hangfire pode repetir execuções (falhas/retries).

Observar exemplo de implementação em: [https://ecom4isi.atlassian.net/wiki/spaces/DEVP/pages/5480972371](https://ecom4isi.atlassian.net/wiki/spaces/DEVP/pages/5480972371)

---

## **5. Estrutura do Projeto**

Observar detalhe em: [https://ecom4isi.atlassian.net/wiki/spaces/DEVP/pages/5014159361](https://ecom4isi.atlassian.net/wiki/spaces/DEVP/pages/5014159361)

---

## **6. Tecnologias Utilizadas**

- **Agendamento**: Hangfire
- **Mensageria**: Solace
- **Processamento**: para Data/Machine Learning: Python. Senão, usar .NET Worker.
- **Infraestrutura**: Docker, Kubernetes CronJob

### 6.1. Uso de NoSQL com Hangfire

Hangfire suporta oficialmente SQL Server, Redis e In-Memory. Outros storages (ex.: PostgreSQL, MongoDB/NoSQL) podem ser usados via providers externos/comunidade, mas não são oficialmente suportados.

> **⚠️ Nota de Risco:** O uso de providers não-oficiais requer validação prévia com Arquitetura e testes de carga específicos. **Não é recomendado para cenários críticos de produção sem ADR aprovado.** Para cenários que exijam PostgreSQL com Hangfire, é obrigatório: (1) testes de carga validando estabilidade do provider, (2) plano de rollback para Redis/SQL Server documentado, (3) monitorização reforçada de filas e locks em produção.

---

## **7. Segurança**

Definições Globais: [https://ecom4isi.atlassian.net/wiki/spaces/DEVP/pages/5217517761](https://ecom4isi.atlassian.net/wiki/spaces/DEVP/pages/5217517761)

### 7.1 Requisitos Mínimos Obrigatórios

Independentemente das definições globais, todos os workers e jobs assíncronos devem cumprir os seguintes requisitos de segurança:

1. **Autenticação:** todos os workers devem autenticar-se nos sistemas que consomem ou produzem dados (broker, APIs externas, bases de dados) usando identidades de serviço dedicadas, nunca credenciais partilhadas ou hardcoded.
2. **Encriptação em trânsito (TLS):** todas as conexões externas (APIs, brokers, bases de dados remotas) devem utilizar TLS; conexões sem encriptação não são permitidas em produção.
3. **Autorização por princípio do menor privilégio:** cada worker deve ter apenas as permissões estritamente necessárias para a sua função (ex.: leitura num tópico específico, escrita numa tabela específica).
4. **Gestão de segredos via Vault:** credenciais, tokens e chaves de API devem ser obtidos do Vault em runtime, nunca em ficheiros de configuração ou variáveis de ambiente em texto claro. Workers devem implementar caching local de segredos com TTL configurável e fallback para credenciais de emergência rotativas em caso de indisponibilidade prolongada do Vault.
5. **Auditoria:** operações sensíveis (acesso a dados pessoais, alterações de estado crítico) devem gerar eventos de auditoria com `jobId`, `runId`, `operatorId` e timestamp.

---

## **8. Infraestrutura**

### **8.2. CI/CD e DevOps**

- Pipelines de build e teste automatizados
- Deploy versionado com rollback
- Jobs/CronJobs no Kubernetes via Helm/ArgoCD
- Promotion de ambientes com aprovação

---

## **9. Padrões e Princípios Arquiteturais**

Os valores abaixo são *defaults corporativos de partida* e devem ser calibrados conforme SLO/SLA, testes de carga e capacidade dos sistemas downstream. As referências R1…Rn suportam os princípios e mecanismos (retry/backoff/timeout/paginação/logging/observabilidade), e podem não refletir valores numéricos idênticos em todos os cenários.

> **⚠️ Nota:** Os valores numéricos definidos nesta secção (retry, timeout, chunking) são **baseline corporativo de partida**. Ajustes devem ser validados com testes de carga e registados em ADR quando divergirem do padrão, com aprovação da Arquitetura.

**Retry máximo:**

- Operações em API externas: **3 tentativas**
- Persistência em DB: **2 tentativas**

**Backoff exponencial:** iniciar em **5s**, dobrando até no máximo **60s**
*(Refs: R1)*

**Timeouts:**

- Chamada de API: **30 segundos**
- Persistência: **60 segundos**
*(Refs: R1)*

**Paginação e Chunking:**

- Padrão: **500 a 2.000** registros por requisição
- Máximo: **5.000** registros por requisição
*(Refs: R3)*

**Limite de execução de job:**

- Jobs menores: até **5 min**
- Jobs de alto volume: até **2 horas** (com checkpoints)
*(Refs: R4)*

**Logs:**

- Nível INFO para início/fim de execução
- Nível ERROR para falhas com stack trace
*(Refs: R5)*

**Idempotência:** obrigatória em todos os jobs
*(Refs: R2, R4)*

**Configuração externa:** sem valores fixos no código (12-Factor App)
*(Refs: R4)*

## 9.1. Referências dos Princípios (R1–R7)

**R1 — Resiliência (.NET): retry, backoff, timeout, circuit breaker**

- Polly (site oficial): [https://www.pollydocs.org/](https://www.pollydocs.org/)
- Microsoft Learn – retries com backoff usando Polly: [https://learn.microsoft.com/en-us/dotnet/architecture/microservices/implement-resilient-applications/implement-http-call-retries-exponential-backoff-polly](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/implement-resilient-applications/implement-http-call-retries-exponential-backoff-polly)

**R2 — Hangfire: retries e comportamento em falhas**

- Hangfire – Dealing with Exceptions (inclui AutomaticRetry): [https://docs.hangfire.io/en/latest/background-processing/dealing-with-exceptions.html](https://docs.hangfire.io/en/latest/background-processing/dealing-with-exceptions.html)
- Hangfire API – AutomaticRetryAttribute Class: [https://api.hangfire.io/html/T_Hangfire_AutomaticRetryAttribute.htm](https://api.hangfire.io/html/T_Hangfire_AutomaticRetryAttribute.htm)

**R3 — Paginação / chunking (guidelines de API)**

- Microsoft Azure REST API Guidelines (nextLink absoluto): [https://github.com/microsoft/api-guidelines/blob/vNext/azure/Guidelines.md](https://github.com/microsoft/api-guidelines/blob/vNext/azure/Guidelines.md)
- Zalando RESTful API Guidelines (cursor-based pagination): [https://opensource.zalando.com/restful-api-guidelines/](https://opensource.zalando.com/restful-api-guidelines/)

**R4 — Agendamento e execução de jobs (Kubernetes CronJob/Job)**

- Kubernetes – CronJobs (conceitos): [https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/)
- Kubernetes – Running Automated Tasks with a CronJob (how-to): [https://kubernetes.io/docs/tasks/job/automated-tasks-with-cron-jobs/](https://kubernetes.io/docs/tasks/job/automated-tasks-with-cron-jobs/)

**R5 — Observabilidade e correlação (OpenTelemetry)**

- OpenTelemetry – Context propagation: [https://opentelemetry.io/docs/concepts/context-propagation/](https://opentelemetry.io/docs/concepts/context-propagation/)
- OpenTelemetry – Logs specification: [https://opentelemetry.io/docs/specs/otel/logs/](https://opentelemetry.io/docs/specs/otel/logs/)

**R6 — Alertas (Prometheus)**

- Prometheus – Alerting rules: [https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/](https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/)
- Prometheus – Alerting overview (Alertmanager, fluxo): [https://prometheus.io/docs/alerting/latest/overview/](https://prometheus.io/docs/alerting/latest/overview/)

**R7 — Alertas e gestão (Grafana Alerting)**

- Grafana – Alerting overview: [https://grafana.com/docs/grafana/latest/alerting/](https://grafana.com/docs/grafana/latest/alerting/)
- Grafana – Alert rules (fundamentals): [https://grafana.com/docs/grafana/latest/alerting/fundamentals/alert-rules/](https://grafana.com/docs/grafana/latest/alerting/fundamentals/alert-rules/)

---

## **10. Observabilidade**

Observar o padrão em: [https://ecom4isi.atlassian.net/wiki/spaces/DEVP/pages/5217517761](https://ecom4isi.atlassian.net/wiki/spaces/DEVP/pages/5217517761)

### 10.1 Requisitos Mínimos Obrigatórios

Todos os adapters/workers devem implementar obrigatoriamente:

1. **Métricas base:** `jobs_started_total`, `jobs_succeeded_total`, `jobs_failed_total`, `job_duration_seconds` (p50/p95/p99).
2. **Correlação end-to-end:** `correlationId` / `traceId` propagado desde o Scheduler → Broker → Worker → API/BD. Incluir sempre no log estruturado: `jobId`, `runId`, `messageId`, `correlationId`, `chunkId/page`, `attempt`.
3. **Alertas mínimos:** job falhado N vezes consecutivas, lag/fila acima de limiar por Y minutos, e duração do job acima do p95 histórico.

### 10.2 Métricas, Correlação e Alertas

1. **Correlation/Tracing end-to-end**
   - `correlationId` / `traceId` propagado desde o Scheduler → Broker → Worker → API/BD.
   - Incluir no log estruturado sempre: `jobId`, `runId` (execução), `messageId`, `correlationId`, `chunkId/page`, `attempt`.

2. **Métricas mínimas por job (com SLI)**
   - `jobs_started_total`, `jobs_succeeded_total`, `jobs_failed_total`
   - `job_duration_seconds` (p50/p95/p99)
   - `messages_consumed_total` e `consume_lag` (ou "queue depth"/"age of oldest message")
   - `retries_total`, `dlq_total` (se existir DLQ)
   - `external_api_latency_seconds` + `external_api_errors_total` (por endpoint)
   - `db_write_latency_seconds` + `db_write_errors_total`

3. **Alertas recomendados (templates)**
   - Job falhado N vezes ou sem sucesso em X tempo (ex.: "última execução OK > 24h")
   - Lag/idade da fila acima de limiar por Y minutos
   - Erro 5xx ou timeouts na API externa acima de limiar
   - Duração do job acima do normal (p95 > baseline)
   - DLQ > 0 ou a crescer continuamente

4. **Dashboards padrão**
   - Por job: sucesso/falha, duração, throughput, retries, lag
   - Por dependência: API externa (latência/erros), BD (latência/erros)
   - Por infra: CPU/memória dos workers, restarts, throttling/rate limit

### 10.3 Monitorização de SLA por Integração

Cada integração crítica deve ter o seu SLA individualmente definido, medido e alertado:

- **Definição:** acordar com o produto/negócio o SLA por tipo de job (ex.: "job de reconciliação financeira deve ter 99,9% de execuções com sucesso e duração < 30 min").
- **Medição:** usar as métricas de `jobs_succeeded_total` e `job_duration_seconds` para calcular a taxa de sucesso e percentis de duração por período (diário/semanal).
- **Alertas de SLA:** configurar alertas específicos por integração quando a taxa de sucesso ou duração divergir do SLA acordado, não apenas alertas genéricos de falha.
- **Revisão periódica:** SLAs devem ser revistos trimestralmente e ajustados conforme evolução do sistema e expectativas do negócio.

---

> **Aplicação de Exemplo:** [sample-swrefarch-batch-job-worker](https://github.com/mcdigital-devplatforms/sample-swrefarch-batch-job-worker)
