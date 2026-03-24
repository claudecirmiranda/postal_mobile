Índice
------

- [DAS Integration Service / Adapter](#das-integration-service-adapter)
- [Histórico de Revisões](#histórico-de-revisões)
- [1. Introdução](#1-introdução)
    - [1.1. Propósito e Objetivo](#11-propósito-e-objetivo)
    - [1.2. Público-Alvo](#12-público-alvo)
    - [1.3. Serviços Abrangidos](#13-serviços-abrangidos)
    - [1.4. Âmbito](#14-âmbito)
- [2. Dependências e Referências](#2-dependências-e-referências)
- [3. Regras de Utilização](#3-regras-de-utilização)
- [4. Arquitetura da Tipologia](#4-arquitetura-da-tipologia)
    - [4.1. Camadas](#41-camadas)
    - [4.2. Diagrama de Sequência](#42-diagrama-de-sequência)
    - [4.3. Princípios Arquiteturais](#43-princípios-arquiteturais)
    - [4.4. Casos de Uso](#44-casos-de-uso)
    - [4.5. Estratégia de Testes](#45-estratégia-de-testes)
- [5. Estrutura do Projeto](#5-estrutura-do-projeto)
- [6. Tecnologias Utilizadas](#6-tecnologias-utilizadas)
- [7. Segurança](#7-segurança)
    - [7.1. Requisitos Mínimos Obrigatórios](#71-requisitos-mínimos-obrigatórios)
- [8. Infraestrutura](#8-infraestrutura)
    - [8.1. CI/CD e DevOps](#81-cicd-e-devops)
- [9. Padrões e Princípios Arquiteturais](#9-padrões-e-princípios-arquiteturais)
    - [9.1. Paginação Padronizada](#91-paginação-padronizada)
    - [9.2. Padrões de Códigos de Erro](#92-padrões-de-códigos-de-erro)
    - [9.3. Padrões de Rate Limit](#93-padrões-de-rate-limit)
    - [9.4. Sugestões para Tamanho de Payload](#94-sugestões-para-tamanho-de-payload)
    - [9.5. Padrões de Timeout](#95-padrões-de-timeout)
    - [9.6. Controlo de Backpressure](#96-controlo-de-backpressure)
    - [9.7. Versionamento e Depreciação de APIs](#97-versionamento-e-depreciação-de-apis)
    - [9.8. Resumo em JSON para Contratos Internos](#98-resumo-em-json-para-contratos-internos)
    - [9.9. Contrato API-first (OpenAPI exemplo YAML)](#99-contrato-api-first-openapi-exemplo-yaml)
    - [9.10. Exemplo de Código – .NET 8 (Minimal API)](#910-exemplo-de-código-net-8-minimal-api)
    - [9.11. Integração para Data Analytics / Data Lake](#911-integração-para-data-analytics-data-lake)
- [10. Observabilidade](#10-observabilidade)
    - [10.1. Requisitos Mínimos Obrigatórios](#101-requisitos-mínimos-obrigatórios)
    - [10.2. O que Deve Ser Coletado](#102-o-que-deve-ser-coletado)
    - [10.3. Monitorização de SLA por Integração](#103-monitorização-de-sla-por-integração)
    - [10.4. Contexto Mínimo Obrigatório (labels)](#104-contexto-mínimo-obrigatório-labels)

# DAS Integration Service / Adapter

**Detalhe: Middleware de Integração (transformação de dados, sync jobs, adaptação de APIs)**

# Histórico de Revisões

| Versão | Data       | Autor(es)        | Resumo das Mudanças                      |
|--------|------------|------------------|------------------------------------------|
| 1.0    | 05/12/2025 | Jamil Costa      | Criação inicial do documento de arquitetura de referência. |
| 2.0    | 06/01/2026 | William Alves    | Adição de Observabilidade.               |
| 2.1    | 19/06/2026 | Felipe Klussmann | Pequenos ajustes e correções.            |

# 1. Introdução

## 1.1. Propósito e Objetivo

Este documento provê diretrizes arquiteturais e práticas recomendadas para times técnicos que desenvolvem, mantêm e operam serviços de integração do tipo **Integration Service / Adapter**. O objetivo principal é garantir **interoperabilidade**, **robustez**, **eficiência** e **modelo consistente** na comunicação entre sistemas distintos, reduzindo o acoplamento e aumentando a escalabilidade. Os owners das aplicações são autônomos e responsáveis por garantir o desenvolvimento e compliance com a arquitetura.

O principal caso de uso deste documento é o cenário em que não seja possível (ou não seja recomendável) a comunicação direta entre dois sistemas/serviços.

## 1.2. Público-Alvo

- Engenharia
- Suporte
- Arquitetura

## 1.3. Serviços Abrangidos

- Middleware para integração entre sistemas internos e/ou externos
- Transformação e enriquecimento de dados
- Serviços de sincronização periódica (sync jobs)
- Mapeamento e adaptação de APIs
- Conectores para protocolos diversos (REST, SOAP, MQ, SFTP, etc.)

## 1.4. Âmbito

Os seguintes objetivos visam endereçar os problemas relatados na etapa de Assessment.

| Objetivos Arquiteturais                          | Problemas Solucionados                                                        |
|--------------------------------------------------|-------------------------------------------------------------------------------|
| **Event-driven architecture**                    | Evita sobrecarga em APIs síncronas e melhora escalabilidade                   |
| **API-first e contrato versionado**              | Garante consistência e evita breaking changes                                 |
| **Padronização de paginação e códigos de erro**  | Mitiga falta de paginação e inconsistências em APIs externas                  |
| **Desacoplamento por adaptadores/mensageria**    | Reduz dependência direta entre aplicações                                     |
| **Rate-limit e payload limit**                   | Protege serviços contra sobrecarga                                            |
| **Observabilidade e rastreabilidade distribuída**| Facilita análise de problemas e auditoria em sistemas distribuídos            |

# 2. Dependências e Referências

Este tipo de serviço depende de alguns elementos fundamentais para operar com segurança e consistência:

- **Contratos de integração**: OpenAPI / AsyncAPI / XSD (para REST, eventos e SOAP).
- **Broker**: Solace.
- **API Gateway / IAM corporativo**: Autenticação, autorização, rate-limit e auditoria.
- **Ferramentas de observabilidade**: OpenTelemetry, Grafana, Prometheus.

> **Nota editorial:** As secções de Segurança (7), Infraestrutura (8) e Observabilidade (10) referenciam documentos externos. Verificar e corrigir os URLs específicos para cada tópico (Gateway, Messaging, Secrets, Identity) — os links atuais parecem ser placeholders repetidos.

# 3. Regras de Utilização

Cada tipologia terá abordados aspetos específicos, incluindo, sempre que aplicável, referências a diretrizes, padrões gerais e boas práticas.

Este documento não é um conjunto de regras inflexíveis, mas sim um guia de **fortes recomendações**. Desvios são permitidos, mas devem ser justificados e validados com Arquitetura e Developer Platform, e devidamente registados (ADR / exceção arquitetural).

# 4. Arquitetura da Tipologia

## 4.1. Camadas

A arquitetura do Middleware para integração é dividida em quatro camadas:

1. **Camada de Interface/Entrada (Inbound Layer)**
   1. Recebe chamadas via APIs, filas, eventos.
   2. Implementa autenticação, rate-limit e validação inicial.

2. **Camada de Processamento e Transformação (Core Adapter Layer)**
   1. Garantia de idempotência no consumo de mensagens.
   2. Aplicação de lógica de transformação.
   3. Enriquecimento de dados.
   4. Aplicação de contratos padrão (schemas, versionamento).

3. **Camada de Interface/Saída (Outbound Layer)**
   1. Comunicação com sistemas externos ou entrega de eventos.
   2. Cuidado com paginação em múltiplos consumidores.
   3. Tratamento de dados antes da persistência ou entrega.

4. **Observabilidade e Resiliência (Cross-cutting Concerns)**
   1. Logging estruturado.
   2. Tracing distribuído.
   3. Retry e tolerância a falhas.

> **Nota:** A Observabilidade e Resiliência são **preocupações transversais** (cross-cutting concerns) — não constituem uma camada física no fluxo de processamento, mas aplicam-se a todas as camadas acima.

## 4.2. Diagrama de Sequência

*(Diagrama de sequência disponível no Confluence — consultar o documento original para o diagrama do fluxo de comunicação entre camadas.)*

## 4.3. Princípios Arquiteturais

- **Event-driven first**: priorizar fluxo assíncrono.
- **API-first design** com documentação via OpenAPI/Swagger.
- **Desacoplamento** por mensageria e adaptadores intermediários.
- **Padronização** (quando aplicável a APIs síncronas): paginação (limit/offset ou cursor), `totalCount` apenas quando fizer sentido e tiver custo aceitável.
- **Resiliência**: circuit breaker, retry, timeout.
- **Segurança by design**: rate-limit, payload limit, autenticação forte e autorização granular.
- **Observabilidade** desde o desenvolvimento: Correlation ID, logs estruturados contextualizados.

## 4.4. Casos de Uso

- Integração de ERP com múltiplos CRMs via event-driven
- Transformação de dados XML para JSON para API unificada
- Sincronização de dados entre sistemas distribuídos
- API adaptadora para consumidores internos com paginação e caching
- Publicação de eventos de atualização para vários microserviços

## 4.5. Estratégia de Testes

- **Unit Tests** para mapeamento e regras de negócio
- **Contract Testing** para APIs e eventos
- **Integration Testing** com sistemas reais/stubs
- **Performance Testing** com grandes volumes e consumidores múltiplos
- **Resilience Testing** simulando falhas de sistemas externos
- **Security Testing** (SQLi, XSS, análise de payloads maliciosos)

# 5. Estrutura do Projeto

Já contemplado no [DAS Backend API](https://ecom4isi.atlassian.net/wiki/x/AQDeKgE).

# 6. Tecnologias Utilizadas

- **Broker/Event-driven**: Solace
- **APIs**: REST (ASP.NET Core em .NET LTS, ex.: .NET 8) / gRPC
- **Orquestração de containers**: Kubernetes
- **Agendamento de jobs**: Hangfire, CronJobs
- **Orquestração de workflows** (quando aplicável): Airflow
- **Monitoramento**: Prometheus, Grafana
- **Segurança**: mTLS, OAuth 2.0, JWT
- **Documentação**: OpenAPI, AsyncAPI

# 7. Segurança

Definições globais: *(verificar URL específico do documento de segurança corporativo)*

## 7.1. Requisitos Mínimos Obrigatórios

Independentemente dos detalhes no documento de segurança corporativo, todos os Integration Services/Adapters devem garantir:

- **Autenticação de todas as chamadas de entrada**: utilizar tokens JWT validados via IAM corporativo ou mTLS para comunicações serviço-a-serviço.
- **Encriptação em trânsito**: TLS obrigatório em todas as comunicações, tanto inbound como outbound.
- **Autorização granular**: aplicar princípio do menor privilégio; cada adapter deve ter apenas as permissões necessárias para os sistemas com que interage.
- **Segredos em vault**: credenciais, tokens e connection strings nunca em repositório de código ou configuração estática. Utilizar o Secret Manager corporativo com suporte a rotação automática.
- **Auditoria**: todas as chamadas de entrada e saída devem ser registadas com contexto suficiente para rastreabilidade (quem chamou, quando, o quê).

> Para rotação de credenciais, caching de segredos em runtime e procedimentos de fallback em caso de indisponibilidade do vault, consultar o documento de Secrets Management: *(verificar URL específico)*.

# 8. Infraestrutura

## 8.1. CI/CD e DevOps

Ver definições globais em: *(verificar URL específico)*

# 9. Padrões e Princípios Arquiteturais

## 9.1. Paginação Padronizada

- Query params:
  - `limit` (int): número de registros por página (default: 50, máximo: 200)
  - `offset` (int): posição inicial para busca
  - `totalCount` no payload para clientes saberem o total

> **Nota:** `totalCount` deve ser incluído apenas quando o custo da query for aceitável. Evitar queries `COUNT(*)` pesadas em tabelas com grande volume de dados — nesses casos, omitir ou disponibilizar `totalCount` de forma assíncrona/aproximada.

```json
{
  "data": [/* registros */],
  "pagination": {
    "limit": 50,
    "offset": 0,
    "totalCount": 15437
  }
}
```

## 9.2. Padrões de Códigos de Erro

**HTTP:**

```
200: sucesso
400: requisição inválida
401: não autenticado
403: sem autorização
404: recurso não encontrado
409: conflito (idempotência, estado inválido, versão/concorrência)
413: payload demasiado grande
429: rate limit excedido
500: erro interno
503: serviço indisponível
```

**Payload de erro padronizado:**

```json
{
  "errorCode": "INTEGRATION_TIMEOUT",
  "message": "O serviço de destino não respondeu a tempo",
  "details": {
    "destination": "CRMService",
    "timeoutMs": 3000
  }
}
```

## 9.3. Padrões de Rate Limit

**Objetivo:** evitar sobrecarga, abuso e garantir disponibilidade para todos os consumidores.

| Diretriz                   | Valor sugerido                              | Observações                                                               |
|----------------------------|---------------------------------------------|---------------------------------------------------------------------------|
| Limite por IP (público)    | 100 req/minuto                              | Ideal para APIs públicas com múltiplos clientes.                          |
| Limite por token/aplicação | 1000 req/minuto                             | Para APIs autenticadas, considerando volume esperado.                     |
| Burst Control              | Máx. 20 req/segundo                         | Evita picos súbitos que possam derrubar serviços.                         |
| Janela de tempo            | 1 minuto (Fixed Window ou Sliding Window)   | Usar algoritmos token-bucket ou leaky-bucket para maior precisão.         |
| Resposta de excesso        | HTTP `429 Too Many Requests`                | Payload JSON padronizado com campos `errorCode`, `message`, `retryAfter`. |

**Payload de erro padronizado:**

```json
{
  "errorCode": "RATE_LIMIT_EXCEEDED",
  "message": "Número máximo de requisições atingido. Tente novamente mais tarde.",
  "details": {
    "limitPerMinute": 100,
    "retryAfterSeconds": 60
  }
}
```

## 9.4. Sugestões para Tamanho de Payload

**Objetivo:** evitar tráfego e processamento excessivo, garantindo tempo de resposta e custo de rede reduzido.

| Diretriz                     | Valor sugerido                                                          | Observações                                                                       |
|------------------------------|-------------------------------------------------------------------------|-----------------------------------------------------------------------------------|
| Payload máximo para requisição | 1 MB para JSON                                                        | Adequado para trocas de dados comuns.                                             |
| Payload máximo para upload   | 10 MB para upload de ficheiros via streaming (exclui soluções dedicadas de MFT) | Usar endpoints específicos para upload e validação no server-side.        |
| Payload máximo para resposta | 2 MB para dados paginados                                               | Dividir resultados grandes usando paginação (default 50 registros, máx. 200).     |
| Negociação de conteúdo       | Suportar compressão (`gzip`)                                            | Reduz tráfego em conexões lentas.                                                 |

> **Nota:** Os valores acima são um baseline sugerido. Integrações com requisitos específicos (ex.: transferência de ficheiros médicos, documentos legais) podem necessitar de limites distintos. Ajustes devem ser validados com Arquitetura e registados em ADR quando justificados por requisitos de negócio.

**Boas práticas:**

- Validar tamanho *antes* de processar conteúdo.
- Responder com HTTP `413 Payload Too Large` em caso de excesso.
- Incluir no contrato (`OpenAPI`) a especificação de tamanho máximo permitido.

## 9.5. Padrões de Timeout

**Objetivo:** prevenir bloqueios e liberar recursos rapidamente.

| Tipo de Timeout                | Valor sugerido                                                        | Observações                                             |
|--------------------------------|-----------------------------------------------------------------------|---------------------------------------------------------|
| Timeout de requisição HTTP     | 30 segundos                                                           | Para operações que envolvem integrações externas.       |
| Timeout de leitura (read)      | 10 segundos                                                           | Tempo máximo para receber dados após a conexão inicial. |
| Timeout de escrita (write)     | 10 segundos                                                           | Tempo máximo para enviar dados ao servidor.             |
| Timeout de operações internas  | Configurar conforme SLA; idealmente ≤ 5 segundos para APIs síncronas | Evita encadeamento de atrasos.                          |
| Retry Policy                   | Máx. 3 tentativas com backoff exponencial                             | Implementar circuit breaker para evitar loops de falha. |

**Boas práticas:**

- Sempre propagar `timeoutMs` no log/erro, inclusive para chamadas downstream.
- Usar tracing distribuído para identificar gargalos de tempo.
- Tratar timeouts como erros recuperáveis ou integrar fallback quando aplicável.

## 9.6. Controlo de Backpressure

**Objetivo:** proteger o adapter e os sistemas de destino em cenários onde os consumidores estão lentos ou indisponíveis.

Quando um sistema de destino apresenta degradação, o adapter não deve acumular mensagens ou requisições indefinidamente — risco de esgotamento de memória e conexões.

**Diretrizes:**

- **Limite de fila interna**: definir um limite máximo de mensagens em processamento simultâneo por adapter (ex.: semáforo, channel com capacidade limitada).
- **Política de rejeição**: ao atingir o limite, responder com `HTTP 429` (para chamadas síncronas) ou `HTTP 503` com `Retry-After` (para indisponibilidade temporária).
- **Pause/resume em consumidores de mensageria**: implementar mecanismo de pausa no consumo de tópicos quando o processamento acumula além de um threshold configurável; retomar quando a carga normalizar.
- **Sinalização de backpressure em métricas**: expor o tamanho da fila e o estado do consumer (ativo/pausado) como métricas observáveis.

## 9.7. Versionamento e Depreciação de APIs

**Objetivo:** garantir evolução de contratos sem introduzir breaking changes para consumidores existentes.

**Política:**

- Versionar via path (ex.: `/v1/customers`, `/v2/customers`) ou header (`Accept: application/vnd.api+json;version=2`).
- Manter a versão anterior ativa por um **mínimo de 6 meses** após a publicação de uma nova versão, salvo acordo explícito com todos os consumidores.
- **Comunicação obrigatória**: notificar todos os consumidores registados antes de iniciar a depreciação de uma versão, com data de fim de suporte definida.
- Breaking changes (remoção de campos, alteração de tipos, renomeação de endpoints) requerem sempre uma nova versão principal.
- Alterações aditivas (novos campos opcionais, novos endpoints) podem ser feitas na versão corrente, desde que retrocompatíveis.

## 9.8. Resumo em JSON para Contratos Internos

Este formato pode ser incluído como parte de metadados da API:

```json
{
  "limits": {
    "rateLimitPerMinute": 100,
    "burstLimitPerSecond": 20,
    "maxPayloadRequestBytes": 1048576,
    "maxPayloadResponseBytes": 2097152,
    "httpTimeoutSeconds": 30,
    "readTimeoutSeconds": 10,
    "writeTimeoutSeconds": 10
  }
}
```

## 9.9. Contrato API-first (OpenAPI exemplo YAML)

```yaml
openapi: 3.0.3
info:
  title: Integration Adapter - ERP to CRM
  version: 1.0.0
paths:
  /customers:
    get:
      summary: Lista clientes com paginação
      parameters:
        - in: query
          name: limit
          schema:
            type: integer
          required: false
        - in: query
          name: offset
          schema:
            type: integer
          required: false
      responses:
        '200':
          description: Lista de clientes
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/CustomerList'
components:
  schemas:
    CustomerList:
      type: object
      properties:
        data:
          type: array
          items:
            $ref: '#/components/schemas/Customer'
        pagination:
          type: object
          properties:
            limit: { type: integer }
            offset: { type: integer }
            totalCount: { type: integer }
    Customer:
      type: object
      properties:
        id: { type: string }
        name: { type: string }
        email: { type: string }
```

## 9.10. Exemplo de Código – .NET 8 (Minimal API)

Podem validar um exemplo de código no documento de apoio: https://ecom4isi.atlassian.net/wiki/spaces/DEVP/pages/5410422839

## 9.11. Integração para Data Analytics / Data Lake

Para integrações cujo destino final seja um Data Lake ou plataforma analítica, o serviço deve priorizar a preservação da fidelidade aos dados de origem. A camada inicial de ingestão deve manter os dados o mais próximos possível da fonte, limitando-se a transformações técnicas e não funcionais (ex.: normalização de formato, enriquecimento com metadata operacional, padronização de envelope). Regras de negócio, agregações e enriquecimentos semânticos devem ser aplicados apenas em camadas analíticas posteriores.

Sempre que suportado pelo sistema de origem, o adapter deve publicar eventos incrementais (delta) em vez de snapshots completos, contendo apenas os campos alterados e os identificadores necessários para reconstrução do estado. Isso reduz volume de tráfego, custo de processamento e latência, além de melhorar a escalabilidade da plataforma de dados. É responsabilidade do consumidor analítico (camadas Silver/Gold) recompor o estado completo quando necessário.

# 10. Observabilidade

Observar o padrão corporativo em: *(verificar URL específico)*

## 10.1. Requisitos Mínimos Obrigatórios

Todos os Integration Services/Adapters devem, no mínimo:

- Expor métricas de **latência p95/p99**, **taxa de erro** e **throughput** por integração.
- Propagar **Correlation ID** em toda a cadeia de chamadas (inbound → processamento → outbound).
- Emitir **logs estruturados** com contexto mínimo obrigatório (ver Secção 10.4).
- Configurar **alertas automáticos** para falhas acima do threshold definido no SLA da integração.
- Expor estado do **circuit breaker** como métrica observável.

## 10.2. O que Deve Ser Coletado

**Tráfego (Volume)**

```
- Total de requisições recebidas (APIs)
- Total de eventos/mensagens consumidas
- Total de eventos/mensagens publicados
```

**Latência**

```
- Tempo total de processamento da integração
- Tempo de chamadas a sistemas externos (downstream)
- Percentis de latência (P95 / P99)
```

**Erros**

```
- Erros de validação/contrato (4xx)
- Erros técnicos (5xx)
- Timeouts
- Falhas de consumo/processamento de eventos
```

**Resiliência**

```
- Número de retries executados
- Falhas definitivas após retry máximo
- Estado do circuit breaker (aberto / fechado)
```

**Mensageria**

```
- Mensagens pendentes na fila
- Taxa de consumo por consumer
- Tempo médio de espera/processamento da mensagem
```

**SLA / Saúde da Integração**

```
- Taxa de sucesso por integração
- Disponibilidade do adapter
- Latência dentro vs. fora do SLA
```

## 10.3. Monitorização de SLA por Integração

Cada integração deve ter o seu SLA definido e monitorizado de forma individualizada. Para cada integração, deve ser especificado:

- **Disponibilidade alvo** (ex.: 99,5% em horário comercial).
- **Latência máxima aceitável** (ex.: p95 < 500 ms para chamadas síncronas).
- **Taxa de erro máxima** (ex.: < 0,1% de falhas definitivas).
- **Alertas configurados** no dashboard de observabilidade para notificar quando qualquer SLA for violado.

## 10.4. Contexto Mínimo Obrigatório (labels)

```
- Nome da integração / adapter
- Operação
- Sistema de destino
- Tipo de protocolo (HTTP, evento, batch)
```

---

> **Aplicação de Exemplo:** [sample-swrefarch-backend-api-messaging](https://github.com/mcdigital-devplatforms/sample-swrefarch-backend-api-messaging)
