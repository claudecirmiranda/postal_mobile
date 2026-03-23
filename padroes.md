Índice
------

- [1. Infrastructure Platform](#1-infrastructure-platform)
    - [1.1. Container/Kubernetes based PGSQL Database](#11-containerkubernetes-based-pgsql-database)
    - [1.2. API Gateway Platform](#12-api-gateway-platform)
    - [1.3. Streaming & Messaging Platform](#13-streaming-messaging-platform)
    - [1.4. Secrets Management](#14-secrets-management)
- [2. Observability Platform](#2-observability-platform)
    - [2.1. Objectivos](#21-objectivos)
    - [2.2. Os três pilares](#22-os-três-pilares)
    - [2.3. Correlação distribuída](#23-correlação-distribuída)
    - [2.4. O que medir](#24-o-que-medir)
    - [2.5. Regras de Logging](#25-regras-de-logging)
    - [2.6. Dashboards e Alertas](#26-dashboards-e-alertas)
    - [2.7. Anti-padrões](#27-anti-padrões)
    - [2.8. Stack de Observabilidade](#28-stack-de-observabilidade)
    - [2.9. Estratégia de Sampling](#29-estratégia-de-sampling)
    - [2.10. Documentação Logging:](#210-documentação-logging)
- [3. Health Checks](#3-health-checks)
- [4. Security - Identity, Authentication & Authorization](#4-security-identity-authentication-authorization)
    - [4.1. Requisitos Mínimos de Segurança](#41-requisitos-mínimos-de-segurança)
    - [4.2. Documentação Security by Design (SbD)](#42-documentação-security-by-design-sbd)
- [5. Notification Pattern](#5-notification-pattern)
    - [**5.1 Padrão base: Event-driven + Notification Service**](#51-padrão-base-event-driven-notification-service)
        - [5.1.1 Componentes essenciais](#511-componentes-essenciais)
        - [5.1.2 Garantias de entrega: at-least-once com idempotência](#512-garantias-de-entrega-at-least-once-com-idempotência)
        - [5.1.3 Preferências do utilizador e governança](#513-preferências-do-utilizador-e-governança)
        - [5.1.4 In-app: persistir + entregar em tempo real](#514-in-app-persistir-entregar-em-tempo-real)
        - [5.1.5 Push: gestão de tokens e falhas](#515-push-gestão-de-tokens-e-falhas)
        - [5.1.6 Email: templates, rastreio e compliance](#516-email-templates-rastreio-e-compliance)
        - [5.1.7 Observabilidade mínima](#517-observabilidade-mínima)
- [6. Fiabilidade e Operação](#6-fiabilidade-e-operação)
    - [6.1. Idempotency](#61-idempotency)
    - [**6.2 Onde Aplicar Idempotência na Arquitetura**](#62-onde-aplicar-idempotência-na-arquitetura)
    - [6.3. Scalability](#63-scalability)
        - [**6.3.1 Arquitetura de Escala com KEDA**](#631-arquitetura-de-escala-com-keda)
        - [**6.3.2 Estratégias Avançadas**](#632-estratégias-avançadas)
        - [**6.3.3 Boas Práticas**](#633-boas-práticas)
        - [**6.3.4 Resultado Esperado**](#634-resultado-esperado)
        - [**6.3.5 Casos de Uso**](#635-casos-de-uso)
    - [6.4. Graceful Shutdown](#64-graceful-shutdown)
        - [**6.4.1. O que é Graceful Shutdown**](#641-o-que-é-graceful-shutdown)
        - [**6.4.2. Por que Graceful Shutdown é crítico**](#642-por-que-graceful-shutdown-é-crítico)
        - [**6.4.3. Graceful Shutdown no contexto de APIs HTTP**](#643-graceful-shutdown-no-contexto-de-apis-http)
        - [**6.4.4. Graceful Shutdown em consumidores de Broker**](#644-graceful-shutdown-em-consumidores-de-broker)
        - [**6.4.5 Regra fundamental**](#645-regra-fundamental)
        - [**6.4.6. Interação com Kubernetes**](#646-interação-com-kubernetes)
        - [**6.4.7. Relação com Escalabilidade e Autoscaling**](#647-relação-com-escalabilidade-e-autoscaling)
        - [**6.4.8. Boas práticas essenciais**](#648-boas-práticas-essenciais)
        - [**6.4.9. Anti-patterns comuns**](#649-anti-patterns-comuns)
        - [**6.4.10. Graceful Shutdown e Idempotência**](#6410-graceful-shutdown-e-idempotência)
- [7. Gestão de Configuração](#7-gestão-de-configuração)
    - [7.1. Hierarquia de Configuração](#71-hierarquia-de-configuração)
    - [7.2. Promoção entre Ambientes](#72-promoção-entre-ambientes)
    - [7.3. Feature Flags](#73-feature-flags)
- [8. CI/CD](#8-cicd)
    - [8.1. Pipeline de Build](#81-pipeline-de-build)
    - [8.2. Deploy](#82-deploy)
    - [8.3. Execução de Testes de Integração com Testcontainers na Pipeline (CI/CD)](#83-execução-de-testes-de-integração-com-testcontainers-na-pipeline-cicd)
    - [8.4. Gestão de Vulnerabilidades em Imagens Docker](#84-gestão-de-vulnerabilidades-em-imagens-docker)
        - [8.4.1 Princípios](#841-princípios)
        - [8.4.2 Responsabilidades](#842-responsabilidades)
        - [8.4.3 Pipeline de Segurança](#843-pipeline-de-segurança)
        - [8.4.4 Atualização e Remediação](#844-atualização-e-remediação)
- [9. Preocupações Transversais Adicionais](#9-preocupações-transversais-adicionais)
    - [9.1. Rate Limiting e Throttling](#91-rate-limiting-e-throttling)
    - [9.2. Estratégia de Cache](#92-estratégia-de-cache)
    - [9.3. Multi-tenancy](#93-multi-tenancy)
    - [9.4. Privacidade de Dados (GDPR/LGPD)](#94-privacidade-de-dados-gdprlgpd)
    - [9.5. Disaster Recovery para Componentes Stateful](#95-disaster-recovery-para-componentes-stateful)

# 1. Infrastructure Platform

## 1.1. Container/Kubernetes based PGSQL Database

Documentação interna sobre como implementar PGSQL em container ou Kubernetes.

[Documentação PGSQL em Kubernetes (Confluence)](https://ecom4isi.atlassian.net/wiki/spaces/INFRA/pages/4733960854)


## 1.2. API Gateway Platform

A plataforma corporativa de API Gateway é o [Gravitee](https://www.gravitee.io/).

[Documentação do Gravitee (Confluence)](https://ecom4isi.atlassian.net/wiki/x/zwBS9)

## 1.3. Streaming & Messaging Platform

Nossa plataforma de Streaming & Messaging é o [Solace](https://solace.com/).

- Pacote: `SolaceSystems.SolClient.Messaging`
- Uso principal: integração com event mesh corporativo e sistemas legados.

[Documentação Solace – Integração Corporativa (Confluence)](https://ecom4isi.atlassian.net/wiki/x/qIC6EQE)

[Documentação Solace – Guia de Configuração (Confluence)](https://ecom4isi.atlassian.net/wiki/x/EYKsGgE)

## 1.4. Secrets Management

Nossa plataforma preferencial de Secrets Management é o [Hashicorp Vault](https://www.hashicorp.com/en/products/vault). Azure Key Vault encontra-se em phase-out. GitHub Secrets podem ser usados apenas em contexto de CI/CD.

[Documentação Hashicorp Vault – Gestão de Segredos (Confluence)](https://ecom4isi.atlassian.net/wiki/x/qgBP9)


---

# 2. Observability Platform

Observabilidade é um requisito obrigatório para todos os serviços e componentes da plataforma. O objetivo é garantir visibilidade operacional consistente, diagnóstico eficiente e suporte a decisões baseadas em dados.

## 2.1. Objectivos

- Reduzir o tempo de diagnóstico (MTTR)
- Correlacionar logs, métricas e traces
- Permitir monitorização de SLO/SLI
- Evitar silos de monitorização por tecnologia ou equipa


---

## 2.2. Os três pilares

**Logs**

- Registos estruturados que descrevem eventos relevantes da aplicação.

**Métricas**

- Medições agregadas que permitem monitorizar comportamento ao longo do tempo.

**Traces**

- Representação distribuída de uma operação ponta-a-ponta entre serviços.


## 2.3. Correlação distribuída

Todos os sistemas devem suportar correlação ponta-a-ponta.

Campos obrigatórios:

- `traceId`
- `spanId`
- `correlationId`
- `service.name`
- `environment`

Broker deve incluir também:

- `causationId`
- `eventType`
- `eventVersion`
- `timestamp`



## 2.4. O que medir

**HTTP**

- Latência (p95 mínimo)
- Taxa de erro
- Throughput

**Broker**

- Tempo de processamento
- Retries
- DLQ
- Backlog/lag

**Dependências**

- Latência
- Taxa de falha
- Timeouts

## 2.5. Regras de Logging

- Logs devem ser estruturados
- Níveis de log devem seguir padrão corporativo
- Dados sensíveis não podem estar contidos em log
- Retenção e sampling seguem política de plataforma

## 2.6. Dashboards e Alertas

Cada serviço deve ter:

- Dashboard padrão com latência, erros e throughput
- Monitorização de dependências
- Alertas baseados em sintomas (ex.: aumento de 5xx + latência)

## 2.7. Anti-padrões

- Logs com PII ou segredos
- Traces gigantes sem valor operacional
- Logging excessivo que aumente custos e ruído
- Métricas sem dono ou sem utilização prática

## 2.8. Stack de Observabilidade

A stack de observabilidade corporativa baseia-se em **OpenTelemetry, Prometheus e Grafana**.

- **Logs:** Seguir o padrão OTEL: Microsoft Logging Extensions → (provider) OpenTelemetry logging → OTLP exporter → collector/backend. Os campos de correlação são automaticamente propagados quando existe Activity ativa e o provider OpenTelemetry está corretamente configurado. Campos adicionais podem ser enriquecidos via scopes ou processors.

- **Métricas:** Métricas chave como lag do consumidor de Broker, latência de processamento e taxa de erros serão coletadas.

  ```
  Métricas técnicas e de negócio:
  - orders_created_total
  - orders_kafka_published_total
  - orders_solace_consumed_total
  ```

- **Traces:** Observar guidelines em [Documentação de Traces OpenTelemetry (Confluence)](https://ecom4isi.atlassian.net/wiki/spaces/DEVP/pages/5425692703)

  - O SDK do OpenTelemetry ([opentelemetry-dotnet-instrumentation](https://github.com/open-telemetry/opentelemetry-dotnet-instrumentation)) deve ser utilizado para instrumentação distribuída. Sempre que possível será usada auto-instrumentação (agente), podendo ser complementada por instrumentação via SDK nos serviços quando necessário, permitindo rastrear uma requisição desde a sua origem até a sua persistência final. A propagação de contexto deve seguir o padrão W3C Trace Context para HTTP e mecanismos equivalentes para Broker. Os traces serão enviados para o Grafana Tempo.

- **Dashboards:** O Grafana será a ferramenta central para visualização de métricas, exploração de logs e análise de traces, com dashboards dedicados para a saúde da plataforma de ingestão.

## 2.9. Estratégia de Sampling

Em cenários de alto volume, a recolha integral de traces pode saturar o backend de observabilidade (Tempo/Grafana) e aumentar custos de forma não linear. A estratégia de sampling deve ser configurável por ambiente:

- **Erros e exceções:** head-based sampling a 100% — todos os traces com erro devem ser recolhidos.
- **Tráfego normal:** tail-based ou probabilístico, com taxa configurável por ambiente (ex.: 10–20% em produção, 100% em staging/dev).
- **Operações críticas de negócio** (ex.: pagamentos, checkout): sampling elevado independentemente do resultado.

A configuração de sampling deve ser externalizada via variável de ambiente ou ficheiro de configuração, sem necessidade de redeploy.

## 2.10. Documentação Logging:

[Documentação de Logging Corporativo (Confluence)](https://ecom4isi.atlassian.net/wiki/spaces/DEVP/pages/5039063075)

[Microsoft Logging Extensions – Documentação Oficial](https://learn.microsoft.com/en-us/dotnet/core/extensions/logging?tabs=command-line) / [NuGet Microsoft.Extensions.Logging](https://www.nuget.org/packages/Microsoft.Extensions.Logging)

[OpenTelemetry .NET Instrumentation (GitHub)](https://github.com/open-telemetry/opentelemetry-dotnet-instrumentation)

# 3. Health Checks

Health checks são mecanismos automáticos usados para verificar se uma aplicação está viva, está pronta para receber tráfego e continua saudável ao longo do tempo. Eles são fundamentais em arquiteturas modernas, pois permitem que plataformas de orquestração, balanceadores e sistemas de monitorização tomem decisões automáticas — como reiniciar instâncias, remover tráfego ou escalar serviços.

**Tipos de Health Check**

1. **Liveness Check**

Verifica se o processo da aplicação ainda está em execução e não entrou em estado irrecuperável (deadlock, loop infinito, crash silencioso).

Se falhar, a instância pode ser reiniciada automaticamente.

1. **Readiness Check**

Indica se a aplicação está pronta para atender requests agora.

Pode falhar mesmo que a aplicação esteja “viva”.

Se falhar, o tráfego é retirado temporariamente, sem reiniciar a app.

1. **Startup Check**

Usado quando a aplicação demora a inicializar (cold start, bootstrap pesado).

Evita que o liveness check mate a app antes do startup terminar.

**Boas práticas**

- Ser rápido (milissegundos)
- Não depender de chamadas externas lentas
- Retornar status simples (200 / 503)
- Ter lógica clara e determinística
- Separar *liveness* de *readiness*

**Evite**

- Consultas pesadas à BD
- Chamadas a APIs externas críticas
- Lógica de negócio complexa
- Efeitos colaterais (writes, locks)

**Operacionalização**

- Endpoints `/health` (liveness) e `/health/ready` (readiness).
- Integração com probes do Kubernetes.

Health checks não substituem métricas, logs e traces, mas trabalham em conjunto:

- **Health check** → decisão binária (ok / não ok)
- **Métricas** → tendências e degradação
- **Logs** → causa raiz
- **Traces** → latência e dependências


---

# 4. Security - Identity, Authentication & Authorization

Identity Managers centralizam e padronizam quem é o utilizador (autenticação) e o que ele pode fazer (autorização) em sistemas distribuídos. Eles atuam como uma camada dedicada de identidade, reduzindo acoplamento entre aplicações e evitando que cada serviço implemente sua própria lógica de segurança.

Na autenticação, o Identity Manager valida credenciais (password, MFA, certificados, social login) e emite tokens (ex.: JWT, opaque tokens) que representam a identidade do utilizador ou serviço. Esses tokens são assinados, possuem expiração e podem carregar claims como userId, tenant e roles.

Na autorização, os serviços consomem esses tokens para decidir acesso com base em roles, scopes ou políticas. A decisão pode ser local (validação de claims) ou centralizada (policy engine), garantindo consistência de permissões em todo o ecossistema.

Essa abordagem é essencial para arquiteturas modernas (microserviços, BFFs, APIs públicas), pois suporta SSO, multi-tenant, zero trust, integração com parceiros e evolução de regras de acesso sem impacto direto no código das aplicações.

## 4.1. Requisitos Mínimos de Segurança

Independentemente dos detalhes nos documentos especializados, todos os serviços devem cumprir obrigatoriamente os seguintes requisitos:

1. **Validação de tokens:** Todos os serviços devem validar tokens JWT assinados pelo IdP corporativo. Tokens expirados ou com assinatura inválida devem resultar em rejeição com HTTP 401.
2. **Gestão de segredos:** Segredos, credenciais e chaves de API devem ser geridos exclusivamente via Hashicorp Vault. É proibido armazenar segredos em variáveis de ambiente não geridas, ficheiros de configuração ou repositórios de código.
3. **Encriptação em trânsito:** Toda a comunicação entre serviços deve usar TLS 1.2 ou superior. Comunicação em texto simples não é permitida em nenhum ambiente (incluindo desenvolvimento).
4. **Princípio do menor privilégio:** Cada serviço deve operar com o conjunto mínimo de permissões necessário. Scopes e roles devem ser revistos periodicamente.
5. **Sem dados sensíveis em logs:** PII, segredos, tokens ou dados de pagamento nunca devem ser registados. Ver secção de regras de logging (2.5).

Os detalhes completos de implementação encontram-se na documentação Security by Design referenciada abaixo.

## 4.2. Documentação Security by Design (SbD)

[Princípios de Security by Design – Visão Geral (Confluence)](https://ecom4isi.atlassian.net/wiki/spaces/SBDR/pages/4099309677)

[Autenticação e Gestão de Identidade (Confluence)](https://ecom4isi.atlassian.net/wiki/spaces/SBDR/pages/4099145846)

[Autorização e Controlo de Acesso (Confluence)](https://ecom4isi.atlassian.net/wiki/spaces/SBDR/pages/4092919859)

[Gestão de Segredos e Credenciais (Confluence)](https://ecom4isi.atlassian.net/wiki/spaces/SBDR/pages/4098621650)

[Encriptação e Comunicação Segura (Confluence)](https://ecom4isi.atlassian.net/wiki/spaces/SBDR/pages/4099342508)

[Zero Trust e Segurança em Microserviços (Confluence)](https://ecom4isi.atlassian.net/wiki/spaces/SBDR/pages/4092690474)


---

# 5. Notification Pattern

Envio de notificações por email, push e in-app para comunicação com o utilizador.

Implementar notificações (e-mail, push, in-app) em arquiteturas distribuídas funciona melhor quando se trata notificação como um produto/serviço separado, acionado por eventos e entregue por canais plugáveis.

## **5.1 Padrão base: Event-driven + Notification Service**

1. Um serviço de domínio (Orders, Payments, Shipping) publica um evento de negócio: OrderPaid, DeliveryDelayed, PasswordChanged.
2. Um Notification Orchestrator (ou “Notification Service”) consome esses eventos.
3. Ele aplica regras (preferências do utilizador, severidade, horário quiet hours, opt-in/opt-out, tenancy).
4. Ele cria comandos de entrega por canal: SendEmail, SendPush, CreateInApp.
5. “Providers” por canal fazem o envio (SMTP/SES/SendGrid; FCM/APNs; In-app via DB + websocket/polling).

Benefícios: desacoplamento, escala independente, resiliente a falhas de canais.

### 5.1.1 Componentes essenciais

- Event Bus / Broker
- Pub/Sub (Kafka, RabbitMQ, SNS/SQS, Azure Service Bus).
- Eventos imutáveis e versionados.
- Consumer groups para escalar consumidores.
- Notification Orchestrator
- Responsável por:
- Deduplicação (idempotência com eventId/messageId)
- Enriquecimento (buscar e-mail, device tokens, idioma, tenant)
- Routing e policies (quem recebe, por qual canal, quando)
- Fan-out (um evento → múltiplos destinatários/canais)
- **Políticas de fallback entre canais:** se um canal primário falhar (ex.: push notification após 2 tentativas), o Orchestrator deve tentar canais alternativos consoante a severidade do evento. Exemplo: para eventos de severidade alta (pagamento, segurança), fallback automático para email se o push falhar. As políticas devem ser configuráveis por tipo de evento.
- Canal como “adapters”
- EmailAdapter, PushAdapter, InAppAdapter.
- Cada adapter com:
- Retentativas + backoff
- Circuit breaker
- Métricas e DLQ (dead-letter queue)

### 5.1.2 Garantias de entrega: at-least-once com idempotência

- Outbox pattern no serviço produtor (grava evento no mesmo commit do dado de negócio).
- Consumidores idempotentes:
- Tabela/redis de “processed message ids”
- Chave idempotente por (eventId, userId, channel).

### 5.1.3 Preferências do utilizador e governança

Modele como um serviço/feature própria:

- Opt-in/opt-out por canal e tipo (marketing, transactional, security)
- Horário silencioso (“quiet hours”)
- Limites (rate limits) e anti-spam
- Templates por idioma e tenant

Dica prática: manter uma “Notification Policy Store” (DB) e cache leve.

### 5.1.4 In-app: persistir + entregar em tempo real

In-app geralmente é:

- Persistência em notifications (por user/tenant, status read/unread)
- API para listar/paginar e marcar como lida
- “Real-time”: websocket / SSE / pubsub → gateway

Assim você garante que a notificação aparece mesmo se o user estiver offline.

### 5.1.5 Push: gestão de tokens e falhas

- Token registry por user/device/tenant
- Rotação/expiração de tokens (limpar tokens inválidos)
- Fallback: se push falhar, manter in-app (e opcionalmente e-mail, dependendo do tipo)

### 5.1.6 Email: templates, rastreio e compliance

- Templates versionados (HTML/text)
- Conteúdo transacional separado de marketing
- Tracking (delivery, bounce) via webhooks do provider
- LGPD/GDPR: consentimento, retenção mínima, auditoria

### 5.1.7 Observabilidade mínima

- Métricas: sent_total, failed_total, retry_total, latency_ms por canal
- Logs com correlationId, eventId, userId, tenantId
- Traces do evento até o provider
- Alarmes: aumento de DLQ, bounce rate, falhas de provider, backlog alto

# 6. Fiabilidade e Operação

## 6.1. Idempotency

Capacidade de uma operação ser executada múltiplas vezes, retomando a partir do ponto onde parou, sem repetir itens previamente processados e garantindo sempre o mesmo resultado final.

Duplicações podem ocorrer por:

- Retries no producer (falhas transitórias).
- Reentrega pelo broker (at-least-once delivery).
- Reprocessamento manual via DLQ.
- Escalonamento horizontal de consumidores.
- Falhas após commit parcial (ex.: DB commit feito, mas ack não enviado).

## **6.2 Onde Aplicar Idempotência na Arquitetura**

**Producers (API / Application)**

- Todo evento publicado deve conter um identificador único:
- eventId
- correlationId
- causationId
- Esse identificador deve ser imutável e persistente.

**Consumers (Regra Crítica)**

Todo consumidor deve verificar se o evento já foi processado antes de executar lógica de negócio.

Padrão recomendado:

**Inbox Pattern (Event Deduplication)**

- Tabela dedicada:

``` 
processed_events
----------------
event_id (PK)
processed_at
handler_name
```

- Fluxo:

1. Recebe mensagem
2. Verifica se event_id já existe
3. Se existir → ACK e retorna
4. Se não existir → processa e grava

Exemplo de Implementação (Consumer):

[https://ecom4isi.atlassian.net/wiki/spaces/DEVP/pages/edit-v2/5378539587#Exemplo-de-Implementa%C3%A7%C3%A3o-(Consumer)](https://ecom4isi.atlassian.net/wiki/spaces/DEVP/pages/edit-v2/5378539587#Exemplo-de-Implementa%C3%A7%C3%A3o-(Consumer))

**Observações importantes**

- A verificação e o processamento devem ser atômicos.
- Preferencialmente dentro da mesma transação BD.
- O ACK só ocorre após sucesso completo.

**Idempotência em Commands HTTP**

Para APIs expostas (ex.: POST /orders):

- Aceitar Idempotency-Key no header.
- Persistir a chave com o resultado da operação.
- Retornar o mesmo resultado em chamadas repetidas.

``` 
POST /api/orders
Idempotency-Key: 8f6d1e...
```

## 6.3. Scalability

Garantir que a aplicação ou serviço pode escalar horizontalmente, permitindo que vários pods partilhem o trabalho de forma segura, e que o escalamento é feito com base em métricas ou eventos (ex.: KEDA).

O Escalonamento Horizontal baseado em CPU/memória é insuficiente, porque:

- Não entende backlog
- Reage tarde
- Escala errado em consumers I/O bound
- Não protege contra overload de broker

KEDA ajuda a mitigar este problema com scaling baseado em métricas externas.

### **6.3.1 Arquitetura de Escala com KEDA**

``` 
Solace / Broker
   ↓
Métrica (queue depth / lag)
   ↓
Prometheus / Adapter
   ↓
KEDA ScaledObject
   ↓
Deployment Consumer
```

**Métricas de Broker**

Exemplos:

- solace_queue_depth
- messages_ready
- messages_unacked
- consumer_lag

**Métricas de Aplicação**

Expor via OpenTelemetry:

- message_processing_duration_seconds
- message_processing_errors_total
- dlq_messages_total

Essas métricas ajudam a ajustar thresholds, detectar saturação lógica e evitar scale infinito.

Exemplo de ScaledObject (Consumer):

[https://ecom4isi.atlassian.net/wiki/spaces/DEVP/pages/edit-v2/5378539587#ScaledObject-(Consumer)%3A](https://ecom4isi.atlassian.net/wiki/spaces/DEVP/pages/edit-v2/5378539587#ScaledObject-(Consumer)%3A)

### **6.3.2 Estratégias Avançadas**

**Backpressure Controlado**

- Limitar concorrência por pod
- Preferir mais pods com menos threads
- Evitar saturar dependências downstream

**Escala + Circuit Breaker**

Combinar:

- KEDA (escala)
- Circuit breaker (proteção)

Exemplo:

- Se DB está lento → circuit abre
- Mensagens acumulam → KEDA escala
- Após recuperação → consumo normaliza

### **6.3.3 Boas Práticas**

- Sempre definir maxReplicaCount
- Nunca escalar infinitamente
- Métrica ≠ CPU
- Escala deve respeitar limites de throughput e concorrência definidos pelo broker para evitar rejeição de mensagens ou degradação
- Testar scale em ambientes não-prod

### **6.3.4 Resultado Esperado**

- Consumo elástico
- Redução de latência sob pico
- Custos controlados
- Sistema previsível sob carga

### **6.3.5 Casos de Uso**

1. **Aplicações que lêem o mesmo arquivo texto**

File → Queue → Consumers (modelo correto para escala)

Transforme o arquivo em fonte, não em meio de coordenação.

``` 
Arquivo
   ↓
Producer (1x)
   ↓
Fila / Topic
   ↓
App A   App B   App C
```

**Por quê?**

- Cada linha vira uma mensagem
- Broker garante distribuição
- Escala horizontal nativa
- Retry, DLQ, backpressure

**Exemplo**

- Arquivo CSV → Kafka / Rabbit / Solace
- Cada linha = evento

## 6.4. Graceful Shutdown

Garantir que, ao receber um sinal de encerramento, a aplicação deixa de aceitar novos pedidos e tenta finalizar os que estão em curso; caso não seja possível, deve propagar um cancellation token para interromper corretamente a operação.

### **6.4.1. O que é Graceful Shutdown**

Graceful shutdown é a capacidade de uma aplicação encerrar sua execução de forma controlada, garantindo que:

- Nenhuma nova requisição/mensagem seja aceita
- Processamentos em andamento sejam concluídos (ou interrompidos com segurança)
- Recursos sejam liberados corretamente
- O sistema não gere efeitos colaterais indesejados (dados inconsistentes, mensagens duplicadas, corrupção de estado)

Em arquiteturas modernas (containers, Kubernetes, Broker), o shutdown não é um evento raro, mas parte do fluxo normal de operação (deploys, autoscaling, falhas, rebalanceamentos).

### **6.4.2. Por que Graceful Shutdown é crítico**

Sem graceful shutdown, a aplicação pode:

- Perder mensagens em processamento
- Processar mensagens parcialmente
- Quebrar idempotência
- Gerar duplicações
- Deixar locks, transações ou conexões abertas
- Corromper dados

Em sistemas event-driven, o risco é ainda maior, pois:

- Brokers assumem que o consumidor falhou
- Mensagens são reenviadas
- Reprocessamento ocorre automaticamente

### **6.4.3. Graceful Shutdown no contexto de APIs HTTP**

Fluxo correto ao receber um sinal de shutdown (SIGTERM):

1. Parar de aceitar novas requisições
2. Permitir que requisições em andamento terminem
3. Aplicar timeout máximo de espera
4. Encerrar a aplicação

**Exemplo conceitual**

- A aplicação deve sinalizar readiness=false antes de iniciar o shutdown, permitindo ao Kubernetes retirar o pod do tráfego.
- Load balancer remove o pod do tráfego
- Requests existentes finalizam
- Após o timeout, o processo encerra

Sem isso:

- Requests são interrompidas abruptamente
- Clientes recebem erros
- Estados intermediários ficam inconsistentes

### **6.4.4. Graceful Shutdown em consumidores de Broker**

Para consumers, o shutdown deve seguir esta ordem exata:

1. Parar de consumir novas mensagens
2. Aguardar mensagens em processamento
3. Commit / ACK apenas após sucesso
4. Encerrar conexões com o broker
5. Finalizar o processo

### **6.4.5 Regra fundamental**

> [!error]
> Nunca dar ACK antes do processamento terminar.

Caso contrário:

- A mensagem é considerada processada
- O efeito colateral pode não ter ocorrido
- O sistema entra em estado inconsistente

### **6.4.6. Interação com Kubernetes**

No Kubernetes, o fluxo é previsível:

1. Pod recebe SIGTERM
2. terminationGracePeriodSeconds inicia
3. K8s aguarda a aplicação encerrar
4. Se exceder o tempo → SIGKILL

Isso exige que a aplicação:

- Escute sinais do sistema
- Respeite CancellationToken
- Tenha tempo suficiente para finalizar

### **6.4.7. Relação com Escalabilidade e Autoscaling**

Graceful shutdown é obrigatório para:

- Rolling updates
- Horizontal scaling
- KEDA / HPA
- Rebalanceamento de consumers

Sem graceful shutdown:

- Escalar para cima funciona
- Escalar para baixo quebra o sistema

### **6.4.8. Boas práticas essenciais**

- Sempre tratar SIGTERM
- Parar intake antes de finalizar
- Esperar processamento ativo
- Usar timeouts claros
- Nunca confiar em shutdown abrupto
- Testar shutdown em ambiente real

### **6.4.9. Anti-patterns comuns**

- process.exit() imediato
- Ignorar sinais do SO
- ACK antes de persistir
- Timeout infinito
- Assumir que o pod “morre rápido”

### **6.4.10. Graceful Shutdown e Idempotência**

Mesmo com graceful shutdown:

- Falhas podem ocorrer
- Mensagens podem ser reenviadas

Por isso:

> [!info]
> Graceful shutdown reduz riscos, idempotência elimina o problema.

Os dois sempre devem coexistir.

# 7. Gestão de Configuração

## 7.1. Hierarquia de Configuração

A configuração dos serviços deve seguir uma hierarquia clara de precedência, do menor para o maior nível de prioridade:

```
appsettings.json (defaults do serviço)
   ↓
appsettings.{Environment}.json (overrides por ambiente)
   ↓
Variáveis de ambiente (definidas no deployment)
   ↓
Hashicorp Vault (segredos e valores sensíveis)
```

O Options Pattern do .NET deve ser utilizado para acesso fortemente tipado às configurações. Configurações sensíveis (credenciais, chaves, tokens) nunca devem constar nos ficheiros `appsettings`, sendo obrigatório o uso do Vault.

## 7.2. Promoção entre Ambientes

A promoção de configurações entre ambientes (dev → staging → prod) deve seguir um processo controlado:

- Ficheiros `appsettings.{Environment}.json` devem ser versionados no repositório (sem valores sensíveis).
- Variáveis específicas de cada ambiente são geridas no sistema de deployment (Helm values, Kubernetes ConfigMaps).
- Segredos são provisionados via Vault com políticas de acesso por ambiente.
- Nenhuma configuração de produção deve ser aplicada manualmente — toda a promoção ocorre via pipeline CI/CD.

## 7.3. Feature Flags

Para funcionalidades que requerem ativação/desativação sem redeploy, deve ser definida uma estratégia de feature flags:

- Considerar solução dedicada (ex.: LaunchDarkly, Unleash) para feature flags dinâmicas com targeting por utilizador, tenant ou percentagem de tráfego.
- Para flags simples de on/off por ambiente, podem ser usadas variáveis de ambiente ou configuração hierárquica.
- Feature flags devem ter ciclo de vida definido: data de criação, responsável e critérios de remoção após estabilização da feature.

# 8. CI/CD

## 8.1. Pipeline de Build

Passos mínimos:

1. Restore (`dotnet restore`)
2. Build (`dotnet build -c Release`)
3. Testes unitários (`dotnet test`)
4. Testes de integração (opcional/conforme projeto)
5. Análise estática de código (SonarQube, etc.)

Dockerfile multi-stage (build + runtime).

Tagging semântico (`vX.Y.Z`, `branch`, `commit sha`).

## 8.2. Deploy

- Deploy em Kubernetes (salvo excepções aprovadas).
- Configuração via Helm/Manifest.
- Rollout com estratégia de rolling update / blue-green, conforme criticidade.

## 8.3. Execução de Testes de Integração com Testcontainers na Pipeline (CI/CD)

Para garantir consistência entre ambientes de desenvolvimento, validação e produção, é  recomendado que todos os testes de integração baseados em Testcontainers rodem automaticamente na pipeline CI.

Com Testcontainers, não é necessário subir containers manualmente. A pipeline só precisa garantir:

``` 
- Docker habilitado no runner
- .NET instalado
- Execução de dotnet test
```

Exemplo:

[https://ecom4isi.atlassian.net/wiki/spaces/DEVP/pages/edit-v2/5378539587#Exemplo-Completo-de-Pipeline-em-GitHub-Actions](https://ecom4isi.atlassian.net/wiki/spaces/DEVP/pages/edit-v2/5378539587#Exemplo-Completo-de-Pipeline-em-GitHub-Actions)

Para funcionar:

``` 
Docker Desktop (Windows), Podman ou Docker Engine (Linux) deve estar instalado
Serviço do Docker a correr
utilizador do runner com permissão para aceder ao Docker
```

Estrutura recomendada dos testes

``` 
tests/
 ├─ Modelo.Tests/                   → unit tests
 └─ Modelo.Tests.Integration/       → integration tests (Testcontainers)
```

Cada container é criado por classe de teste:

```csharp 
public class OrderRepositoryTests : IAsyncLifetime
{
    private readonly PostgreSqlContainer _pg =
        new PostgreSqlBuilder().WithDatabase("integration").Build();
}
```

Boas práticas em pipelines CI para Testcontainers

- Preferir `ubuntu-latest` (Linux) — containers iniciam muito mais rápido.
- Aumentar timeouts caso necessário:

```shell 
dotnet test -- RunConfiguration.TestSessionTimeout=600000
```

- Evitar paralelismo exagerado se subir muitos containers pesados
- Utilizar categorias:

```csharp 
[Trait("Category", "Integration")]
```

Com isso, todos os serviços passam a ter testes de integração reais rodando automaticamente no CI/CD, garantindo consistência, qualidade e segurança antes de cada merge ou deploy.

## 8.4. Gestão de Vulnerabilidades em Imagens Docker

A gestão de vulnerabilidades em imagens Docker segue o modelo DevSecOps, com responsabilidades claramente distribuídas entre Plataforma, Segurança e Squads de Desenvolvimento.

### 8.4.1 Princípios

- Todas as imagens Docker devem derivar de imagens base oficiais e aprovadas
- Nenhuma imagem pode ser promovida para produção sem scan automático de vulnerabilidades
- Vulnerabilidades são tratadas como dívida técnica de segurança, com SLA definido por severidade

### 8.4.2 Responsabilidades

| Atividade                                                         | Responsável                 |
| ----------------------------------------------------------------- | --------------------------- |
| Definição da imagem base oficial (.NET, distroless, alpine, etc.) | Plataforma / Infrastructure |
| Manutenção e atualização da imagem base                           | Plataforma / Infrastructure |
| Scanning automático de imagens (CI/CD)                            | DevSecOps / Plataforma      |
| Correção de vulnerabilidades da imagem base                       | Plataforma / Infrastructure |
| Correção de dependências da aplicação                             | Squad de Desenvolvimento    |
| Avaliação e aceite de risco (exceções)                            | Segurança / Arquitetura     |

### 8.4.3 Pipeline de Segurança

O pipeline CI/CD deve incluir, obrigatoriamente:

- Scan de vulnerabilidades da imagem Docker (ex.: Trivy, Snyk, Grype)
- Classificação por severidade (Critical, High, Medium, Low)
- Bloqueio automático para vulnerabilidade Critical ou High (sem exceção aprovada)



Exemplo de políticas:

> [!error]
> **Critical / High** → build falha (salvo exceção formal)

> [!warning]
> **Medium / Low** → alertas e backlog técnico



### 8.4.4 Atualização e Remediação

- Realizar rebuilds periódicos de imagens.
- Atualizar dependências vulneráveis da aplicação.
- Correções devem seguir SLA definidos por severidade.

# 9. Preocupações Transversais Adicionais

## 9.1. Rate Limiting e Throttling

Para proteger dependências downstream e prevenir abuso, todos os serviços devem definir estratégias de controlo de taxa:

**A nível de API Gateway (Gravitee):**

- Rate limiting por cliente/token para APIs expostas externamente.
- Quotas por plano de subscrição.

**A nível de aplicação:**

- Throttling de chamadas a dependências externas (bases de dados, APIs de terceiros, brokers) para evitar sobrecarga em cascata.
- Implementar backoff exponencial com jitter em retries.
- Configurar semáforos ou filas internas para limitar concorrência por dependência.

Combinar com Circuit Breaker (secção 6.2) para proteção em camadas: o throttling controla o ritmo de entrada, o circuit breaker protege quando a dependência falha.

## 9.2. Estratégia de Cache

O uso de cache deve ser explicitamente definido para reduzir latência e aliviar pressão sobre dependências:

**Níveis de cache permitidos:**

- **In-memory (IMemoryCache):** Dados de curta duração, locais ao pod. Não partilhado entre instâncias — usar apenas para dados imutáveis ou com baixa consistência exigida.
- **Distribuído (ex.: Redis):** Dados partilhados entre pods. Obrigatório para sessões, tokens de idempotência e resultados de queries frequentes.

**Regras de invalidação:**

- Definir TTL explícito para todas as entradas de cache.
- Preferir invalidação por evento (cache-aside com eventos de domínio) a TTL longo.
- Evitar cache de dados transacionais que exijam consistência forte.

**Consistência em cenários distribuídos:**

- Em ambientes multi-pod, alterações a dados cacheados devem propagar invalidação via evento ou pub/sub.
- Documentar o nível de consistência aceite para cada cache (eventual vs. forte).

## 9.3. Multi-tenancy

Para sistemas que suportam múltiplos tenants, as seguintes considerações são obrigatórias:

**Isolamento de dados:**

- A identificação do tenant (`tenantId`) deve ser propagada em todos os contextos: HTTP headers, mensagens de broker, tokens JWT e logs/traces.
- Queries à base de dados devem incluir sempre o filtro de `tenantId` para evitar data leakage entre tenants.

**Configuração por tenant:**

- Suportar overrides de configuração por tenant (ex.: limites, features, canais de notificação) via Notification Policy Store ou equivalente.

**Observabilidade:**

- O campo `tenantId` é obrigatório em logs estruturados e atributos de trace (ver secção 2.3 — Correlação distribuída).
- Dashboards e alertas devem permitir filtragem por tenant para diagnóstico isolado.

## 9.4. Privacidade de Dados (GDPR/LGPD)

Além da regra geral de não registar dados sensíveis (secção 2.5), os serviços devem cumprir os seguintes requisitos:

**Campos obrigatoriamente mascarados ou excluídos de logs e traces:**

- Dados pessoais identificáveis: nome completo, email, NIF, número de documento, data de nascimento.
- Dados de pagamento: número de cartão, CVV, IBAN.
- Credenciais e tokens de autenticação.
- Dados de saúde ou categorias especiais (RGPD Art. 9).

**Retenção de logs:**

- Logs contendo dados pessoais (mesmo anonimizados) devem respeitar políticas de retenção definidas pela equipa de Privacidade/Legal.
- Por omissão, não armazenar logs com PII por mais de 90 dias sem aprovação explícita.

**Auditoria e consentimento:**

- Operações sensíveis (acesso a dados pessoais, exportação, eliminação) devem gerar eventos de auditoria rastreáveis.
- O consentimento do utilizador para comunicações (email marketing, push) deve ser registado e auditável — ver secção 5.1.3.

---

## 9.5. Disaster Recovery para Componentes Stateful

Os componentes stateful da plataforma requerem estratégias explícitas de backup e recuperação:

**PostgreSQL:**

- Backups automáticos diários com retenção mínima de 30 dias.
- Point-in-Time Recovery (PITR) habilitado para bases de dados críticas.
- Testes de restore periódicos (pelo menos trimestrais) documentados.
- RTO e RPO definidos por criticidade do serviço.

**Solace (Broker):**

- Definir política de retenção de mensagens em filas persistentes para cenários de falha do consumidor.
- Em caso de falha catastrófica do broker, documentar o procedimento de replay ou reprocessamento a partir de fontes de origem.
- DLQs devem ser monitorizadas e ter procedimento de reprocessamento documentado.

**Princípio geral:**

- Todo o componente stateful deve ter RTO (Recovery Time Objective) e RPO (Recovery Point Objective) definidos, validados e documentados antes de entrar em produção.
- Os procedimentos de DR devem ser testados em ambiente não-produtivo pelo menos uma vez por semestre.
