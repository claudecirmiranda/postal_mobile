# DAS Mobile Apps – Uso Interno (B2E)

> **Detalhe:** React Native preferencialmente para iOS e Android. Hybrid MAUI como opção secundária.

---

## Histórico de Revisões

| Versão | Data       | Autor(es)       | Resumo das Mudanças                                        |
|--------|------------|-----------------|------------------------------------------------------------|
| 1.0    | 28/11/2025 | William Alves   | Criação inicial do documento de arquitetura de referência. |
| 2.0    | 10/12/2025 | William Alves   | Ajustes conforme comentários deixados no Confluence.       |
| 3.0    | 16/12/2025 | William Alves   | Ajustes de formatação para uniformizar todos os DAS.       |
| 4.0    | 17/12/2025 | William Alves   | Ajustes pós-apresentação.                                  |
| 4.1    | 26/12/2025 | William Alves   | Ajustes conforme novos comentários.                        |
| 4.2    | 05/01/2026 | William Alves   | Ajustes conforme novos comentários.                        |
| 4.3    | 06/01/2026 | William Alves   | Ajustes conforme novos comentários.                        |
| 4.4    | 05/06/2026 | Felipe Klussmann | Pequenos ajustes.                                         |

---

## Índice

1. [Introdução](#1-introdução)
2. [Dependências e Referências](#2-dependências-e-referências)
3. [Regras de Utilização](#3-regras-de-utilização)
4. [Arquitetura da Tipologia](#4-arquitetura-da-tipologia)
5. [Estrutura do Projeto](#5-estrutura-do-projeto)
6. [Tecnologias Utilizadas](#6-tecnologias-utilizadas)
7. [Segurança](#7-segurança)
8. [Infraestrutura](#8-infraestrutura)
9. [Padrões e Princípios Arquiteturais](#9-padrões-e-princípios-arquiteturais)
10. [Observabilidade](#10-observabilidade)

---

## 1. Introdução

### 1.1. Propósito e Objetivos

Este documento descreve a arquitetura de referência para o desenvolvimento, deployment e gestão de aplicações mobile performáticas, escaláveis e consistentes usando React Native como primeira opção e Hybrid MAUI como opção secundária.

Esta arquitetura é o padrão a ser seguido para todos os novos frontends mobile para aplicações com acesso exclusivamente interno à Organização. Abrange desde a estrutura do código-fonte até o deployment e monitorização em ambiente produtivo.

### 1.2. Público-Alvo

- Engenharia
- Suporte
- Arquitetura

### 1.3. Âmbito

Os seguintes objetivos visam endereçar os problemas relatados na etapa de Assessment.

| **Objetivo Arquitetural**                          | **Problema Endereçado**                                                                                                                                                                                    | **Solução**                                                                                                                      |
|----------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------|
| Agregação de serviços                              | Remoção de lógica de negócio e API stitching do cliente, e redução de complexidade.                                                                                                                        | Quando for necessária lógica que envolva múltiplos domínios, adotar o padrão Backend for Frontend (BFF).                        |
| Alta Performance e Estabilidade da aplicação       | Intermitência, instabilidade do sistema e dificuldade em realizar testes de performance.                                                                                                                   | MVVM, com processamento pesado executado fora da thread principal de UI.                                                         |
| Adaptabilidade de Serviços aos Dispositivos Móveis | O *Backend* de API de propósito geral tende a falhar, pois a experiência móvel frequentemente exige chamadas diferentes, em menor número, e a exibição de menos dados do que as aplicações para *desktop*. | Backend For Frontend (BFF).                                                                                                      |
| Resiliência da aplicação                           | Falta de suporte a operação *Offline*. A instabilidade da rede em áreas com cobertura limitada exige que as aplicações móveis sejam projetadas para operar *offline*.                                      | Princípio Offline First.                                                                                                         |
| Redução do volume de dados trafegados              | A gestão de grandes volumes de dados (ex: 6.000 artigos por loja) traz desafios para uso em dispositivos móveis (PDAs¹). Há preocupações sobre se a quantidade de dados (10 ou 50 GB) cabe no dispositivo. | Endpoints GraphQL (sugerido para casos com maturidade na equipa); remoção de dados analíticos do dispositivo; sincronização com filtros de afinidade à loja; avaliação de filtro de gama ativa. |
| Code Signing Automatizado                          | O processo de code signing e a publicação do SDK são complexos devido a problemas de infraestrutura e falta de mecanismos automatizados.                                                                   | Para React Native: Plataforma Expo.dev (biblioteca `expo-updates`).                                                              |

> ¹ **PDA:** Personal Digital Assistant — dispositivo móvel de campo utilizado em operações de retalho e logística.

- **Agregação de Serviços:** Uso do padrão Backend For Frontend (BFF) para desacoplar o *frontend* das APIs de negócio, reduzir a complexidade das integrações (autenticação), e evitar que o *frontend* tenha de implementar a lógica de agregação (*stitching*).
- **Alta Performance e Estabilidade da aplicação:** A arquitetura é proposta com a aplicação correta do MVVM e a remoção do processamento na *thread* da UI, um fator chave para mitigar o elevado número de ANR's (*Application Not Responding*).
- **Adaptabilidade de Serviços aos Dispositivos Móveis:** Com a aplicação do padrão BFF, os endpoints são adaptados às necessidades dos dispositivos móveis, como: agregação dos resultados das chamadas a outras APIs, redução das chamadas ao backend realizadas pela aplicação; além disso, itens como paginação e classificação (*sorting*) dos dados podem ser realizados no BFF.
- **Resiliência da aplicação:** A aplicação deve ser concebida para funcionar *offline* por defeito, sendo a conectividade *online* uma consideração secundária. O objetivo é garantir que os utilizadores tenham acesso a funcionalidades críticas (como contagens e inventários) mesmo quando a rede está instável ou inexistente. É necessário um mecanismo de sincronização regular com o backend.
- **Redução do volume de dados trafegados:** Uso de GraphQL para tornar os payloads mais eficientes, com apenas os dados necessários para atender a requisição. Dessa forma, haverá menos chamadas à API do BFF e, consequentemente, menos dados trafegados pela rede, contribuindo para maior performance. Além disso, deve-se garantir que os sistemas transacionais não sejam a fonte analítica. Os dados de grande volume para análises devem ser segregados e direcionados para um *back office* analítico ou *Data Lake*, removendo o ónus do dispositivo.

> **Atenção:** Quando se trata de gama de produtos por loja, é imperativo que haja filtros dos dados por afinidade à loja e gama ativa. A aplicação deste filtro reduz o volume de dados a ser trabalhado num dispositivo móvel de campo, por exemplo.

- **Code Signing Automatizado:** A biblioteca `expo-updates` suporta assinatura de código *end-to-end* usando criptografia de chave pública. A assinatura de código permite aos programadores assinar criptograficamente as suas atualizações com as suas próprias chaves. As assinaturas são verificadas no cliente antes da atualização ser aplicada, o que garante que ISPs, CDNs, fornecedores de cloud e até o próprio EAS não podem interferir com as atualizações executadas pelas aplicações.

---

## 2. Dependências e Referências

- **Expo Application Services (EAS):** Conjunto de serviços hospedados para projetos React Native:
  - Automatizar CI/CD para construir, submeter e atualizar a aplicação.
  - O EAS resolve problemas que requerem recursos físicos, como servidores de aplicações e CDNs para fornecer atualizações *over-the-air* e servidores físicos para execução de builds.

- **Framework React Native (configuração mínima):**
  - Biblioteca para UI: React Native Core.
  - Bibliotecas para navegação entre ecrãs: `@react-navigation/native`, `@react-navigation/stack`.
  - Uso principal: construção de UI e navegação entre ecrãs.

- **BFF Backend:**
  - Projeto na pasta `Bff/`.
  - Uso principal: intermediar a comunicação entre o frontend e o backend de forma a otimizá-la e simplificá-la.

- **Linguagem GraphQL:**
  - Biblioteca cliente: `@apollo/client graphql`.
  - Biblioteca servidor (.NET 8): `HotChocolate.AspNetCore`.
  - Uso principal: implementar GraphQL na comunicação entre frontend e backend quando fizer sentido (ecrãs com composição pesada, alta variação por contexto, múltiplas versões/clients, otimização de banda/latência).

- **Offline-First:**
  - Biblioteca para armazenamento local (dados de domínio): `react-native-sqlite-storage`.
  - Biblioteca para armazenamento auxiliar (cache, flags, dados leves): `@react-native-async-storage/async-storage`.
  - Biblioteca para monitorar o estado de rede: `@react-native-community/netinfo`.
  - Biblioteca para gerir estado global (preferências, flags, cache leve, sessão): `redux` + `@reduxjs/toolkit` + `redux-persist`.
  - **Funcionamento:**
    1. Os dados são obtidos do servidor (via GraphQL), mas ao serem recebidos, são guardados no armazenamento local do dispositivo (cache / base de dados).
    2. A aplicação lê dados preferencialmente do armazenamento local — não depende de rede para mostrar conteúdo já existente.
    3. Quando o utilizador cria/edita dados (ex: cria um novo registo), a operação é enfileirada localmente (flag "pendente de sync"), e refletida na UI imediatamente.
    4. A aplicação monitoriza o estado da rede; quando deteta reconexão, envia para o servidor todas as operações pendentes — e ao confirmar sucesso, marca como "sincronizado".
    5. Em caso de dados atualizados no servidor, ao sincronizar, o armazenamento local pode ser atualizado (pull + merge) para refletir a versão mais recente.

---

## 3. Regras de Utilização

Cada tipologia terá abordados aspetos específicos da tipologia, incluindo, sempre que aplicável, referências a diretrizes, padrões gerais e boas práticas.

Este documento não é um conjunto de regras inflexíveis, mas sim um guia de "fortes recomendações". Desvios são permitidos, mas devem ser justificados, documentados em uma ADR (Architecture Decision Record) e aprovados pelo Comité de Arquitetura.

---

## 4. Arquitetura da Tipologia

> **Diagrama:** Consultar o diagrama de arquitetura no Confluence (`internal_external mobile.jpg`).

### 4.1. Camadas

#### UI Components e Screens

- Camada visível para o utilizador: tudo que aparece no ecrã — botões, formulários, listas, textos, navegação entre ecrãs, estilo visual, etc.
- Responsabilidades:
  - Renderizar a interface.
  - Capturar interações do utilizador.
  - Mostrar estados de carga, erro e feedback.
  - Integrar com a lógica de negócio — usando hooks, contexto, estado global ou store.

#### Armazenamento Local (Local Database)

- Armazenamento persistente no dispositivo.
- Responsabilidades:
  - Permitir que a aplicação funcione *offline*: o utilizador pode ver dados previamente sincronizados, criar novos dados e editar dados, sem dependência de conexão.
  - Agir como fonte de verdade no cliente — a UI e a lógica consultam diretamente o armazenamento local, não a rede, garantindo consistência e velocidade.
  - Persistir informações entre sessões, reinícios da aplicação ou do dispositivo — garantindo que dados não se perdem e que o utilizador sempre tem acesso ao que já transferiu ou criou.

#### Queue de Operações Pendentes (sync pendente)

- Uma queue de operações realizadas pelo utilizador enquanto a aplicação está *offline* ou sem conexão — por exemplo, criação de ordens, edições, deleções, etc. Cada operação é registada localmente com marcação de "pendente de sincronização". Esta queue pode fazer parte do armazenamento local ou estar em estrutura separada (ex: tabela de *pending changes*). Essencialmente, armazena "intenção de modificação" para executar futuramente quando houver rede.
- Responsabilidades:
  - Garantir que ações do utilizador não se percam — mesmo que ele esteja *offline* ou encerre a aplicação — a intenção de mudança é persistida e será sincronizada depois.
  - Permitir *optimistic UI*: quando o utilizador cria ou edita algo, a aplicação já reflete na interface local, sem precisar aguardar resposta do servidor.
  - Servir de buffer de sincronização: quando a rede voltar, a aplicação envia todas as operações pendentes de forma organizada para o servidor.
- **Regras de Sincronização:**
  - Garantir idempotência (`operationId`).
  - Ordenação por entidade (*ordering*).
  - Estratégia de conflito (*conflict strategy*): ver Secção 9.4 para critérios de seleção.
  - Backoff/retry e limites de armazenamento.
  - Definição clara de "fonte de verdade" no cliente (quando e como diverge do servidor).

#### Lógica de Sync + Detecção de Rede (Sync Engine / Network Manager)

- Módulo responsável por monitorizar o estado de conectividade de rede (online/offline), detetar quando a rede ficar disponível e disparar a sincronização das pendências.
- Também responsável por executar o processo de sincronização: enviar operações pendentes ao servidor, tratar respostas, resolver conflitos, atualizar o armazenamento local, remover pendências marcadas, e possivelmente fazer *pull* de dados novos do servidor.
- Responsabilidades:
  - Orquestrar a comunicação entre local e remoto: ao detetar que a rede está disponível, processar a queue de pendências, sincronizar dados e garantir que o armazenamento local e o servidor convirjam.
  - Garantir que a experiência do utilizador seja transparente: ele pode usar a aplicação *offline* e a sincronização acontece em background ao reconectar.
  - Lidar com casos típicos de conectividade móvel: reconexão intermitente, redes instáveis, reconciliação de dados, conflitos, retry, etc.

#### GraphQL Client

- Cliente que a aplicação usa para fazer requisições ao backend (quando *online*) — por exemplo, usando GraphQL (via Apollo Client) para consultar dados, enviar mutations e sincronizar dados com o servidor.
- Responsabilidades:
  - Permitir comunicação com o servidor backend: buscar dados globais, enviar novos dados criados localmente, receber dados de outros utilizadores, obter atualizações, etc.
  - Quando combinado com a lógica de sync, serve para "materializar" as operações pendentes: as ações registadas na queue acabam tornando-se requisições GraphQL ao servidor.

### 4.2. Melhores Práticas em React Native

Consultar: [https://ecom4isi.atlassian.net/wiki/spaces/DEVP/pages/5227315219](https://ecom4isi.atlassian.net/wiki/spaces/DEVP/pages/5227315219)

---

## 5. Estrutura do Projeto

Para o cenário onde existe mais de uma equipa a trabalhar no mesmo domínio/módulo, sugere-se a seguinte arquitetura.

> **Diagrama:** Consultar o diagrama de estrutura no Confluence (`Mermaid Chart - 2026-02-05.png`).

Princípios:

- **Domain Ownership explícito por subdomínio.** Cada equipa é dona de uma fatia funcional, não do domínio inteiro.
- **Client-Side Composition (obrigatório).** UI final é composta no runtime.
- **Isolamento por Boundary Técnica.** Sem imports cruzados entre equipas.
- **Contratos estáveis públicos:** Tipos, Schemas, Eventos, APIs.

**public-api/routes.ts**

```ts
import type { RouteObject } from 'react-router';

export const ordersBillingRoutes: RouteObject[] = [
  {
    path: '/orders/:id/billing',
    element: <BillingScreen />,
  },
];
```

**public-api/events.ts**

```ts
export const OrdersEvents = {
  ORDER_CREATED: 'orders.created',
  PAYMENT_CONFIRMED: 'orders.payment.confirmed',
} as const;
```

Comunicação via eventos, não imports diretos.

- **Deploy independente.** Cada equipa pode versionar, buildar e publicar sem bloquear outras.

```
/src
├── app-shell/                          # Equipa Plataforma
│   ├── App.tsx
│   ├── bootstrap.tsx
│   ├── router/
│   ├── providers/
│   ├── composition/
│   ├── observability/
│   ├── auth/
│   └── __tests__/
│       ├── AppShell.test.tsx
│       ├── routing.test.tsx
│       └── composition.test.ts
│
├── domains/
│   └── orders/
│
│       ├── orders-core/                # Equipa A
│       │   ├── public-api/
│       │   │   ├── routes.ts
│       │   │   ├── events.ts
│       │   │   ├── permissions.ts
│       │   │   └── __tests__/
│       │   │       ├── routes.contract.test.ts
│       │   │       └── events.contract.test.ts
│       │   │
│       │   ├── screens/
│       │   ├── hooks/
│       │   ├── state/
│       │   ├── __tests__/
│       │   │   ├── OrdersListScreen.test.tsx
│       │   │   └── useOrdersViewModel.test.ts
│       │   └── index.ts
│       │
│       ├── orders-billing/             # Equipa B
│       │   ├── public-api/
│       │   ├── screens/
│       │   ├── hooks/
│       │   ├── __tests__/
│       │   │   ├── BillingScreen.test.tsx
│       │   │   └── useBillingViewModel.test.ts
│       │   └── index.ts
│       │
│       └── orders-fulfillment/         # Equipa C
│           ├── public-api/
│           ├── screens/
│           ├── hooks/
│           ├── __tests__/
│           └── index.ts
│
├── shared-contracts/                   # Governança (testável)
│   ├── orders/
│   │   ├── schemas/
│   │   │   ├── order.schema.ts
│   │   │   └── __tests__/
│   │   │       └── order.schema.test.ts
│   │   ├── events/
│   │   │   └── __tests__/
│   │   └── permissions.ts
│   └── auth/
│
├── shared-ui/
│   ├── atoms/
│   ├── molecules/
│   └── __tests__/
│       └── Button.test.tsx
│
├── infra/
│   ├── observability/
│   │   └── __tests__/
│   │       └── otel.test.ts
│   ├── storage/
│   └── i18n/
│
├── test-utils/                         # Helpers globais
│   ├── renderWithProviders.tsx
│   ├── mockNavigation.ts
│   └── setup.ts
│
└── jest.config.js
```

---

## 6. Tecnologias Utilizadas

### Stack principal

| **Categoria**               | **Tecnologia / Biblioteca**                                  | **Observações**                                      |
|-----------------------------|--------------------------------------------------------------|------------------------------------------------------|
| Linguagem                   | JavaScript (padrão ES* última versão)                        | Linguagem principal da aplicação mobile.             |
| Linguagem                   | C# (.NET LTS)                                                | Linguagem usada no backend (BFF + APIs).             |
| Mobile Framework            | React Native                                                 | Framework principal para desenvolvimento mobile nativo. |
| Gestão de Pacotes           | npm                                                          | Gestão de dependências JavaScript.                   |
| UI / Componentes            | React Native Core Components                                 | `<View>`, `<Text>`, `<Image>`, etc.                 |
| Navegação                   | React Navigation                                             | Navegação entre ecrãs (stack/tabs).                  |
|                             | `@react-navigation/native`                                   | Core da navegação.                                   |
|                             | `@react-navigation/stack`                                    | Navegação baseada em stack.                          |
|                             | `react-native-screens`                                       | Otimizações de navegação nativa.                     |
|                             | `react-native-safe-area-context`                             | Gestão de áreas seguras.                             |
| Comunicação com Backend     | Apollo Client (`@apollo/client`)                             | Cliente GraphQL usado para consultar o backend.      |
| Armazenamento local / Offline-First | SQLite (`react-native-sqlite-storage`)               | Armazenamento local nativo.                          |
|                             | Camada JS do SQLite (`productDb.js`, `cartDb.js`)            | Acesso via JS ao armazenamento SQLite real.          |
|                             | Async Storage (`@react-native-async-storage/async-storage`)  | Armazenamento leve de chaves/valores.                |
| Sync / Rede                 | `@react-native-community/netinfo`                            | Deteta conectividade online/offline.                 |
|                             | Serviço de Sync Customizado                                  | Envia pendências para o servidor quando online.      |
| Estado / Store              | Redux Toolkit                                                | Estado global da aplicação (cart, products, orders). |
|                             | `react-redux`                                                | Integração React + Redux.                            |
|                             | `redux-persist`                                              | Persistência do store no dispositivo.                |
| Utilidades                  | `debounce.js`                                                | Funções auxiliares de debounce.                      |
|                             | `formatPrice.js`                                             | Formatação de preços (e-commerce).                   |
|                             | `logger.js`                                                  | Log interno de debug.                                |
| Constantes / Configurações  | `routes.js`                                                  | Nomes de rotas centralizados.                        |
|                             | `apiConfig.js`                                               | URLs e configurações do backend.                     |
|                             | `storageKeys.js`                                             | Chaves de armazenamento local.                       |
| Backend / BFF (.NET)        | ASP.NET Core                                                 | Framework usado no backend.                          |

### Stack sem Expo (ferramentas de desenvolvimento)

| **Categoria**    | **Tecnologia / Biblioteca** | **Observações**                        |
|------------------|-----------------------------|----------------------------------------|
| Ferramentas / Dev | VSCode                     | IDE principal utilizado.               |
|                  | Xcode                       | Build iOS.                             |
|                  | Android Studio              | Build Android.                         |
|                  | CocoaPods                   | Gerenciador de dependências iOS.       |
| Build Mobile     | Gradle (Android)            | Sistema de build nativo.               |
|                  | Xcode Build (iOS)           | Build para iOS.                        |

### Stack com Expo (recomendado)

Em caso de utilizar uma plataforma de publicação automática em Loja de Apps como o Expo (recomendado), têm-se as seguintes dependências adicionais:

| **Categoria** | **Tecnologia / Biblioteca** | **Descrição / Função**                          |
|---------------|-----------------------------|-------------------------------------------------|
| Build / Infra | EAS Build                   | Compilação para Android/iOS sem ambiente local. |
| OTA Updates   | `expo-updates`              | Atualizações *over-the-air*.                    |
|               | `expo-application`          | Info da aplicação.                              |
|               | `expo-file-system`          | Acesso ao sistema de ficheiros do dispositivo.  |

Com o Expo, não há necessidade de ferramentas para realizar o build manual para iOS/Android.

---

## 7. Segurança

Definições Globais: [https://ecom4isi.atlassian.net/wiki/spaces/DEVP/pages/5217517761](https://ecom4isi.atlassian.net/wiki/spaces/DEVP/pages/5217517761)

### 7.1. Requisitos Mínimos de Segurança para Mobile

Independentemente das definições globais, toda a aplicação mobile desenvolvida sob esta arquitetura deve implementar obrigatoriamente os seguintes controlos:

1. **Armazenamento Seguro de Credenciais:** Tokens de autenticação e refresh tokens devem ser armazenados exclusivamente em *secure storage* nativa (`expo-secure-store` ou `react-native-keychain` / `react-native-encrypted-storage`). É proibido o uso de `AsyncStorage` para dados sensíveis.
2. **Certificate Pinning (SSL Pinning):** Todas as comunicações com o backend devem implementar *certificate pinning* para prevenir ataques Man-in-the-Middle (MITM). Utilizar `react-native-ssl-public-key-pinning` (Expo) ou configuração via Network Security Configuration / TrustKit (CLI).
3. **Ofuscação de Código:** Builds de produção devem habilitar minificação e ofuscação de código para dificultar engenharia reversa de lógica sensível.
4. **Deteção de Root/Jailbreak:** Aplicações que acedam a dados corporativos sensíveis devem implementar deteção de ambientes comprometidos e restringir o acesso nesses cenários. Utilizar `expo-device` (Expo) ou `jail-monkey` / `react-native-jailbreak-detector` (CLI).
5. **Prevenção de Logging de PII:** É proibido registar dados de identificação pessoal (nomes, credenciais, tokens, payloads completos) em logs. Todos os logs devem ser sanitizados antes de qualquer envio a sistemas de monitorização.
6. **Proteção de Dados em Repouso:** Armazenamento local SQLite que contenha dados corporativos deve ser encriptado. Utilizar `react-native-sqlite-storage` com suporte a encriptação ou equivalente.

---

## 8. Infraestrutura

Recomenda-se o serviço de Hosting da plataforma Expo para CI/CD com code signing e publicação em loja de aplicações automatizadas.

### 8.1. Infraestrutura da Aplicação Mobile (Expo)

**Desenvolvimento:**
- Expo CLI.
- Expo Go em dispositivos físicos.
- Conexão local (LAN ou tunnel).

### 8.2. Build

A infraestrutura de build é 100% gerenciada:

**EAS Build (Expo Application Services):**
- Builds remotos para Android/iOS.
- Sem necessidade de Xcode ou Android Studio local.
- Armazenamento seguro de secrets.
- Workers distribuídos.

**Publicação:**
- Google Play Console.
- Apple App Store Connect.
- Expo EAS Submit automatiza a submissão.

**OTA Updates:**
- Com `expo-updates`, a aplicação pode receber atualizações *over-the-air* sem reenviar para a loja.

> **Política de OTA e Versionamento:** Atualizações OTA aplicam-se exclusivamente a alterações em JS/Assets — alterações em código nativo requerem submissão nova às lojas. Deve ser definida uma política de versionamento mínimo obrigatório: versões abaixo do mínimo devem receber notificação de *forced upgrade* via feature flag ou resposta do BFF. Atualizações OTA em produção devem ser validadas previamente em canal de staging.

Requer:
- Conta Expo.
- Setup do projeto com `eas.json`.
- Canal de release (production/staging/dev).

### 8.3. Gestão de Erros

Uma estratégia unificada de tratamento de erros deve ser implementada em toda a aplicação, cobrindo:

- **Retry policies:** Definir número máximo de tentativas e intervalo de backoff exponencial para chamadas de rede e operações de sincronização.
- **Fallback UI:** Em caso de falha de carregamento de dados (rede indisponível e cache vazio), apresentar estados de erro claros com opções de retry visíveis ao utilizador.
- **Notificação ao utilizador:** Erros de sincronização prolongados ou falhas críticas devem ser comunicados de forma não intrusiva (ex: banner ou toast), sem bloquear a interface.
- **Logging estruturado:** Todos os erros devem ser capturados e enviados à plataforma de observabilidade com contexto suficiente (`traceId`, tipo de erro, ecrã de origem), sem incluir dados sensíveis.

---

## 9. Padrões e Princípios Arquiteturais

### 9.1. GraphQL

Consultar: [https://ecom4isi.atlassian.net/wiki/spaces/DEVP/pages/edit-v2/5212340226#GraphQL](https://ecom4isi.atlassian.net/wiki/spaces/DEVP/pages/edit-v2/5212340226#GraphQL)

#### 9.1.1. Integração com a Aplicação Mobile (Apollo Client)

A comunicação entre o frontend (React Native / Expo) e o backend GraphQL ocorre via:

```ts
const client = new ApolloClient({
  uri: "https://api.exemplo.com/graphql",
  cache: new InMemoryCache(),
  headers: {
    Authorization: `Bearer ${token}`,
  },
});
```

Benefícios:
- Economia de banda no mobile.
- Queries específicas por contexto.
- Caching automático.
- Mutations com suporte a re-tentativas e replay via mecanismo de sync.
- Integração com sincronização local.

#### 9.1.2. Critérios de Seleção — GraphQL vs REST

A adoção de GraphQL deve ser avaliada com base nos seguintes critérios objetivos. Em caso de dúvida, registar a decisão numa ADR.

| **Critério**                         | **Favorece GraphQL**                                   | **Favorece REST**                                  |
|--------------------------------------|--------------------------------------------------------|----------------------------------------------------|
| Composição de dados                  | Ecrãs com dados de múltiplas fontes                    | Endpoints com payload simples e fixo               |
| Variabilidade de payload             | Alta variação de campos por contexto/cliente           | Payload uniforme entre clientes                    |
| Volume de round-trips                | Necessidade de reduzir chamadas ao backend             | Número de chamadas já reduzido                     |
| Real-time                            | Subscriptions necessárias                              | Polling suficiente                                 |
| Maturidade da equipa com GraphQL     | Equipa com experiência prévia                          | Equipa sem experiência prévia em GraphQL           |
| Infraestrutura de BFF                | BFF GraphQL já disponível                              | Apenas endpoints REST disponíveis                  |

### 9.2. MVVM

Consultar: [https://ecom4isi.atlassian.net/wiki/spaces/DEVP/pages/edit-v2/5212340226#MVVM](https://ecom4isi.atlassian.net/wiki/spaces/DEVP/pages/edit-v2/5212340226#MVVM)

### 9.3. Offline-First

Consultar: [https://ecom4isi.atlassian.net/wiki/spaces/DEVP/pages/edit-v2/5212340226#Princípio-Offline-First](https://ecom4isi.atlassian.net/wiki/spaces/DEVP/pages/edit-v2/5212340226#Princ%C3%ADpio-Offline-First)

### 9.4. Estratégia de Resolução de Conflitos

A escolha da estratégia de conflito deve ser definida por entidade ou cenário de negócio, e não de forma global. A tabela abaixo serve como ponto de partida — decisões específicas devem ser registadas em ADR ou num documento de governança de dados.

| **Tipo de Dado / Entidade**       | **Estratégia Recomendada** | **Justificação**                                                                 |
|-----------------------------------|----------------------------|----------------------------------------------------------------------------------|
| Inventário / Contagens            | *Server-wins*              | O servidor é a fonte de verdade para dados de stock.                             |
| Rascunhos de formulário / Notas   | *Last-Write-Wins (LWW)*    | O utilizador tem controlo total; conflitos são improváveis e de baixo impacto.   |
| Dados mestres (produtos, preços)  | *Server-wins*              | Dados de referência devem sempre refletir o estado do sistema central.           |
| Ordens / Transações               | *Manual review*            | Alto impacto financeiro; conflitos devem ser sinalizados e resolvidos manualmente. |
| Configurações de utilizador       | *LWW*                      | Preferências locais têm prioridade sobre o estado remoto.                        |

> **Nota:** Em cenários de *offline* prolongado (> 24h), considerar invalidar dados de inventário e forçar resincronização completa antes de permitir novas operações críticas.

### 9.5. Gestão do Armazenamento Local e Cache

Para evitar esgotamento de espaço em dispositivos com recursos limitados e garantir performance em queries sobre grandes volumes de dados:

- **Estratégia de limpeza (TTL por entidade):** Definir tempo de vida máximo para cada tipo de dado local. Exemplo orientativo: dados de catálogo de produtos → 24h; configurações de sessão → até logout; dados analíticos → não persistir localmente.
- **Limites de armazenamento:** Definir limites máximos de uso de armazenamento por utilizador/loja (ex: máximo de X MB para a base SQLite). Implementar alertas ou limpeza automática ao atingir o limiar.
- **Requisitos de indexação:** Garantir que queries frequentes sobre a base SQLite local (ex: pesquisa de produtos por código ou nome) têm índices definidos explicitamente, evitando *full table scans* em tabelas de grande volume.
- **Dados analíticos:** Não persistir dados de grande volume para análise no dispositivo. Estes devem ser removidos da sincronização e direcionados para o *back office* analítico ou *Data Lake*.

### 9.6. Testabilidade da Estratégia Offline

A arquitetura *offline-first* deve ser validada por testes automatizados que cubram os seguintes cenários:

- **Simulação de estados de rede:** Utilizar *mocking* de `@react-native-community/netinfo` para simular transições online → offline → online em testes de integração.
- **Replay de operações pendentes:** Validar que todas as operações enfileiradas durante *offline* são corretamente enviadas ao servidor após reconexão, na ordem correta e sem duplicação (idempotência verificada via `operationId`).
- **Injeção de conflitos:** Simular cenários onde o servidor retorna dados divergentes dos dados locais e validar que a estratégia de conflito definida (Secção 9.4) é aplicada corretamente.
- **Métricas de sucesso de sincronização:** Definir critérios de aceitação mensuráveis: ex. taxa de sucesso de sync ≥ 99% em cenários de rede instável simulada; tempo máximo de processamento da queue de pendências.

### 9.7. Acessibilidade

As aplicações mobile internas devem respeitar requisitos mínimos de acessibilidade para garantir inclusão e conformidade:

- **Suporte a leitores de ecrã:** Todos os componentes interativos devem ter `accessibilityLabel` definido. Garantir compatibilidade com VoiceOver (iOS) e TalkBack (Android).
- **Contraste e tamanho de texto:** Seguir as diretrizes WCAG 2.1 AA para rácio de contraste (mínimo 4,5:1 para texto normal) e suportar escalamento de texto do sistema.
- **Elementos focáveis:** Garantir que todos os elementos interativos são alcançáveis via navegação por teclado/acessibilidade e têm área de toque mínima de 44×44 pt.
- **Testes de acessibilidade:** Incluir validação de acessibilidade no pipeline de CI (ex: `@testing-library/react-native` com `toBeAccessible`).

---

## 10. Observabilidade

Recomendações de observabilidade especificadas em: [https://ecom4isi.atlassian.net/wiki/spaces/DEVP/pages/5224530029](https://ecom4isi.atlassian.net/wiki/spaces/DEVP/pages/5224530029)

### 10.1. Telemetria Mínima para Mobile

Independentemente da plataforma de observabilidade adotada, as seguintes métricas são obrigatórias para aplicações mobile internas:

| **Métrica**                        | **Descrição**                                                                 | **Prioridade** |
|------------------------------------|-------------------------------------------------------------------------------|----------------|
| Cold start time                    | Tempo desde o lançamento até ao primeiro ecrã interativo (TTI).               | Alta           |
| Taxa de falha de sincronização     | % de operações pendentes que falharam após N tentativas.                      | Alta           |
| Tempo de resposta: local vs remoto | Latência de leitura do armazenamento local vs. chamadas ao BFF (P50/P95).    | Alta           |
| Crash-free sessions                | % de sessões sem crash JS ou nativo.                                          | Alta           |
| Taxa de erros por ecrã             | % de erros por rota/ecrã para identificar pontos de falha recorrentes.        | Média-Alta     |
| Tempo de carga por ecrã            | Duração desde navegação até ecrã totalmente renderizado.                      | Média          |
| ANR / App Not Responding           | Taxa de ANRs reportados (Android); equivalente de *hang* no iOS.              | Média          |
| Duração da queue de pendências     | Tempo médio entre criação de operação pendente e sincronização bem-sucedida.  | Média          |

> **Segmentação:** As métricas de performance (especialmente *cold start* e *crash rate*) devem ser segmentadas por modelo de dispositivo e versão do SO sempre que possível, dado que dispositivos de campo (PDAs) podem apresentar características muito distintas de dispositivos de gama alta.

---

> **Aplicação de Exemplo:** [sample-swrefarch-mobile-internal](https://github.com/mcdigital-devplatforms/sample-swrefarch-mobile-internal)
