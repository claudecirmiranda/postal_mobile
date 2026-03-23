Índice
------

- [DAS Backend API Messaging](#das-backend-api-messaging)
        - [Detalhe: Serviços internos de backend (APIs, BFFs, workers, integradores)](#detalhe-serviços-internos-de-backend-apis-bffs-workers-integradores)
        - [Histórico de Revisões](#histórico-de-revisões)
    - [1. Introdução](#1-introdução)
        - [1.1. Propósito e Objetivos](#11-propósito-e-objetivos)
        - [1.2. Público-Alvo](#12-público-alvo)
        - [1.3. Serviços Abrangidos](#13-serviços-abrangidos)
        - [1.4. Âmbito](#14-âmbito)
    - [2. Dependências e Referências](#2-dependências-e-referências)
    - [3. Regras de Utilização](#3-regras-de-utilização)
    - [4. Arquitetura da Tipologia](#4-arquitetura-da-tipologia)
        - [4.1. Camadas](#41-camadas)
        - [4.2. Fronteiras e Dependências](#42-fronteiras-e-dependências)
        - [4.3. Estratégia de Testes](#43-estratégia-de-testes)
        - [4.4. Fluxo End-to-End de Exemplo](#44-fluxo-end-to-end-de-exemplo)
        - [4.5. Padrões de Design para Resiliência](#45-padrões-de-design-para-resiliência)
            - [4.5.1. Retries em Publicação de Mensagens](#451-retries-em-publicação-de-mensagens)
            - [4.5.2. Retries em Consumo de Mensagens](#452-retries-em-consumo-de-mensagens)
            - [4.5.3. Circuit Breaker](#453-circuit-breaker)
            - [4.5.4. Outbox Pattern](#454-outbox-pattern)
        - [4.6. Versionamento de Eventos](#46-versionamento-de-eventos)
        - [4.7. Processamento em Background (Jobs)](#47-processamento-em-background-jobs)
    - [5. Estrutura do Projeto](#5-estrutura-do-projeto)
    - [6. Tecnologias Utilizadas](#6-tecnologias-utilizadas)
        - [6.2. Boas Práticas em Cliente de Mensageria](#62-boas-práticas-em-cliente-de-mensageria)
            - [Confiabilidade e Garantias de Entrega](#confiabilidade-e-garantias-de-entrega)
            - [Resiliência e Tolerância a Falhas](#resiliência-e-tolerância-a-falhas)
            - [Desempenho e Escalabilidade](#desempenho-e-escalabilidade)
            - [Contratos de Mensagem](#contratos-de-mensagem)
            - [Segurança na Mensageria](#segurança-na-mensageria)
        - [6.3. Quando Utilizar gRPC](#63-quando-utilizar-grpc)
        - [6.4. Gestão de Configurações Aplicacionais](#64-gestão-de-configurações-aplicacionais)
            - [6.4.1. Princípios](#641-princípios)
            - [6.4.2. Exemplo – appsettings.json](#642-exemplo-appsettingsjson)
            - [6.4.3. Classe de Options](#643-classe-de-options)
            - [6.4.4. Binding e Validação no Startup](#644-binding-e-validação-no-startup)
            - [6.4.5. Uso via IOptions / IOptionsMonitor](#645-uso-via-ioptions-ioptionsmonitor)
    - [7. Segurança](#7-segurança)
    - [8. Infraestrutura](#8-infraestrutura)
    - [9. Padrões e Princípios Arquiteturais](#9-padrões-e-princípios-arquiteturais)
        - [9.1. Princípios Fundamentais](#91-princípios-fundamentais)
        - [9.2. Diretrizes Gerais](#92-diretrizes-gerais)
        - [9.3. Governança](#93-governança)
        - [9.4. Quando Utilizar API Assíncrona ou Evento](#94-quando-utilizar-api-assíncrona-ou-evento)
        - [9.5. Gestão de Erros de Domínio](#95-gestão-de-erros-de-domínio)
    - [10. Observabilidade](#10-observabilidade)
        - [10.1. Instrumentação em .NET com OpenTelemetry](#101-instrumentação-em-net-com-opentelemetry)
            - [Auto-instrumentação (preferencial)](#auto-instrumentação-preferencial)
            - [Instrumentação manual (quando necessária)](#instrumentação-manual-quando-necessária)
        - [10.2. Logging Estruturado com Microsoft.Extensions.Logging](#102-logging-estruturado-com-microsoftextensionslogging)
            - [Regras](#regras)
        - [10.3. Correlação em Serviços Backend](#103-correlação-em-serviços-backend)
        - [10.4. Métricas Mínimas Esperadas em Backends](#104-métricas-mínimas-esperadas-em-backends)
            - [APIs / BFFs](#apis-bffs)
            - [Workers / Mensageria](#workers-mensageria)
            - [Dependências Externas](#dependências-externas)
        - [10.5. Boas Práticas Específicas para .NET](#105-boas-práticas-específicas-para-net)
    - [Apêndice](#apêndice)
        - [Template de Projeto](#template-de-projeto)
        - [Convenções de Nomenclatura C#](#convenções-de-nomenclatura-c)

# DAS Backend API Messaging

### Detalhe: Serviços internos de backend (APIs, BFFs, workers, integradores)

---

### Histórico de Revisões

| Versão | Data       | Autor(es)         | Resumo das Mudanças                                                                                     |
|--------|------------|-------------------|---------------------------------------------------------------------------------------------------------|
| 1.0    | 21/11/2025 | Thiago Giraldes   | Criação inicial do documento de arquitetura de referência.                                              |
| 2.0    | 16/12/2025 | Thiago Giraldes   | Ajustes de Formatação para uniformizar todos os DAS. Ajustes conforme comentários deixados no Confluence.|
| 3.0    | 17/12/2025 | Thiago Giraldes   | Ajustes conforme comentários deixados no Confluence.                                                    |
| 3.1    | 05/04/2026 | Felipe Klussmann  | Pequenos ajustes.                                                                                       |

> **Nota editorial:** Confirmar se as datas das versões 1.0–3.0 (novembro/dezembro 2025) estão alinhadas com o ciclo de vida atual (contexto de 2026). Garantir que a versão 3.1 reflete o estado atual do documento em produção.

---

## 1. Introdução

### 1.1. Propósito e Objetivos

Este documento define as diretrizes técnicas e de arquitetura para desenvolvimento, evolução e operação de backends internos em .NET, com foco em:

- APIs REST / BFFs em **ASP.NET Core LTS**
- Serviços orientados a domínio (DDD / Clean Architecture)
- Arquitetura orientada a eventos (Event-Driven Architecture – EDA)
- Uso de Solace como broker de mensagens
- Integrações com bases de dados, cache, serviços externos e frontends corporativos

O objetivo é servir como **documento de arquitetura de referência** para equipas de backend, arquitetura e DevOps, garantindo consistência entre os serviços.

### 1.2. Público-Alvo

- Engenharia
- Suporte
- Arquitetura

### 1.3. Serviços Abrangidos

Esta arquitetura aplica-se a:

- Novas APIs REST internas e externas
- BFFs (Backends for Frontends) para web/mobile
- Evolução/migração de monólitos legados para modelo modular/microserviços

### 1.4. Âmbito

Os seguintes objetivos visam endereçar os problemas relatados na etapa de Assessment.

| Objetivo Arquitetural                              | Problema Endereçado                                                                                                                                   | Solução Proposta                                                                                                                                                                                                                                                                                                                                          |
|----------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Modernização contínua e eliminação de legados**  | Tecnologias obsoletas (PHP 5, .NET Framework 4.x, Oracle Forms); dificuldade de manutenção e falta de profissionais especializados.                    | Definir arquiteturas modulares e evolutivas, divididas por contexto, que permitam a substituição incremental de componentes legados, garantindo compatibilidade, baixo risco e continuidade operacional. A modernização pode ocorrer através de: evolução de monólitos para monólitos modulares; extração progressiva de componentes para serviços independentes; adoção de integração assíncrona e mensageria orientada a eventos para reduzir acoplamento. Atualização contínua para versões LTS suportadas é mandatória. |
| **Segurança by default**                           | Fluxo de autenticação e autorização complexo e instável devido à coexistência de múltiplos Identity Providers; falta de aplicação consistente de segurança. | Implementar segurança aplicada por padrão no backend (middleware ASP.NET Core + autenticação padronizada); segredos armazenados em cofres seguros.                                                                                                                                                                                                         |
| **Observabilidade desde o design**                 | Documentação deficiente; pouca visibilidade de impacto e métricas; suporte e operação precisam "mergulhar no código" durante incidentes.              | Implantar **OpenTelemetry + Prometheus + Grafana/Jaeger** para métricas e tracing distribuído; logs estruturados com MS.Extensions.Logging (JSON) e **CorrelationId** desde a origem.                                                                                                                                                                    |
| **Automação de ponta a ponta (CI/CD)**             | Deploys pouco automatizados e com erros básicos; resistência em seguir processos de mudança; ausência de pipelines padronizados.                      | Criar pipelines únicos com build, testes (xUnit, Testcontainers), análise estática e deploy automatizados; integração com SonarQube e Swagger para documentação automática.                                                                                                                                                                               |
| **Governança de parceiros e retenção de conhecimento** | Integrações de sistemas externos feitas diretamente pelo negócio sem seguir padrões internos; falta de documentação; risco de "caixa preta".       | Estabelecer contrato arquitetural e documentação via Swagger/NSwag e ARDOQ; obrigatoriedade de seguir padrões de stack e arquitetura definida.                                                                                                                                                                                                            |

---

## 2. Dependências e Referências

> **Nota editorial:** Os links abaixo parecem ser placeholders copiados — todos apontam para o mesmo URL. Verificar e corrigir para os URLs específicos de cada tópico (Gateway, Messaging, Secrets, Identity).

- API Gateway Platform — *(verificar URL específico)*
- Streaming & Messaging Platform — *(verificar URL específico)*
- Secrets Management — *(verificar URL específico)*
- Identity, Authentication & Authorization — *(verificar URL específico)*

---

## 3. Regras de Utilização

Cada tipologia terá abordados aspetos específicos, incluindo, sempre que aplicável, referências a diretrizes, padrões gerais e boas práticas.

Este documento não é um conjunto de regras inflexíveis, mas sim um guia de **fortes recomendações**. Desvios devem ser justificados, aprovados por Arquitetura/DevPlatforms e registados num ADR.

---

## 4. Arquitetura da Tipologia

### 4.1. Camadas

1. **Apresentação (API / BFF)**
   1. ASP.NET Core Controllers.
   2. Autenticação, autorização, validação de entrada.
   3. Não contém lógica de domínio.

2. **Aplicação (Application)**
   1. Casos de uso (Commands/Queries).
   2. Orquestra fluxo de operações: chama repositórios, serviços de domínio, mensageria.
   3. Conhece DTOs de entrada/saída.

3. **Domínio (Domain)**
   1. Entidades, Value Objects, agregados, regras de negócio.
   2. Eventos de domínio.
   3. Interfaces de repositório (sem implementação).

4. **Infraestrutura (Infrastructure)**
   1. Implementação de repositórios (EF Core).
   2. DbContext, migrations.
   3. Serviços externos (HTTP clients, etc.).
   4. Configuração de IoC/DI.

### 4.2. Fronteiras e Dependências

Dependências e fronteiras (camadas) dentro de um projeto de desenvolvimento de software.

- `API` depende de `Application` e `Infrastructure`.
- `Application` depende apenas de `Domain`.
- `Domain` não depende de nenhum outro projeto.
- `Infrastructure` depende de `Domain` e implementa as interfaces definidas em `Application`/`Domain` (para implementar os contratos que eles definem, como `IEventPublisher`, `IOrderRepository`, etc.). A ligação é feita via composição (DI) no arranque da aplicação.

> **Nota arquitetural:** A `Infrastructure` deve depender apenas das **interfaces/contratos** definidos em `Application`/`Domain` — nunca de tipos concretos de lógica de negócio ou casos de uso. Isto garante a inversão de dependência e evita acoplamento circular.

### 4.3. Estratégia de Testes

- Unit tests (Domain/Application)
- Integration tests (Infra, messaging, DB)
- Contract tests (API/events)

Observar estratégia e implementação de testes em: https://ecom4isi.atlassian.net/wiki/spaces/DEVP/pages/5483724895

### 4.4. Fluxo End-to-End de Exemplo

```plaintext
[1] Cliente chama POST /api/orders
   → Modelo.API (OrdersController) valida e manda CreateOrderCommand pro Wolverine
[2] Application (CreateOrderCommandHandler)
   → Cria entidade Order
   → Persiste via IOrderRepository + IUnitOfWork
   → Dispara OrderCreatedEvent
   → Tópico modelo.orders.order-created.v1 recebe o evento
[4] Serviço B (outro microserviço)
   → Aplica lógica de negócio (ex.: criar fatura)
[5] Solace (event mesh)
   → Tópico corp/modelo/orders/order-created/v1 recebe espelho do evento
[6] Sistema legado / integração corporativa
   → SolaceOrderCreatedConsumer lê o evento
   → Atualiza sistemas externos / ERP / etc.
```

### 4.5. Padrões de Design para Resiliência

Mensageria sem estratégia de **resiliência** vira um gerador de problemas difíceis de rastrear. Aqui definimos como tratar **falhas temporárias**, **indisponibilidade de brokers** e **erros em consumidores**.

#### 4.5.1. Retries em Publicação de Mensagens

- **Retries com backoff exponencial** para falhas transitórias (ex.: timeout, conexão recusada, erro 5xx).
- **Limite máximo de tentativas**, para evitar tempestades de retries.
- **Logging estruturado** por tentativa, com `eventId`, `topic`, `retryCount`, `error`.

```csharp
public async Task PublishWithRetryAsync<TEvent>(
    string topic,
    TEvent @event,
    CancellationToken ct = default)
{
    var retries = 3;
    var delay = TimeSpan.FromSeconds(2);

    for (int attempt = 1; attempt <= retries; attempt++)
    {
        try
        {
            await PublishAsync(topic, @event, ct);
            return;
        }
        catch (Exception ex) when (attempt < retries)
        {
            _logger.LogWarning(ex,
                "Falha ao publicar mensagem. Tentativa {Attempt}/{Retries}. Topic={Topic}",
                attempt, retries, topic);

            await Task.Delay(delay, ct);
            delay = delay * 2; // backoff exponencial
        }
    }

    // Se chegou aqui, todas as tentativas falharam
    _logger.LogError(
        "Falha definitiva ao publicar mensagem após {Retries} tentativas. Topic={Topic}",
        retries, topic);
    throw;
}
```

#### 4.5.2. Retries em Consumo de Mensagens

**Retries imediatos (in-memory)**

O consumidor tenta processar a mensagem **até 3 vezes** dentro do mesmo loop (retry local).

- Útil para erros transitórios (timeout, erro de rede, dependência lenta).
- Mantém o processamento rápido sem pressionar o broker.

**Fallback para Dead Letter Queue**

Se todas as tentativas falharem, a mensagem deve ser enviada para uma **DLQ** específica por evento:

`modelo.<boundedContext>.<evento>.dlq`

- **Solace:** `corp/modelo/<boundedContext>/<evento>/dlq`

**Exemplo de retry interno + fallback para DLQ**

```csharp
public async Task ProcessMessageAsync(string json, CancellationToken ct)
{
    const int maxAttempts = 3;

    for (int attempt = 1; attempt <= maxAttempts; attempt++)
    {
        try
        {
            var evt = JsonSerializer.Deserialize<OrderCreatedEvent>(json);
            // Chamar Application/Domain aqui
            await _domainHandler.HandleAsync(evt, ct);
            return;
        }
        catch (Exception ex) when (attempt < maxAttempts)
        {
            _logger.LogWarning(ex,
                "Erro ao processar mensagem. Tentativa {Attempt}/{MaxAttempts}",
                attempt, maxAttempts);
        }
    }

    _logger.LogError(
        "Mensagem enviada para DLQ após {MaxAttempts} tentativas.",
        maxAttempts);

    await _dlqPublisher.PublishAsync("modelo.orders.order-created.dlq", json, ct);
}
```

#### 4.5.3. Circuit Breaker

O **circuit breaker** é um padrão de resiliência usado para **proteger o sistema contra falhas em cascata** quando uma dependência externa (API, broker, BD, serviço remoto) começa a falhar ou responder lentamente.

Em vez de insistir em chamadas que vão falhar, o circuito **"abre"** temporariamente, bloqueando novas tentativas até que o sistema tenha chance de se recuperar.

**Estados do Circuit Breaker**

1. **Closed (Fechado)** — As chamadas fluem normalmente. Falhas são monitorizadas.
2. **Open (Aberto)** — Chamadas são bloqueadas imediatamente (*fail fast*). Evita consumo de recursos desnecessários.
3. **Half-Open (Meio-aberto)** — Após um tempo de espera, permite poucas chamadas de teste. Se tiver sucesso → volta para *Closed*. Se falhar → retorna para *Open*.

**Quando usar**

- Chamadas HTTP para serviços externos
- Producers/consumers de mensageria
- Integrações com APIs de terceiros
- Qualquer dependência **fora do seu controlo**

**Boas práticas**

- Sempre combinar com **timeouts** (circuit breaker não substitui timeout)
- Usar junto com **retry com backoff**
- Expor **métricas**: `circuit_open`, `failures`, `half_open`
- Não usar circuit breaker para **erros funcionais** (ex.: validação de input)

**Exemplo em C# usando Polly**

A biblioteca **Polly** é o padrão de mercado em .NET para resiliência.

Observar exemplo em: https://ecom4isi.atlassian.net/wiki/spaces/DEVP/pages/5483724895

#### 4.5.4. Outbox Pattern

O Outbox Pattern garante consistência entre a base de dados e o envio de mensagens para um broker.

A aplicação grava o dado e a mensagem numa tabela *outbox* dentro da **mesma transação**. Depois, um processo assíncrono lê essa tabela e envia a mensagem ao broker, garantindo que nada se perde mesmo em caso de falhas.

> **Nota arquitetural:** Especificar o mecanismo de relay responsável pelo envio das mensagens da tabela *outbox* para o broker. As opções são: (a) **polling via `BackgroundService`** na mesma aplicação, ou (b) **processo/serviço externo** dedicado. A escolha deve ser documentada para garantir consistência operacional e evitar duplicação de envios.

Observar exemplo em: https://ecom4isi.atlassian.net/wiki/spaces/DEVP/pages/5483724895

### 4.6. Versionamento de Eventos

Os tópicos utilizam sufixo de versão (ex.: `.v1`, `corp/modelo/orders/order-created/v1`). Para garantir evolução segura do schema de eventos sem quebrar consumidores existentes, devem ser seguidas as seguintes diretrizes:

- **Eventos devem ser aditivos**: apenas adicionar campos opcionais; nunca remover ou renomear campos existentes numa versão publicada.
- **Breaking changes** requerem uma nova versão do tópico (ex.: `v2`), mantendo a versão anterior ativa durante o período de transição.
- Considerar o uso de um **Schema Registry** para validar contratos de mensagem em tempo de publicação/consumo.

> **Nota:** Embora o documento mencione `v1` nos tópicos, não havia diretrizes explícitas para evolução de schema. Esta secção endereça essa lacuna.

### 4.7. Processamento em Background (Jobs)

Nem todo processamento assíncrono precisa (ou deve) acontecer dentro da thread principal ou request HTTP. Utilizar uma task ou thread em background permite executar processamento assíncrono sem bloquear o fluxo principal da aplicação:

- quando usar (tarefas longas, retries, batch)
- boas práticas (idempotência, cancelamento, observabilidade)

Mais detalhes em: https://ecom4isi.atlassian.net/wiki/x/qQETOAE

---

## 5. Estrutura do Projeto

```plaintext
/src
  /Modelo.API
      Controllers/
      Middlewares/
      Filters/
      Extensions/
      Config/
      Program.cs
      appsettings.json

  /Modelo.Application
      DTOs/
      Commands/
          Orders/
      Queries/
      Handlers/
          Orders/
      Services/
      Interfaces/

  /Modelo.Domain
      Entities/
          Order.cs
      ValueObjects/
      Aggregates/
      Interfaces/
          IRepository.cs
          IUnitOfWork.cs
          IOrderRepository.cs
      Events/
          OrderCreatedEvent.cs
      Exceptions/
      Specifications/

  /Modelo.Infrastructure
      Context/
          AppDbContext.cs
      Mappings/
      Repositories/
          OrderRepository.cs
      Services/
      Migrations/
      Configurations/
      IoC/
          DependencyInjection.cs
      Messaging/
          Solace/
              SolaceEventPublisher.cs
              SolaceOrderCreatedConsumer.cs
      BackgroundJobs/
          Hangfire/
              HangfireBackgroundJobScheduler.cs
              HangfireJobs/
                  ReprocessarFaturasJob.cs
                  EnviarEmailsJob.cs

/tests
  /Modelo.Tests
      Domain/
      Application/
      Infrastructure/
      API/
```

| Diretório / Ficheiro            | Papel Principal                                                                                                       |
|---------------------------------|-----------------------------------------------------------------------------------------------------------------------|
| /src/Modelo.API                 | Camada de entrada da aplicação (Interface Adapters). Expõe endpoints REST/HTTP, configurações e integrações externas. |
| Controllers/                    | Pontos de entrada da API, responsáveis por orquestrar chamadas para Application Layer sem conter lógica de negócio.   |
| Middlewares/                    | Tratamento transversal (logging, autenticação, observabilidade, exceções).                                            |
| Filters/                        | Filtros de requisição/resposta, validação e cross-cutting concerns.                                                   |
| Extensions/                     | Métodos de extensão para facilitar configuração e reutilização.                                                       |
| Config/                         | Configurações específicas da API (ex.: Swagger, CORS, observabilidade).                                               |
| Program.cs                      | Ponto inicial da aplicação, configuração do host.                                                                     |
| appsettings.json                | Configurações externas (conexões, mensageria, observabilidade).                                                       |
| **/src/Modelo.Application**     | Camada de aplicação (Application Layer). Orquestra casos de uso, sem lógica de domínio.                               |
| DTOs/                           | Objetos de transporte entre camadas (entrada/saída).                                                                  |
| Commands/Orders/                | Comandos que representam intenções de mudança de estado (ex.: criar pedido).                                          |
| Queries/                        | Consultas de leitura, seguindo CQRS.                                                                                  |
| Handlers/Orders/                | Manipuladores de comandos/queries, coordenam execução de casos de uso.                                                |
| Services/                       | Serviços de aplicação, encapsulam lógica de orquestração.                                                             |
| Interfaces/                     | Contratos que definem dependências externas (ex.: serviços de mensageria).                                            |
| **/src/Modelo.Domain**          | Camada de domínio (Core Domain). Contém regras de negócio e invariantes.                                              |
| Entities/Order.cs               | Entidades centrais do domínio, com identidade e comportamento.                                                        |
| ValueObjects/                   | Objetos de valor imutáveis, sem identidade própria.                                                                   |
| Aggregates/                     | Raízes de agregados que garantem consistência de invariantes.                                                         |
| Interfaces/                     | Contratos de persistência e abstrações (Repository, UnitOfWork).                                                      |
| Events/OrderCreatedEvent.cs     | Eventos de domínio para suportar EDA e integração assíncrona.                                                         |
| Exceptions/                     | Exceções específicas do domínio.                                                                                      |
| Specifications/                 | Padrão Specification para regras de negócio reutilizáveis.                                                            |
| **/src/Modelo.Infrastructure**  | Camada de infraestrutura (Infra Layer). Implementa interfaces do domínio e application.                               |
| Context/AppDbContext.cs         | Contexto de persistência (EF Core).                                                                                   |
| Mappings/                       | Configuração de mapeamento ORM.                                                                                       |
| Repositories/OrderRepository.cs | Implementação concreta dos repositórios.                                                                              |
| Services/                       | Serviços de infraestrutura (ex.: envio de e-mails, integração externa).                                               |
| Migrations/                     | Scripts de migração de base de dados.                                                                                 |
| Configurations/                 | Configurações adicionais de infraestrutura.                                                                           |
| IoC/DependencyInjection.cs      | Configuração de injeção de dependência para infraestrutura.                                                           |
| Messaging/Solace/               | Publicadores e consumidores de eventos via Solace.                                                                    |
| **/tests/Modelo.Tests**         | Testes automatizados para garantir qualidade e evolução segura.                                                       |
| Domain/                         | Testes de regras de negócio (unitários).                                                                              |
| Application/                    | Testes de casos de uso e handlers.                                                                                    |
| Infrastructure/                 | Testes de integração com BD, mensageria, serviços externos.                                                           |
| API/                            | Testes de contrato e integração da API.                                                                               |

Observar exemplos de implementação e casos de uso de camadas em: https://ecom4isi.atlassian.net/wiki/spaces/DEVP/pages/5483724895

---

## 6. Tecnologias Utilizadas

| Categoria          | Tecnologia                                  | Observações                                                                 |
|--------------------|---------------------------------------------|-----------------------------------------------------------------------------|
| Linguagem          | C#                                          | Padrão para todos os serviços backend.                                      |
| Runtime            | .NET LTS                                    | Long Term Support; foco em performance.                                     |
| Web Framework      | ASP.NET Core                                | APIs REST, BFFs, webhooks.                                                  |
| Arquitetura        | DDD + Clean Architecture                    | Domain / Application / Infrastructure / API.                                |
| ORM                | Entity Framework Core LTS                   | Regras de acesso a dados, migrations.                                       |
| DB Relacional      | PostgreSQL                                  | Preferencial para novos serviços; para serviços em contexto de legado, a escolha pode variar — confirmar com a equipa de Arquitetura. |
| Mensageria         | Solace (SolaceSystems.SolClient)            | EDA, integração entre serviços, event mesh.                                 |
| DI                 | Microsoft.Extensions.DependencyInjection    | Injeção de dependência nativa do .NET.                                      |
| Logging            | MS.Extensions.Logging                       | Logs estruturados.                                                          |
| Observabilidade    | OpenTelemetry + Prometheus + Grafana        | Métricas e tracing distribuído.                                             |
| Testes             | xUnit, FluentAssertions, Testcontainers     | Testes unitários e de integração.                                           |
| Documentação API   | Swashbuckle (Swagger) / NSwag               | Documentação e geração de clientes.                                         |

> **Nota editorial:** A tabela anterior indicava "Depende do domínio / legado" para PostgreSQL, o que criava ambiguidade face à indicação de C# como "Padrão para todos os serviços backend". A observação foi clarificada acima.

### 6.2. Boas Práticas em Cliente de Mensageria

Um cliente de mensageria (producer/consumer) é um ponto crítico da arquitetura orientada a eventos. Erros nesse nível geram efeitos em cascata: mensagens perdidas, duplicadas, atrasos, backpressure e falhas difíceis de diagnosticar. Abaixo estão as principais boas práticas, organizadas por responsabilidade.

#### Confiabilidade e Garantias de Entrega

- **Confirmações (acks)** explícitas após processamento bem-sucedido; evitar *auto-ack* sem controlo.
- **Retries com backoff exponencial** e *jitter* para evitar thundering herd.

#### Resiliência e Tolerância a Falhas

- **Timeouts bem definidos** em conexões, publish e consume (nunca infinitos).
- **Dead Letter Queue (DLQ)** para mensagens inválidas ou que excederam tentativas.
- **Bulkhead**: limite de concorrência por tópico/fila para evitar saturação global.

#### Desempenho e Escalabilidade

- **Batching** (quando o broker suporta) para reduzir overhead de rede.
- **Controlo de concorrência**: consumidores paralelos, mas respeitando ordem quando necessário.
- **Prefetch / fetch size** ajustado ao payload e à latência esperada.
- **Backpressure**: pause/resume de consumo quando o processamento atrasa.

#### Contratos de Mensagem

- **Headers padronizados**: `correlationId`, `causationId`, `eventType`, `version`, `timestamp`. Obrigatórios e aplicados por middleware/decorator.
- **Evitar payloads gigantes**; preferir referências (ex.: IDs) quando possível.

#### Segurança na Mensageria

Para garantir a integridade das comunicações via Solace, os seguintes requisitos de segurança devem ser observados:

- **Autenticação de clientes**: todos os producers e consumers devem autenticar-se no broker (ex.: SASL ou certificados de cliente).
- **Encriptação em trânsito**: as ligações ao Solace devem usar TLS/mTLS.
- **Controlo de acesso por tópico**: definir ACLs para restringir quais serviços podem publicar ou consumir em cada tópico.

> Para detalhes completos, consultar o documento de Segurança em: *(verificar URL específico)*.

### 6.3. Quando Utilizar gRPC

gRPC é recomendado para:

- Comunicação síncrona de alta performance entre serviços internos.
- Cenários de baixo overhead com necessidade de contrato fortemente tipado.
- Streaming bidirecional.
- Integração entre aplicações em diferentes linguagens com alto throughput.

**Exemplo básico de envio via gRPC**

```csharp
var channel = GrpcChannel.ForAddress("https://servico-interno");
var client = new OrdersGrpc.OrdersGrpcClient(channel);

var request = new CreateOrderRequest
{
    CustomerId = "123",
    TotalAmount = 150.0
};

var response = await client.CreateOrderAsync(request);
Console.WriteLine($"Pedido criado: {response.OrderId}");
```

### 6.4. Gestão de Configurações Aplicacionais

A gestão de configurações deve seguir o **Options Pattern do .NET**, garantindo:

- Forte tipagem
- Validação em tempo de startup
- Separação clara entre configuração e segredo
- Facilidade de troca de provider (local, cloud, vault)

#### 6.4.1. Princípios

- **Configurações não sensíveis** → Podem residir em `appsettings.json`
- **Segredos (passwords, tokens, keys)** → **Nunca** devem ser versionados. Devem ser providos via variáveis de ambiente ou Secret Manager.
- A Application Layer não deve aceder a `IConfiguration` diretamente.
- O acesso deve ser feito **exclusivamente via** `IOptions<T>` ou `IOptionsMonitor<T>`.

#### 6.4.2. Exemplo – appsettings.json

```json
{
  "Database": {
    "Host": "localhost",
    "Port": 5432,
    "Name": "modelo",
    "User": "modelo_user"
  },
  "Solace": {
    "Host": "tcp://localhost:55555",
    "VpnName": "default",
    "Username": "default"
  }
}
```

> ⚠️ **Passwords, connection strings completas e tokens não devem estar contidas neste ficheiro.**

#### 6.4.3. Classe de Options

```csharp
public sealed class SolaceOptions
{
    public string Host { get; init; } = default!;
    public string VpnName { get; init; } = default!;
    public string Username { get; init; } = default!;
}
```

#### 6.4.4. Binding e Validação no Startup

```csharp
builder.Services
    .AddOptions<SolaceOptions>()
    .Bind(builder.Configuration.GetSection("Solace"))
    .ValidateDataAnnotations()
    .ValidateOnStart();
```

#### 6.4.5. Uso via IOptions / IOptionsMonitor

```csharp
public sealed class SolaceEventPublisher : IEventPublisher
{
    private readonly SolaceOptions _options;

    public SolaceEventPublisher(IOptions<SolaceOptions> options)
    {
        _options = options.Value;
    }
}
```

---

## 7. Segurança

Observar definições globais em: *(verificar URL específico do documento de segurança)*

> **Nota arquitetural:** Para o contexto específico de mensageria, os requisitos de segurança do Solace (autenticação de clientes, encriptação em trânsito, ACLs por tópico) estão resumidos na Secção 6.2 deste documento, mesmo que os detalhes completos residam no documento de segurança corporativo.

---

## 8. Infraestrutura

Observar definições globais em: *(verificar URL específico)*

---

## 9. Padrões e Princípios Arquiteturais

A arquitetura segue princípios essenciais que garantem consistência e evolução sustentável.

### 9.1. Princípios Fundamentais

**Domínio em primeiro lugar (DDD)**
- Modelo de domínio como fonte da verdade para regras de negócio.
- Linguagem global partilhada entre negócio e tecnologia.
- Modelagem orientada ao negócio e centrada no domínio.

**Clean Architecture / Ports & Adapters**
- Separação clara entre Domínio, Aplicação, Infraestrutura e Apresentação.
- Dependências apontam para o centro (API → Application → Domain); a Infrastructure implementa contratos definidos em Domain/Application.

**Event-Driven Architecture (EDA)**
- Uso de eventos de domínio e/ou de integração para comunicar mudanças relevantes.

**Serviços pequenos, claros e evolutivos**
- Foco em contextos claros.
- Componentes desacoplados e comunicando-se por interfaces.
- Evitar serviços multifunção com vários contextos.

**Observabilidade by design**
- Logs estruturados, métricas e tracing desde o início.
- CorrelationId em toda cadeia de chamada.

**Segurança por padrão**
- Autenticação e autorização aplicadas "by default".
- Segredos sempre em vault apropriado.

**Automação de ponta a ponta**
- Build, testes, análise estática e deploy automatizados em pipelines.

### 9.2. Diretrizes Gerais

- Evitar acoplamento desnecessário e dependências cíclicas.
- Manter serviços e handlers pequenos, focados e coesos.
- Aplicar resiliência e tratar falhas como parte natural do fluxo.

### 9.3. Governança

- Revisão técnica contínua.
- Conformidade com padrões corporativos.
- Garantia de testabilidade em todos os níveis.

### 9.4. Quando Utilizar API Assíncrona ou Evento

**Utilizar API Assíncrona quando:**
- Existe relação direta entre quem chama e quem responde.
- O producer precisa saber se a requisição foi aceite.
- O fluxo pode ser processado em background, mas continua a ser uma interação ponto-a-ponto (ex.: 202 Accepted + endpoint de estado / callback).
- Há necessidade de controlo de erros e contratos mais explícitos entre os serviços.

**Utilizar Evento quando:**
- O producer não deve conhecer os consumidores.
- Uma ação pode gerar efeitos em múltiplos serviços.
- Os consumidores podem processar em ritmos diferentes.
- O sistema precisa de alto desacoplamento e escala independente.

### 9.5. Gestão de Erros de Domínio

Os erros de domínio devem ser mapeados de forma consistente para os seus equivalentes na camada de saída:

- **Erros de domínio → HTTP**: as exceções de domínio (ex.: `OrderNotFoundException`, `InvalidOrderStateException`) devem ser capturadas no middleware de exceções da camada API e mapeadas para os códigos HTTP adequados (ex.: 404, 422). A lógica de mapeamento não deve residir nos Controllers.
- **Erros de domínio → Mensageria**: falhas no processamento de eventos que resultem de regras de negócio devem ser distinguidas de erros de infraestrutura. Erros funcionais não devem ser reenviados para retry nem enviados para DLQ sem enriquecimento de contexto.

---

## 10. Observabilidade

Observar o padrão em: *(verificar URL específico)*

Esta secção descreve como implementar observabilidade em serviços backend .NET, alinhado com os standards corporativos definidos no documento de Cross-cutting Concerns.

O objetivo é que qualquer API, BFF ou Worker tenha **logs estruturados, métricas e tracing distribuído ativos por defeito.**

---

### 10.1. Instrumentação em .NET com OpenTelemetry

Os serviços devem usar OpenTelemetry SDK e auto-instrumentação para capturar telemetria com o mínimo de código adicional.

#### Auto-instrumentação (preferencial)

Sempre que suportado pela plataforma, deve ser ativada para recolher automaticamente:

- Requests ASP.NET Core (entrada HTTP)
- Chamadas HttpClient (dependências externas)
- Métricas de runtime (GC, CPU, memória — quando disponível)

Isto garante cobertura base sem alterar código de negócio.

#### Instrumentação manual (quando necessária)

Adicionar spans ou métricas manuais apenas em pontos de negócio relevantes, por exemplo:

- Processamento de pagamentos
- Cálculo de preços
- Integrações críticas
- Processamento de mensagens long-running

Evitar instrumentar métodos triviais ou de infraestrutura interna.

---

### 10.2. Logging Estruturado com Microsoft.Extensions.Logging

Todos os serviços devem utilizar logging estruturado, nunca texto solto.

#### Regras

- Usar message templates com propriedades nomeadas.
- Não usar string interpolation para logs estruturados.
- Incluir sempre contexto técnico relevante.

**Exemplo correto**

```csharp
_logger.LogInformation(
    "Order created successfully. OrderId={OrderId} CustomerId={CustomerId}",
    order.Id,
    order.CustomerId);
```

**Exemplo incorreto**

```csharp
_logger.LogInformation($"Order created {order.Id}");
```

---

### 10.3. Correlação em Serviços Backend

Os serviços devem garantir que logs e traces estão correlacionados.

Em .NET, a correlação é facilitada quando OpenTelemetry está ativo, mas requer configuração explícita — nomeadamente, garantir que o `ActivityIdFormat` está definido para W3C e que os headers de propagação (`traceparent`, `tracestate`) são lidos e emitidos corretamente. Não é automático sem essa configuração.

Deve ser assegurado que:

- O `TraceId` está presente nos logs.
- O `CorrelationId` (quando existir no pedido ou mensagem) é propagado.

Em mensageria, os consumidores devem:

- Ler `correlationId` e `causationId` dos headers.
- Reemitir esses valores em logs e novos eventos.

---

### 10.4. Métricas Mínimas Esperadas em Backends

#### APIs / BFFs

- Latência por endpoint (p50/p95/p99)
- Taxa de erro (4xx/5xx)
- Throughput (requests/s)

#### Workers / Mensageria

- Tempo de processamento por mensagem
- Número de retries
- Mensagens enviadas para DLQ
- Backlog/lag de fila (quando disponível)

#### Dependências Externas

- Latência e taxa de erro por dependência (HTTP, base de dados, cache)

---

### 10.5. Boas Práticas Específicas para .NET

- Usar `ILogger<T>` via DI, nunca instanciar loggers manualmente.
- Garantir que exceções não tratadas são registadas com `LogError`.
- Não fazer logging de dados sensíveis (tokens, passwords, PII).
- Não fazer logs excessivos dentro de loops ou retries sem controlo.

---

## Apêndice

### Template de Projeto

Ver repositório base (exemplo): `modelo-backend-ddd` com:

```
Solution: Modelo.sln

Projetos:
- Modelo.Domain
- Modelo.Application
- Modelo.Infrastructure
- Modelo.API
```

### Convenções de Nomenclatura C#

- https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/identifier-names
- https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions

- **Namespaces:** `Modelo.<Camada>.<Contexto>`
- **Tópicos Solace:** `corp/modelo/<boundedContext>/<evento>/vN`
- **Branches Git:** `main`, `develop`, `feature/<nome>`, `hotfix/<nome>`

> ℹ️ **Aplicação de Exemplo:** https://github.com/mcdigital-devplatforms/sample-swrefarch-backend-api-messaging
