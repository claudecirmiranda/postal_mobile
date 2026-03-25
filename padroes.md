# Documento de Arquitetura de Referência - Plataforma de Dispositivos Inteligentes Sonae - Android Embedded

## Histórico de Revisões

| Versão | Data       | Autor(es)         | Resumo das Mudanças                                      |
|--------|------------|-------------------|----------------------------------------------------------|
| 1.0    | 16/12/2025 | Kleber Santos     | Criação inicial do documento de arquitetura de referência. |
| 2.0    | 26/12/2025 | Kleber Santos     | Ajustes conforme comentários.                            |
| 2.1    | 03/02/2026 | Felipe Klussmann  | Pequenos ajustes.                                        |

Índice
------

- [1. Introdução](#1-introdução)
	- [1.1. Propósito e Objetivos](#11-propósito-e-objetivos)
	- [1.2. Público-Alvo](#12-público-alvo)
	- [1.3. Âmbito](#13-âmbito)
- [2. Dependências e referências](#2-dependências-e-referências)
- [3. Regras de Utilização](#3-regras-de-utilização)
- [4. Arquitetura da Tipologia](#4-arquitetura-da-tipologia)
	- [4.1. Diagrama de Contexto do Sistema](#41-diagrama-de-contexto-do-sistema)
	- [4.2. Arquitetura no dispositivo](#42-arquitetura-no-dispositivo)
	- [4.2.1. Camadas](#421-camadas)
	- [4.3. Comunicação](#43-comunicação)
	- [4.4. Fluxo de Comunicação](#44-fluxo-de-comunicação)
	- [4.5. Offline-First](#45-offline-first)
	- [4.6. Testes](#46-testes)
		- [4.6.1. Unitários](#461-unitários)
		- [4.6.2. Frameworks de Teste](#462-frameworks-de-teste)
		- [4.6.3. Estratégia de Testes para Dispositivos](#463-estratégia-de-testes-para-dispositivos)
- [5. Estrutura do Projeto](#5-estrutura-do-projeto)
- [6. Tecnologias Utilizadas](#6-tecnologias-utilizadas)
- [7. Segurança](#7-segurança)
	- [7.1. Componentes de Segurança](#71-componentes-de-segurança)
	- [7.2. Requisitos Mínimos de Segurança para Dispositivos IoT](#72-requisitos-mínimos-de-segurança-para-dispositivos-iot)
	- [7.3. Fluxo de Provisionamento e Ciclo de Vida de Certificados](#73-fluxo-de-provisionamento-e-ciclo-de-vida-de-certificados)
- [8. Infraestrutura](#8-infraestrutura)
	- [8.1. Topologia de Implantação](#81-topologia-de-implantação)
	- [8.2. Atualização de Software (OTA)](#82-atualização-de-software-ota)
	- [8.3. Estratégia de Rollout](#83-estratégia-de-rollout)
	- [8.4. Gerenciamento de Configuração e Segredos](#84-gerenciamento-de-configuração-e-segredos)
	- [8.5. Ciclo de Vida de Dispositivos](#85-ciclo-de-vida-de-dispositivos)
- [9. Padrões Arquiteturais Adotados](#9-padrões-arquiteturais-adotados)
	- [9.1. Event-Driven Architecture](#91-event-driven-architecture)
	- [9.2. CQRS (Command Query Responsibility Segregation)](#92-cqrs-command-query-responsibility-segregation)
	- [9.3. MVVM (Mobile)](#93-mvvm-mobile)
	- [9.4. Offline-First](#94-offline-first)
- [10. Observabilidade](#10-observabilidade)
	- [10.1. Métricas Chave](#101-métricas-chave)
	- [10.2. Métricas de Bateria e Eficiência Energética](#102-métricas-de-bateria-e-eficiência-energética)
	- [10.3. Privacidade de Dados em Dispositivo (GDPR)](#103-privacidade-de-dados-em-dispositivo-gdpr)
- [11. Plano de Contingência para Indisponibilidade de Cloud](#11-plano-de-contingência-para-indisponibilidade-de-cloud)

## 1. Introdução

### 1.1. Propósito e Objetivos

Este documento descreve a arquitetura de referência para a construção de uma plataforma unificada de dispositivos inteligentes (IoT) da Sonae baseada em Android Embedded. A arquitetura é projetada para permitir a comunicação bidirecional entre dispositivos inteligentes, aplicações móveis e serviços em nuvem, garantindo segurança, confiabilidade, escalabilidade e observabilidade em toda a cadeia de valor. O objetivo é criar um ecossistema coeso que modernize a forma como a Sonae interage com seus clientes através de dispositivos inteligentes, desde eletrodomésticos até sistemas de ponto de venda (POS) e dispositivos de loja.

Esta arquitetura define o padrão a ser seguido para todos os novos desenvolvimentos de dispositivos inteligentes baseados em Android Embedded. Ela abrange desde a estrutura do firmware do dispositivo, a comunicação com serviços em nuvem, a sincronização de dados, até as estratégias de atualização de software (OTA - Over-The-Air), segurança, monitoramento e observabilidade. A solução visa criar uma plataforma modular e extensível que permita a rápida integração de novos tipos de dispositivos, reduzindo o tempo de mercado e os custos de desenvolvimento.

### 1.2. Público-Alvo

- Engenharia
- Suporte
- Arquitetura

### 1.3. Âmbito

| Objetivo                               | Descrição                                                                                                                                                                                     |
| -------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Segurança por Design**               | Implementar múltiplas camadas de segurança: autenticação mútua, criptografia de dados em trânsito e em repouso, gerenciamento seguro de certificados, proteção contra ataques comuns (OWASP). |
| **Confiabilidade e Resiliência**       | Garantir funcionamento contínuo mesmo com conectividade intermitente, sincronização automática de dados quando a conexão é restaurada, recuperação automática de falhas.                      |
| **Desempenho e Eficiência Energética** | Otimizar o consumo de bateria, minimizar o uso de memória e processamento, garantir latência baixa nas operações críticas.                                                                    |
| **Escalabilidade**                     | Suportar um grande número de dispositivos (milhões) conectados simultaneamente, escalando horizontalmente a infraestrutura em nuvem.                                                          |
| **Observabilidade**                    | Fornecer visibilidade completa do estado dos dispositivos, com logs estruturados, métricas de performance, alertas proativos e rastreamento de eventos.                                       |
| **Manutenibilidade**                   | Permitir atualizações de software de forma segura e transparente (OTA), com rollback automático em caso de falha, facilitando a evolução contínua da plataforma.                              |
| **Modularidade**                       | Arquitetura baseada em componentes reutilizáveis, permitindo a composição de diferentes tipos de dispositivos com funcionalidades específicas.                                                |

---

## 2. Dependências e referências

> **Nota:** Os links abaixo devem ser atualizados com URLs específicos para cada dependência quando disponíveis na documentação interna.

- Android (AOSP / Android Embedded)
- MQTT Broker + Device Registry/Provisioning
- Backend APIs + IAM (OAuth2/OIDC)
- OTA service / artifact repository
- Observability stack (OTel/Prometheus/Grafana)
- Secrets manager / Key Vault

---

## 3. Regras de Utilização

Cada tipologia terá abordados aspetos específicos da tipologia, incluindo, sempre que aplicável, referências a diretrizes, padrões gerais e boas práticas.

Este documento não é um conjunto de regras inflexíveis, mas sim um guia de "fortes recomendações". Desvios são permitidos, mas devem ser justificados, documentados em uma ADR (Architecture Decision Record) e aprovados pelo Comitê de Arquitetura.

---

## 4. Arquitetura da Tipologia

A arquitetura é decomposta em três camadas principais: **Camada de Dispositivo** (Android Embedded), **Camada de Comunicação** (APIs e Mensageria), e **Camada de Nuvem** (Serviços Backend). Cada camada é responsável por funcionalidades específicas e se comunica através de protocolos bem definidos.

### 4.1. Diagrama de Contexto do Sistema

_[Diagrama de contexto disponível no repositório: docs/diagrams/context-diagram.png]_

### 4.2. Arquitetura no dispositivo

A arquitetura do dispositivo segue um modelo em camadas que separa as responsabilidades entre o firmware de baixo nível, a camada de abstração de hardware (HAL), os serviços de sistema e as aplicações.

| Componente                     | Descrição                                                                                                                                       |
| ------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| **Firmware Customizado**       | Sistema operacional Android Embedded otimizado para o hardware específico, com suporte a atualizações OTA seguras.                              |
| **Hardware Abstraction Layer** | Interface padronizada que abstrai as diferenças de hardware, permitindo portabilidade entre diferentes dispositivos.                            |
| **Serviços de Sistema**        | Serviços essenciais como gerenciamento de conectividade, sincronização de dados, autenticação local e gerenciamento de energia.                 |
| **Aplicações de Negócio**      | Aplicações específicas do domínio (ex: controle de eletrodoméstico, ponto de venda, monitoramento de loja) que implementam a lógica de negócio. |
| **Local Database**             | Banco de dados local (SQLite ou similar) para armazenamento de dados offline e sincronização posterior.                                         |
| **Message Queue Local**        | Fila local para armazenar mensagens quando offline, sincronizando quando a conectividade é restaurada.                                          |

### 4.2.1. Camadas

Cada aplicação no dispositivo segue o padrão **MVVM (Model-View-ViewModel)** para separação de responsabilidades e facilitar testes:

- **Model:** Camada de dados, incluindo acesso ao banco de dados local e APIs remotas.
- **View:** Interface do utilizador (Activities, Fragments).
- **ViewModel:** Lógica de apresentação, gerenciamento de estado e comunicação entre View e Model.

### 4.3. Comunicação

A comunicação entre dispositivos e serviços em nuvem é realizada através de múltiplos canais para garantir flexibilidade, confiabilidade e eficiência.

| Protocolo | Uso                                                                                                       | Características                                     |
| --------- | --------------------------------------------------------------------------------------------------------- | --------------------------------------------------- |
| **MQTT**  | Comunicação assíncrona, publish-subscribe, ideal para IoT com conectividade intermitente.                 | Leve, suporta QoS, reconexão automática.            |
| **HTTPS** | Chamadas síncronas para operações que requerem resposta imediata, como autenticação e consultas de dados. | Seguro, confiável, amplamente suportado.            |
| **gRPC**  | Comunicação de alta performance entre serviços internos (opcional, para casos de baixa latência).         | Bidirecional, multiplexing, serialização eficiente. |

### 4.4. Fluxo de Comunicação

_[Diagrama de fluxo de comunicação disponível no repositório: docs/diagrams/communication-flow.png]_

### 4.5. Offline-First

> Referência completa do padrão: https://ecom4isi.atlassian.net/wiki/spaces/DEVP/pages/5212340226

O dispositivo implementa um modelo Offline-First que permite operação contínua mesmo sem conectividade:

1. **Operações Locais:** Todas as operações são executadas localmente no banco de dados do dispositivo.
2. **Fila de Sincronização:** Mudanças são enfileiradas para sincronização posterior.
3. **Sincronização Automática:** Quando a conectividade é restaurada, as mudanças são sincronizadas com o servidor.
4. **Resolução de Conflitos:** A estratégia de resolução de conflitos deve ser definida por tipo de dado em ADR específico, seguindo as diretrizes abaixo:
   - **Dados de configuração** → Last-Write-Wins
   - **Dados transacionais** → Merge semântico ou intervenção manual
   - **Telemetria** → Append-only, sem conflitos

### 4.6. Testes

#### 4.6.1. Unitários

Para garantir a qualidade e a manutenibilidade do código no dispositivo Android Embedded, a estratégia de testes unitários é fundamental. Todos os novos desenvolvimentos de lógica de negócio e componentes de aplicação devem ser acompanhados de testes unitários correspondentes.

#### 4.6.2. Frameworks de Teste

**JUnit:** É o framework padrão para a escrita de testes unitários em Java e Kotlin. Os testes serão estruturados utilizando as anotações do JUnit (`@Test`, `@BeforeEach`, `@AfterEach`, etc.) para definir o ciclo de vida e a execução dos casos de teste.

**Mockito Core:** Para isolar os componentes sob teste, o Mockito será utilizado para criar mocks e stubs de dependências, como acesso a banco de dados, chamadas de rede ou interações com o sistema Android. Permite testar a lógica de uma unidade de forma isolada, sem depender de componentes externos.

#### 4.6.3. Estratégia de Testes para Dispositivos

Para cobrir riscos específicos de dispositivos embedded não detectáveis por testes unitários, a estratégia de testes deve incluir:

1. **Testes de integração** com emulador de MQTT broker (ex: Mosquitto em ambiente de CI).
2. **Testes de condições de rede** simulando latência, packet loss e desconexão abrupta, para validar o comportamento offline-first e a fila de sincronização.
3. **Testes de consumo de bateria e memória** em hardware real, executados antes de cada release com thresholds definidos por modelo de dispositivo.
4. **Testes de OTA** em ambiente controlado (canary interno) antes de qualquer rollout para produção.

---

## 5. Estrutura do Projeto

> **Nota:** A estrutura abaixo é um exemplo de referência — adaptar conforme o contexto de projeto específico.

```
sonae-iot-reference-platform/
│
├── device-android-embedded/
│   ├── app/
│   │   ├── src/
│   │   |   ├── main/
│   │   │   └── /java/com/sonae/device/
│   │   │       ├── data/
│   │   │       ├── domain/
│   │   │       ├── ui/
│   │   │       ├── App.kt
│   │   |       ├── /core/
│   │   │   │     ├── mqtt/
│   │   │   │     ├── security/
│   │   │   │     ├── sync/
│   │   │   └── res/
│   │   |   └── AndroidManifest.xml
|   |   |   └── test/
│   └── README.md
│
│
├── infra/
│   ├── k8s/
│   ├── terraform/
│   └── observability/
│
├── docs/
│   ├── architecture.md
│   ├── adr/
│   └── diagrams/
│
└── README.md
```

| **Diretório / Módulo**              | **Responsabilidade Primária**                                           |
| ----------------------------------- | ----------------------------------------------------------------------- |
| / (root)                            | Raiz do repositório, documentação geral e visão unificada da plataforma |
| docs/                               | Documentação arquitetural e de governança                               |
| docs/architecture.md                | Descrição da arquitetura de alto nível (C4, camadas, fluxos)            |
| docs/adr/                           | Architecture Decision Records (decisões arquiteturais e desvios)        |
| docs/diagrams/                      | Diagramas C4, sequência, contexto e containers                          |
| device-android-embedded/            | Código-fonte do dispositivo Android Embedded                            |
| device-android-embedded/app/        | Aplicação Android principal                                             |
| app/src/main/                       | Código principal do aplicativo                                          |
| app/src/main/java/com/sonae/device/ | Namespace base da aplicação                                             |
| core/                               | Componentes centrais e reutilizáveis do dispositivo                     |
| core/mqtt/                          | Comunicação MQTT (telemetria, comandos, eventos)                        |
| core/sync/                          | Sincronização offline-first e fila local                                |
| core/security/                      | Autenticação, tokens, keystore e pinning                                |
| core/ota/                           | Cliente OTA e verificação de updates                                    |
| data/                               | Acesso a dados (Room DB, APIs remotas, repositórios)                    |
| domain/                             | Regras de negócio e modelos de domínio                                  |
| ui/                                 | Camada de apresentação (Views / ViewModels – MVVM)                      |

---

## 6. Tecnologias Utilizadas

| **Categoria**       | **Tecnologia (com versão)** | **Observações**                            |
| ------------------- | --------------------------- | ------------------------------------------ |
| Sistema Operacional | Android Embedded (AOSP)     | Build customizado para hardware específico |
| Linguagem           | Kotlin (1.9+)               | Linguagem oficial Android                  |
| Arquitetura         | MVVM                        | Separação View / ViewModel / Model         |
| UI                  | Android Views / Jetpack     | UI nativa, baixo overhead                  |
| Mensageria          | MQTT (Eclipse Paho 1.2+)    | Comunicação assíncrona IoT                 |
| Transporte Seguro   | TLS 1.3                     | Criptografia de dados em trânsito          |
| Autenticação        | JWT                         | Tokens emitidos pelo backend               |
| Persistência Local  | SQLite / Room (2.6+)        | Base offline-first                         |
| Sincronização       | WorkManager (2.9+)          | Execução resiliente em background          |
| Fila Local          | Custom Queue                | Eventos offline para sincronização         |

## 7. Segurança

A segurança é implementada em múltiplas camadas, seguindo o princípio de **Defense in Depth**.

> Definições Globais: https://ecom4isi.atlassian.net/wiki/spaces/DEVP/pages/5217517761

### 7.1. Componentes de Segurança

| Componente                              | Descrição                                                                                         |
| --------------------------------------- | ------------------------------------------------------------------------------------------------- |
| **Secure Boot**                         | Verifica integridade do firmware antes da execução, impedindo execução de código não autorizado.  |
| **Trusted Execution Environment (TEE)** | Ambiente isolado para executar operações criptográficas sensíveis, protegido do SO principal.     |
| **Keystore Seguro**                     | Armazenamento seguro de chaves privadas e certificados, inacessível a aplicações não autorizadas. |
| **TLS 1.3**                             | Criptografia de dados em trânsito, com suporte a Perfect Forward Secrecy.                         |
| **Certificate Pinning**                 | Validação de certificados do servidor, prevenindo ataques Man-in-the-Middle.                      |
| **Autenticação Mútua**                  | Dispositivo e servidor se autenticam mutuamente usando certificados X.509.                        |
| **JWT Tokens**                          | Tokens de sessão com claims, permitindo autorização granular.                                     |
| **RBAC/ABAC**                           | Controle de acesso baseado em roles ou atributos, granular e flexível.                            |

### 7.2. Requisitos Mínimos de Segurança para Dispositivos IoT

Os seguintes requisitos são **obrigatórios** para todos os dispositivos. Desvios devem ser documentados em ADR e aprovados pelo Comitê de Arquitetura.

1. **Secure Boot** obrigatório em todos os dispositivos antes de saírem de fábrica.
2. **Certificados X.509** armazenados em Keystore de hardware (Secure Element ou TEE) — nunca em armazenamento de software.
3. **TLS 1.3** para todas as comunicações de rede, sem exceções.
4. **Certificate Pinning** para todas as APIs e brokers MQTT críticos.
5. **Rotação automática de certificados** antes da expiração (mínimo 30 dias de antecedência, via OTA).
6. **Detecção de root/tamper** ativa, com resposta definida: desconexão do broker, notificação ao Device Registry e bloqueio de operações sensíveis.

### 7.3. Fluxo de Provisionamento e Ciclo de Vida de Certificados

1. Device Registry gera par de chaves e certificado X.509 para o dispositivo.
2. Certificado injetado em fábrica via Secure Element ou fluxo seguro de fabricação.
3. Na primeira inicialização, o dispositivo autentica no Device Registry com o certificado de fábrica.
4. Rotação automática de certificados via OTA antes da expiração (mínimo 30 dias de antecedência).
5. Em caso de comprometimento, revogação imediata via Device Registry com bloqueio de todos os canais de comunicação do dispositivo.

---

## 8. Infraestrutura

Os serviços em nuvem são implantados em um cluster Kubernetes, gerenciados via IaC.

### 8.1. Topologia de Implantação

_[Diagrama de topologia disponível no repositório: docs/diagrams/deployment-topology.png]_

### 8.2. Atualização de Software (OTA)

As atualizações de software são críticas para manter a segurança e a funcionalidade dos dispositivos. A arquitetura OTA implementa um processo seguro e confiável, incluindo verificação de integridade, rollback automático e suporte à rotação de certificados.

### 8.3. Estratégia de Rollout

A estratégia de rollout segue um modelo de implantação gradual:

1. **Canary Release:** 5% dos dispositivos recebem a atualização primeiro.
2. **Monitoramento:** Métricas de erro e performance são coletadas.
3. **Rollout Progressivo:** Se tudo estiver bem, aumentar para 25%, depois 50%, depois 100%.
4. **Rollback Automático:** Se a taxa de erro ultrapassar um limite, fazer rollback automático.

> Para frotas com mais de 1.000 dispositivos, a segmentação de rollout por região, modelo de dispositivo e versão de firmware atual é obrigatória. Definir grupos de rollout em ADR específico.

### 8.4. Gerenciamento de Configuração e Segredos

- **Configurações:** Externalizadas em ConfigMaps do Kubernetes e injetadas nos Pods como variáveis de ambiente.
- **Segredos:** Credenciais de acesso (MQTT, banco de dados, APIs) serão gerenciadas por um Key Vault e montadas de forma segura nos Pods utilizando o driver CSI do Key Vault.

### 8.5. Ciclo de Vida de Dispositivos

O ciclo de vida completo do dispositivo deve ser gerenciado pelo Device Registry, cobrindo as seguintes fases:

- **Provisionamento:** conforme fluxo descrito na Secção 7.3.
- **Operação:** monitoramento contínuo via stack de observabilidade (Secção 10).
- **Decommissioning:** limpeza segura de dados sensíveis (chaves, PII, tokens) antes do descarte físico.
- **Transferência de Propriedade:** revogação de certificados do tenant anterior e novo provisionamento antes da ativação no novo tenant.

---

## 9. Padrões Arquiteturais Adotados

### 9.1. Event-Driven Architecture

Arquitetura orientada a eventos: componentes comunicam-se através de eventos assíncronos, reduzindo acoplamento e aumentando resiliência. Dispositivos publicam eventos de telemetria e estado via MQTT; o backend consome e reage de forma assíncrona.

> Referência: https://ecom4isi.atlassian.net/wiki/spaces/DEVP/pages/5212340226

### 9.2. CQRS (Command Query Responsibility Segregation)

Separação entre operações de escrita (Commands) e leitura (Queries). No contexto IoT: comandos enviados aos dispositivos via MQTT (ex: atualizar configuração), consultas de estado via API REST.

> Referência: https://ecom4isi.atlassian.net/wiki/spaces/DEVP/pages/5212340226

### 9.3. MVVM (Mobile)

Padrão Model-View-ViewModel para aplicações Android. Separa a lógica de negócio (Model), a lógica de apresentação (ViewModel) e a interface do utilizador (View), facilitando testes unitários e manutenibilidade.

> Referência: https://ecom4isi.atlassian.net/wiki/spaces/DEVP/pages/5212340226

### 9.4. Offline-First

Todas as operações são executadas localmente primeiro; sincronização com o servidor é realizada quando a conectividade está disponível. Estratégias de resolução de conflitos por tipo de dado devem ser definidas em ADR (ver Secção 4.5).

> Referência: https://ecom4isi.atlassian.net/wiki/spaces/DEVP/pages/5212340226

---

## 10. Observabilidade

> Referência ao padrão global de Observabilidade: https://ecom4isi.atlassian.net/wiki/spaces/DEVP/pages/5217517761

A observabilidade é implementada através da stack OpenTelemetry, Prometheus e Grafana, fornecendo visibilidade completa do sistema.

### 10.1. Métricas Chave

| Métrica                         | Descrição                                                                                              |
| ------------------------------- | ------------------------------------------------------------------------------------------------------ |
| **Device Online/Offline Ratio** | Percentual de dispositivos online vs offline, indicador de saúde geral da plataforma.                  |
| **Message Latency**             | Latência de entrega de mensagens MQTT, importante para operações críticas.                             |
| **OTA Success Rate**            | Percentual de atualizações bem-sucedidas, indicador de qualidade do processo OTA.                      |
| **API Response Time**           | Latência das APIs REST, importante para experiência do utilizador.                                     |
| **Data Sync Lag**               | Diferença de tempo entre evento no dispositivo e persistência em nuvem, importante para Offline-First. |
| **Error Rate by Service**       | Taxa de erros por serviço, para identificar problemas rapidamente.                                     |
| **Memory/CPU Usage**            | Utilização de recursos nos dispositivos e serviços, para otimização e planejamento de capacidade.      |

### 10.2. Métricas de Bateria e Eficiência Energética

As seguintes métricas devem ser monitoradas para todos os dispositivos com bateria:

| Métrica                         | Descrição                                                                                         | Threshold de Alerta          |
| ------------------------------- | ------------------------------------------------------------------------------------------------- | ---------------------------- |
| **Battery Level**               | Nível de bateria em percentual.                                                                   | Alerta abaixo de 20%         |
| **Battery Drain Rate**          | Taxa de consumo (% por hora).                                                                     | Alerta se > 2x do baseline   |
| **Anomalous Power Consumption** | Desvio em relação ao consumo esperado — pode indicar falha de hardware ou bug de software.        | Alerta se > 2x do baseline   |
| **Wake Lock Duration**          | Tempo total de wake locks ativos por sessão — excessos indicam possível bug de consumo energético.| A definir por modelo         |

> Thresholds de baseline devem ser definidos por modelo de dispositivo e documentados em ADR.

### 10.3. Privacidade de Dados em Dispositivo (GDPR)

Dados Pessoais Identificáveis (PII) coletados por sensores ou aplicações no dispositivo devem seguir as seguintes diretrizes:

1. PII não deve ser armazenada em texto claro — usar criptografia a nível de campo no banco de dados local.
2. Dados de PII devem ter tempo de retenção local definido e ser purgados automaticamente após expiração.
3. No decommissioning do dispositivo, todos os dados locais devem ser apagados de forma segura (secure wipe).
4. Compliance com GDPR deve ser avaliado por tipo de dado coletado e documentado em ADR específico.

---

## 11. Plano de Contingência para Indisponibilidade de Cloud

O comportamento do dispositivo em caso de indisponibilidade prolongada dos serviços cloud deve ser definido, configurado e testado para cada tipo de dispositivo:

| Cenário                          | Comportamento Esperado                                                                                                        |
| -------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| **Indisponibilidade < 1h**       | Operação normal offline-first; fila de sincronização retém eventos; reconexão automática ao restabelecer conectividade.       |
| **Indisponibilidade 1h–24h**     | Cache local extendido ativo; funcionalidades não críticas desativadas conforme configuração; alertas locais no dispositivo.   |
| **Indisponibilidade > 24h**      | Apenas operações essenciais de negócio permitidas (modo degradado); logs compactados localmente; notificação ao operador via canal alternativo (SMS/e-mail), se configurado. |
| **Retorno de conectividade**     | Sincronização incremental priorizada; resolução de conflitos conforme regras da Secção 4.5; auditoria de eventos pendentes.   |

> Thresholds e comportamentos específicos por contexto de negócio devem ser documentados em ADR e configuráveis por tipo de dispositivo.

---

> **Aplicação de Exemplo:** [sample-swrefarch-android](https://github.com/mcdigital-devplatforms/sample-swrefarch-android)
