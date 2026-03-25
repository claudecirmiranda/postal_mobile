# DAS Mobile Apps – Uso Público (B2C, B2B)

> **Detalhe:** React Native preferencialmente para iOS e Android. Hybrid MAUI quando houver necessidade de suportar Windows/Mac.

---

## Histórico de Revisões

| Versão | Data       | Autor(es)       | Resumo das Mudanças                                        |
|--------|------------|-----------------|------------------------------------------------------------|
| 1.0    | 28/11/2025 | Matheus Fraga   | Criação inicial do documento de arquitetura de referência. |
| 2.0    | 16/12/2025 | Matheus Fraga   | Ajustes de formatação para uniformizar todos os DAS.       |
| 3.0    | 17/12/2025 | Matheus Fraga   | Implementação dos requisitos do app Cartão Continente.     |
| 4.0    | 05/01/2026 | Matheus Fraga   | Ajustes conforme comentários deixados no Confluence.       |
| 4.1    | 05/06/2026 | Felipe Klussmann | Pequenos ajustes.                                         |

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

Este documento descreve a arquitetura de referência para o desenvolvimento, implantação e gestão de aplicativos mobile performáticos, escaláveis e consistentes usando React Native como primeira opção. O foco reside na maximização da reutilização de código com a web (quando aplicável), manutenção da performance nativa (60fps, P95 frame drops) e segurança do dispositivo.

Esta arquitetura é o padrão a ser seguido para todos os novos frontends mobile para aplicações com acesso exclusivamente externo à Organização (público). Ela abrange desde a estrutura do código-fonte até a implantação e monitorização em um ambiente de produção.

### 1.2. Público-Alvo

- Engenharia
- Suporte
- Arquitetura

### 1.3. Âmbito

Os seguintes objetivos visam endereçar os seguintes problemas relatados na etapa de Assessment.

| **Objetivo Arquitetural**                              | **Problema Endereçado**                                                                                                                                                                                    | **Solução**                                                                        |
|--------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------|
| Agregação de serviços                                  | Remoção de lógica de negócio e API stitching do Cliente, e redução de complexidade.                                                                                                                        | Backend For Frontend (BFF)                                                         |
| Alta Performance e Estabilidade da App                 | Intermitência, instabilidade do sistema e dificuldade em realizar testes de performance.                                                                                                                   | MVVM (Model-View-ViewModel), e remoção do processamento da *thread* da UI.         |
| Adaptabilidade de Serviços aos Dispositivos Móveis     | O *Backend* de API de propósito geral tende a falhar, pois a experiência móvel frequentemente exige chamadas diferentes, em menor número, e a exibição de menos dados do que as aplicações para *desktop*. | Backend For Frontend (BFF)                                                         |
| Resiliência da App                                     | Falta de suporte a operação *Offline*. A instabilidade da rede em áreas com cobertura limitada exige que as aplicações móveis sejam projetadas para operar *offline*.                                      | Princípio Offline First                                                            |
| Code Signing Automatizado                              | O processo de code signing e a publicação do SDK são complexos devido a problemas de infraestrutura e falta de mecanismos automatizados.                                                                   | Plataforma Expo.dev (biblioteca expo-updates)                                      |

- **Agregação de Serviços:** Uso do padrão Backend For Frontend (BFF) para desacoplar o *frontend* das APIs de negócio, reduzir a complexidade das integrações (autenticação), e evitar que o *frontend* tenha de implementar a lógica de agregação (*stitching*).
- **Alta Performance e Estabilidade da App:** A arquitetura é proposta com a aplicação correta do MVVM (UI/State/Data) e a remoção do processamento na *thread* da UI, um fator chave para mitigar o elevado número de ANR's (*Application Not Responding*).
- **Adaptabilidade de Serviços aos Dispositivos Móveis:** Com a aplicação do padrão BFF, os endpoints são adaptados às necessidades dos dispositivos móveis, como: agregação dos resultados das chamadas a outras APIs, causando assim a redução das chamadas ao backend realizadas pelo app; além disso, itens como paginação e classificação (*sorting*) dos dados podem ser realizados no BFF.
- **Resiliência da App:** A aplicação deve ser concebida para funcionar offline por defeito, sendo a conectividade online uma consideração secundária. O objetivo é garantir que os utilizadores tenham acesso a funcionalidades críticas (como contagens e inventários) mesmo quando a rede está instável ou inexistente. É necessário um mecanismo de sincronização regular com o backend.
- **Code Signing Automatizado:** A biblioteca expo-updates suporta assinatura de código end-to-end usando criptografia de chave pública. A assinatura de código permite aos programadores assinar criptograficamente as suas atualizações com as suas próprias chaves. As assinaturas são então verificadas no cliente antes da atualização ser aplicada, o que garante que ISPs, CDNs, fornecedores de cloud e até o próprio EAS não podem interferir com as atualizações executadas pelas aplicações.

---

## 2. Dependências e Referências

- **Expo Application Services (EAS):** Conjunto de serviços hospedados para projetos React Native para automatizar CI/CD (pode ser executado localmente). O Expo é um projeto open-source que oferece ferramentas para construir e manter aplicações React Native em qualquer escala. O EAS é um conjunto de *hosted services* que permite:
  - Construir, submeter e atualizar a aplicação.
  - Automatizar todos esses processos.
  - Colaborar com a equipa.
  - Resolver problemas que requerem recursos físicos, como servidores de aplicações e CDNs para fornecer atualizações *over-the-air*.

- **React Native Framework:** Configuração mínima utilizando `react-native-elements` para UI e `@react-navigation` para navegação.

- **Offline-First Libs:** `react-native-sqlite-storage` para persistência robusta e `@react-native-community/netinfo` para monitorização de rede.

- **Comunicação e Notificações:** SDKs do MarketingCloud para Push Notifications e bibliotecas de Event Source para SSE.

---

## 3. Regras de Utilização

Cada tipologia terá abordados aspetos específicos da tipologia, incluindo, sempre que aplicável, referências a diretrizes, padrões gerais e boas práticas.

Este documento não é um conjunto de regras inflexíveis, mas sim um guia de fortes recomendações. Desvios são permitidos, mas devem ser justificados, documentados em uma ADR (Architecture Decision Record) e aprovados pelo Comité de Arquitetura.

---

## 4. Arquitetura da Tipologia

> **Diagrama:** Consultar o diagrama de arquitetura no Confluence (draw.io: *Untitled Diagram-1770304017322.drawio*).

### 4.1. Camadas

- **UI Components e Screens**
  - Camada visível para o utilizador: tudo que aparece no ecrã — botões, formulários, listas, textos, navegação entre ecrãs, estilo visual, etc.
  - Responsabilidades:
    - Renderizar a interface.
    - Capturar interações do utilizador.
    - Mostrar estados de carga, erro e feedback.
    - Integrar com a lógica de negócio — usando hooks, contexto, estado global ou store.

- **Local Database**
  - Base de dados ou armazenamento persistente no dispositivo.
  - Responsabilidades:
    - Permitir que a app funcione *offline*: o utilizador pode ver dados previamente sincronizados, criar e editar dados sem dependência de conexão.
    - Agir como fonte de verdade no cliente — a UI e lógica consultam diretamente a base local, não a rede, garantindo consistência e velocidade.
    - Persistir informações entre sessões, reinícios da app ou do dispositivo.

- **Queue de Operações Pendentes (sync pendente)**
  - Fila ou lista de operações realizadas pelo utilizador enquanto a app está offline — por exemplo, criação de ordens, edições, deleções, etc. Cada operação é registada localmente com marcação de "pendente de sincronização". Esta fila pode estar como parte da *local database* ou em estrutura separada (ex: tabela de *pending changes*).
  - Responsabilidades:
    - Garantir que ações do utilizador não se percam — mesmo que ele esteja offline ou encerre a app.
    - Permitir *optimistic UI*: quando o utilizador cria ou edita algo, a app já reflete na interface local, sem precisar aguardar resposta do servidor.
    - Servir de buffer de sincronização: quando a rede voltar, a app envia todas as operações pendentes de forma organizada para o servidor.

- **Lógica de Sync + Detecção de Rede (Sync Engine / Network Manager)**
  - Módulo responsável por monitorar o estado de conectividade de rede (online/offline), detetar quando a rede ficar disponível e disparar a sincronização das pendências.
  - Responsabilidades:
    - Orquestrar a comunicação entre local e remoto: ao detetar que a rede está disponível, processar a queue de pendências, sincronizar dados e garantir que a base local e o servidor convirjam.
    - Garantir que a experiência do utilizador seja transparente — ele pode usar a app offline e a sincronização acontece em background ao reconectar.
    - Lidar com casos típicos de conectividade móvel: reconexão intermitente, redes instáveis, reconciliação de dados, conflitos, retry, etc.

- **HTTP Client (REST/GraphQL), sempre apontando para o BFF**
  - Cliente que a app usa para fazer requisições ao backend (quando online) — por exemplo, usando GraphQL (via Apollo Client) para consultar dados, enviar mutations e sincronizar dados com o servidor.
  - Responsabilidades:
    - Permitir comunicação com o servidor backend: buscar dados globais, enviar novos dados criados localmente, receber dados de outros utilizadores, obter atualizações, etc.
    - Quando combinado com a lógica de sync, serve para "materializar" as operações pendentes: as ações registadas na queue acabam tornando-se requisições ao servidor.

### 4.2. Melhores Práticas em React Native

Consultar: [https://ecom4isi.atlassian.net/wiki/x/E4CSNwE](https://ecom4isi.atlassian.net/wiki/x/E4CSNwE)

---

## 5. Estrutura do Projeto

A estrutura segue o padrão *Feature-Based* definido para Web, com adaptações para navegação e assets nativos.

```
mobile-app/
├── src/
│   ├── App.tsx                               # Entry Point (Configura Providers)
│   ├── navigation/                           # Núcleo de Navegação
│   │   ├── routes.ts                         # Definição de Rotas (Enums/Consts)
│   │   ├── types.ts                          # Tipagem de Parâmetros (NavigationProp)
│   │   ├── MainStack.tsx                     # Stack Navigator Principal
│   │   └── AuthStack.tsx                     # Stack de Autenticação
│   ├── features/                             # Módulos de Negócio
│   │   └── orders/
│   │       ├── components/                   # Componentes Visuais (Nativos)
│   │       │   └── OrderCard.tsx
│   │       ├── screens/                      # Ecrãs registadas no Navigator
│   │       │   ├── OrdersListScreen.tsx
│   │       │   └── OrderDetailScreen.tsx
│   │       ├── hooks/                        # Lógica de Negócio
│   │       └── data-access/                  # Camada de Dados
│   ├── shared/
│   │   ├── components/                       # UI Kit Nativo
│   │   │   ├── ui/
│   │   │   │   ├── Box.tsx
│   │   │   │   ├── Typography.tsx
│   │   │   │   └── Touchable.tsx
│   │   ├── utils/
│   │   │   └── storage/
│   │   │       ├── mmkv.adapter.ts
│   │   │       └── secure.adapter.ts
├── assets/                                   # Recursos Nativos (Fonts, Splash)
├── babel.config.js
└── package.json
```

---

## 6. Tecnologias Utilizadas

### 6.1. Stack Tecnológico Mobile

Diferenças e adições em relação ao Stack Web.

| **Categoria**  | **Tecnologia**            | **Observações**                                                                                                          |
|----------------|---------------------------|--------------------------------------------------------------------------------------------------------------------------|
| Runtime        | Hermes Engine             | Motor JavaScript otimizado para React Native. Recomendado para redução de TTI (*Time to Interaction*) e uso de memória. |
| Navegação      | React Navigation 6+       | Expo Router (opcional).                                                                                                  |
| Estilização    | StyleSheet                | Styled Components.                                                                                                       |
| Listas         | FlashList (Shopify)       | Substituto do `FlatList` para listas com mais de 50 itens ou layout complexo.                                           |
| Storage        | MMKV                      | Expo SecureStore.                                                                                                        |
| Forms          | React Hook Form + Zod     | Mesma stack da web. Uso obrigatório de `Controller` para inputs controlados.                                             |
| Animação       | Reanimated 3              | Animações declarativas que rodam na *UI Thread*, evitando gargalos na *JS Thread*.                                       |
| Notificações   | MarketingCloud SDK        | Integração oficial para Push Notifications e Badging.                                                                    |
| SSE            | EventSource               | Suporte a *Server Sent Events* para atualizações em tempo real unidirecionais.                                           |

### 6.2. Matriz de Decisão de Inicialização (Expo vs CLI)

A escolha da ferramenta de *build* e *runtime* deve seguir a seguinte matriz decisória:

| **Critério**       | **Expo (Managed Workflow)**                                                                                            | **React Native CLI (Bare)**                                                                                              |
|--------------------|------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------|
| **Cenário de Uso** | 95% das aplicações corporativas padrão; Integrações nativas comuns (Câmera, Push, Biometria).                          | Dependência de SDKs nativos proprietários legados; Necessidade de alteração profunda no *Build System* (Gradle/Podfile). |
| **Vantagens**      | *Over-the-Air (OTA) Updates*; Configuração zero de ambiente nativo; *Prebuild* contínuo.                               | Controle absoluto sobre o código nativo (Java/Kotlin/Obj-C/Swift).                                                       |
| **Restrições**     | Incompatibilidade com bibliotecas que exigem *linking* manual complexo ou *hooks* nativos de baixo nível não expostos. | Maior complexidade de CI/CD; Curva de aprendizado elevada para manutenção de ambiente.                                   |

---

## 7. Segurança

Observar Definições Globais em: [https://ecom4isi.atlassian.net/wiki/spaces/DEVP/pages/5217517761](https://ecom4isi.atlassian.net/wiki/spaces/DEVP/pages/5217517761)

### 7.0. Requisitos Mínimos de Segurança para Mobile

Todo app móvel desenvolvido sob esta arquitetura deve, obrigatoriamente, implementar os seguintes controlos de segurança de base. Os detalhes de implementação estão nas secções subsequentes e nos documentos especializados referenciados acima.

1. **SSL Pinning:** Todos os apps devem implementar SSL Pinning para prevenir ataques Man-in-the-Middle (MITM). Nenhuma comunicação de rede deve ocorrer sem validação do certificado do servidor ou da chave pública.
2. **Armazenamento Seguro de Segredos:** Tokens de autenticação, refresh tokens e quaisquer dados sensíveis devem ser armazenados exclusivamente em `expo-secure-store` ou `react-native-keychain` (Keystore/Keychain). É proibido o uso de `AsyncStorage` ou armazenamento em texto claro para dados sensíveis.
3. **Detecção de Root/Jailbreak:** Apps que lidam com funcionalidades críticas (ex: pagamentos, dados pessoais sensíveis) devem implementar detecção de dispositivos comprometidos e bloquear ou restringir o acesso nessas situações.
4. **Ofuscação de Build:** Builds de produção devem habilitar minificação e ofuscação de código para dificultar engenharia reversa.
5. **Prevenção de Logging de PII:** É estritamente proibido registar em logs quaisquer dados de identificação pessoal (nomes, emails, tokens, payloads completos). Logs devem ser sanitizados antes de qualquer envio a sistemas de monitorização.
6. **Gestão de Permissões Just-in-Time:** Permissões de hardware e dados sensíveis devem ser solicitadas apenas no momento do uso, seguindo o princípio do menor privilégio.

### 7.1. Gestão de Estado e Persistência Segura

Diferente da web (localStorage), o ambiente mobile exige segregação de dados.

- **Dados Não-Sensíveis** (Cache, Configurações): Utilizar `react-native-mmkv` ou `AsyncStorage`.
- **Dados Sensíveis** (Tokens, Refresh Tokens): Utilizar `expo-secure-store` ou `react-native-keychain`. Estes dados são criptografados e armazenados no Keystore (Android) ou Keychain (iOS).

> **Gestão de Estado Global:** Para gestão de estado global, preferir Zustand com persistência via MMKV. Alternativas (Redux, MobX, Context API) devem ser validadas com Arquitetura e registadas em ADR quando justificadas por requisitos específicos.

Exemplo de Adapter para Zustand (Persist Middleware):

```tsx
import { StateStorage } from "zustand/middleware";
import { MMKV } from "react-native-mmkv";

const storage = new MMKV();

export const mmkvStorage: StateStorage = {
  setItem: (name, value) => storage.set(name, value),
  getItem: (name) => {
    const value = storage.getString(name);
    return value ?? null;
  },
  removeItem: (name) => storage.delete(name),
};
```

### 7.2. Controles de Segurança e Permissões

A implementação de controles de segurança deve respeitar o fluxo de trabalho escolhido (Expo ou CLI), garantindo a mesma eficácia de proteção.

- Prevenção de logging de PII (Personally Identifiable Information).
- Hardening de build (minify/obfuscation, jailbreak/root gating para features críticas).

#### 7.2.1. Gestão de Permissões

O acesso a hardware e dados sensíveis deve ser solicitado apenas no momento do uso ("Just-in-time").

**Via Expo**
- Utilizar as APIs nativas do SDK (ex: `expo-camera`, `expo-media-library`).
- Configurar as chaves de uso no `app.json` (plugins) para garantir que as descrições de uso apareçam no `Info.plist` e `AndroidManifest.xml`.

**Via React Native CLI**
- Utilizar a biblioteca `react-native-permissions` para uma gestão unificada de status (Granted, Denied, Blocked) em ambas as plataformas.
- Configuração manual obrigatória no `Podfile` e `AndroidManifest.xml` para incluir apenas as permissões necessárias, reduzindo a superfície de ataque.

### 7.3. Geolocalização

**Via Expo**
- Utilizar `expo-location`. Configurar explicitamente as permissões de *Foreground* e *Background* no manifesto via Config Plugins.

**Via React Native CLI**
- Utilizar `react-native-geolocation-service` (que utiliza o Google Play Services no Android para maior precisão e performance) ou `@react-native-community/geolocation`.
- Implementar verificação manual para garantir que o GPS está ativo antes de solicitar a localização.

### 7.4. Integridade do Dispositivo (Root/Jailbreak)

Detecção de ambientes comprometidos para bloquear funcionalidades críticas (ex: pagamentos).

**Via Expo**
- Utilizar `expo-device` para verificações básicas (`isRootedExperimental`). Para verificações robustas, injetar bibliotecas nativas como `jail-monkey` através de **Expo Config Plugins** (requer Prebuild/Dev Client).

**Via React Native CLI**
- Integração direta de bibliotecas como `jail-monkey` ou `react-native-jailbreak-detector`. Implementar verificação na inicialização do app (`App.tsx` ou `index.js`) para fechar a aplicação ou limitar acesso caso o dispositivo esteja comprometido.

### 7.5. Criptografia e Armazenamento Seguro

**Via Expo**
- Armazenamento de chaves: `expo-secure-store`.
- Operações criptográficas: `expo-crypto` (SHA, UUID).

**Via React Native CLI**
- Armazenamento de chaves: `react-native-keychain` ou `react-native-encrypted-storage` (wrapper do EncryptedSharedPreferences/Keychain).
- Criptografia Pesada (AES/RSA): Utilizar `react-native-quick-crypto` (implementação de alta performance via JSI) ou `react-native-rsa-native`.

### 7.6. Autenticação Biométrica

Camada de conveniência vinculada a uma chave criptográfica segura.

**Via Expo**
- Utilizar `expo-local-authentication`. Abstrai FaceID e TouchID de forma simples.

**Via React Native CLI**
- Utilizar `react-native-biometrics`. Esta biblioteca permite não apenas validar a biometria, mas criar pares de chaves criptográficas (Private/Public keys) que só podem ser acedidas mediante sucesso biométrico, oferecendo segurança de nível bancário.

### 7.7. Segurança de Rede (SSL Pinning)

Prevenção contra ataques Man-in-the-Middle (MITM) fixando o certificado do servidor ou chave pública.

**Via Expo**
- Utilizar a biblioteca `react-native-ssl-public-key-pinning` configurada via **Config Plugin** para injetar os hashes dos certificados nos arquivos nativos durante o build do EAS.

**Via React Native CLI**
- **Android:** Configuração via Network Security Configuration (XML nativo).
- **iOS:** Configuração via TrustKit ou injeção direta no `Info.plist`.
- **Alternativa Cross-Platform:** Utilizar `react-native-ssl-pinning` que atua na camada do fetch/axios.

---

## 8. Infraestrutura

Recomenda-se o serviço de Hosting da plataforma Expo para CI/CD com code signing e publicação em loja de apps automatizadas.

### 8.1. Infraestrutura Gerenciada (Expo EAS) – Padrão

Recomendado para 95% dos casos devido à redução de complexidade de DevOps.

- **Build:** 100% em nuvem via EAS Build. Elimina a necessidade de Macs locais para compilar iOS.
- **Distribuição:** EAS Submit para envio automatizado à Google Play e App Store.
- **OTA Updates:** Expo Updates para correção de bugs críticos em produção sem passar pelo processo de revisão da loja, respeitando as regras da Apple/Google: OTA apenas para JS/Assets.

> **Diretriz de OTA:** Atualizações OTA devem ser testadas em canal de staging com 5–10% de utilizadores antes de rollout completo. Implementar mecanismo de rollback automático em caso de aumento de crash rate acima do limiar definido para o projeto (ex: > 2%). Atualizações com impacto em código nativo requerem obrigatoriamente nova submissão às lojas.

### 8.2. Infraestrutura Standalone (React Native CLI) – *Exceção*

Para projetos que optarem pelo *Bare Workflow* devido a requisitos nativos específicos.

- **Automação Local/CI:** Uso recomendado do Fastlane (`Fastfile`) para padronizar scripts de build, assinatura de código e deploy.
- **CI/CD:** Pipelines configuradas no GitHub Actions ou Bitrise.
- *Nota:* Requer *runners* macOS para compilação do iOS.
- **Distribuição de Testes:** Firebase App Distribution ou TestFlight para envio de builds de QA/Homologação.

### 8.3. Gestão e Monitorização

- **Feature Toggles:** Implementação de sistema de flags para ativar/desativar funcionalidades sem deploy.
- **Observabilidade:** Integração obrigatória com plataforma de monitorização para rastreamento de performance e estabilidade.
- **Logging:** Sistema estruturado de logs para captura de informações e erros aplicacionais, segregando níveis (Info, Warn, Error) e evitando registo de dados sensíveis.
- **Crash Reporting** (ex: Sentry/Firebase Crashlytics).
- **Release Health** (crash-free sessions, ANR, startup time).

---

## 9. Padrões e Princípios Arquiteturais

### 9.1. Princípio Offline-First

A aplicação deve ser concebida para funcionar offline por padrão, tratando a conectividade como uma melhoria progressiva.

- **Local Database:** Utilização de SQLite (`react-native-sqlite-storage`) como fonte de verdade no cliente. A UI consulta a base local, não a rede.
- **Queue de Operações:** Ações do utilizador (ex: criar pedido) realizadas offline são armazenadas em uma fila local ("pendente de sync") para garantir que nenhuma intenção do utilizador seja perdida.
- **Sync Engine:** Um módulo dedicado monitoriza a rede (`@react-native-community/netinfo`) e, ao detetar reconexão, processa a fila de pendências enviando-as ao servidor e reconciliando os dados.

> **Testes de Integração Offline:** As estratégias de teste devem cobrir o fluxo completo de sincronização (offline → online) em testes automatizados. Recomenda-se o uso de mocks de conectividade para simular estados de rede nos testes de integração, garantindo a resiliência do Sync Engine.

### 9.2. Atomic Design Adaptado

A organização dos componentes visuais deve seguir uma adaptação do Atomic Design para promover reutilização e consistência visual entre as features.

- **Átomos** (`shared/components/ui`): Componentes indivisíveis e agnósticos ao negócio.
  - *Exemplos:* `Typography`, `Box`, `Button`, `Icon`.
- **Moléculas** (`shared/components`): Agrupamento de átomos que formam uma unidade funcional simples.
  - *Exemplos:* `InputGroup` (Label + Input + ErrorMessage), `UserAvatar` (Image + Name).
- **Organismos** (`features/**/components`): Componentes complexos que formam seções distintas de uma interface, com contexto de negócio.
  - *Exemplos:* `OrderCard`, `ProductList`, `LoginForm`.
- **Templates/Screens** (`features/**/screens`): A estrutura da página onde os organismos são orquestrados.

### 9.3. GraphQL

GraphQL é opcional e, quando usado, deve ser implementado no BFF (BFF GraphQL), não diretamente contra múltiplos serviços.

Consultar: [https://ecom4isi.atlassian.net/wiki/spaces/DEVP/pages/edit-v2/5212340226#GraphQL](https://ecom4isi.atlassian.net/wiki/spaces/DEVP/pages/edit-v2/5212340226#GraphQL)

#### 9.3.1. Integração com o App Mobile (Apollo Client)

A comunicação entre o frontend (React Native / Expo) e o backend GraphQL ocorre via:

```tsx
const client = new ApolloClient({
  uri: "https://api.exemplo.com/bff/graphql",
  cache: new InMemoryCache(),
  headers: {
    Authorization: `Bearer ${token}`,
  },
});
```

**Benefícios:**
- Redução de roundtrips por ecrã.
- Payload sob medida.
- Caching consistente.
- Compatível com evolução incremental.

**Guardrails:**
- Persisted Queries / query allowlist.
- Limites de complexidade/custo e depth.
- Timeouts e caching no BFF.

### 9.4. Offline-First

Consultar: [https://ecom4isi.atlassian.net/wiki/spaces/DEVP/pages/edit-v2/5212340226#Princípio-Offline-First](https://ecom4isi.atlassian.net/wiki/spaces/DEVP/pages/edit-v2/5212340226#Princ%C3%ADpio-Offline-First)

### 9.5. Cache de Dados de Referência

Dados frequentemente consultados mas raramente alterados (ex: catálogos de produtos, listas de configuração, feature flags) devem seguir uma estratégia de cache dedicada para otimizar performance e consumo de dados móveis:

- **TTL explícito:** Definir um tempo de vida (*Time to Live*) por tipo de dado. Dados de catálogo podem ter TTL de horas; configurações de feature flags podem exigir TTL mais curto.
- **Invalidação ativa:** O Sync Engine deve ser capaz de invalidar e refrescar o cache de dados de referência ao receber sinalização do BFF (ex: via ETag, header de versão ou evento SSE).
- **Stale-while-revalidate:** Servir dados do cache imediatamente enquanto a atualização ocorre em background, evitando bloqueio da UI.

### 9.6. Gestão de Versões de API e Compatibilidade Retroativa

A app deve ser resiliente a evoluções do backend sem exigir atualizações forçadas nas lojas:

- **Campos novos:** Ignorar campos desconhecidos nas respostas (tolerância a adições).
- **Campos obsoletos:** O BFF deve manter compatibilidade retroativa por, no mínimo, 2 versões de release do app.
- **Versionamento do BFF:** Utilizar versionamento de API (ex: `/v1/`, `/v2/`) e comunicar depreciações com antecedência às equipas de frontend.
- **Forced Update:** Implementar mecanismo de *forced update* (via feature flag ou resposta do BFF) para situações em que versões antigas do app não são mais compatíveis com o backend.

### 9.7. Privacidade de Dados (GDPR/LGPD)

Além das restrições de logging descritas em Segurança, a persistência local deve cumprir os seguintes requisitos de privacidade:

- **Encriptação de dados locais:** Bases de dados SQLite e ficheiros de cache devem ser encriptados em repouso, especialmente quando contiverem dados pessoais do utilizador.
- **Retenção mínima:** Dados pessoais não devem ser persistidos localmente por mais tempo do que o necessário para a funcionalidade. Definir TTL de purga para dados de utilizador no armazenamento local.
- **Exclusão ao desinstalar:** Garantir que dados sensíveis armazenados no Keychain (iOS) são removidos ao desinstalar a app (comportamento padrão no Android; requer configuração explícita no iOS via `kSecAttrAccessibleAfterFirstUnlock` e limpeza no primeiro boot pós-reinstalação).
- **Consentimento e opt-out:** Telemetria e analytics de UX devem respeitar as preferências de consentimento do utilizador, com mecanismo de opt-out funcional.

---

## 10. Observabilidade

Observar o padrão em: [https://ecom4isi.atlassian.net/wiki/spaces/DEVP/pages/5217517761](https://ecom4isi.atlassian.net/wiki/spaces/DEVP/pages/5217517761)

A observabilidade deve seguir um modelo unificado para atender aplicações *frontend* (Mobile React Native / Expo e Web) e componentes *backend* (BFF/APIs), com o objetivo de reduzir instrumentação manual, garantir consistência de dados e permitir correlação ponta-a-ponta.

### 10.1. Princípios

- **OpenTelemetry como padrão** para *traces*, *metrics* e *logs*.
- **Instrumentação automática por defeito** (HTTP/GraphQL, navegação, erros, runtime). **Instrumentação manual apenas quando há valor claro** (spans de negócio e medições específicas).
- **Correlação end-to-end** via `traceId` e propagação de contexto: **Frontend → BFF → serviços downstream** devem manter o mesmo *trace* sempre que possível.
- **Atributos padronizados**: a maioria deve vir de forma automática (SDKs/agents). Evitar "setar campos" manualmente em cada chamada.

> **Instrumentação Mobile:** A instrumentação deve priorizar: (1) propagação de `traceId` via headers HTTP para o BFF; (2) *sampling* configurável por ambiente (100% em dev, 1–10% em prod); (3) métricas de UX obrigatórias: cold start, screen load time e crash-free sessions. Métricas opcionais, quando suportadas tecnicamente: FPS/jank e ANR rate.

---

### 10.2. Stack Padrão

- **Collector:** OpenTelemetry Collector (HTTP OTLP).
- **Tracing/Metrics:** Grafana (Tempo/Prometheus) e Prometheus.
- **Logs:** Stack corporativa (ex: Loki/Elastic, conforme standard interno).
- **Analytics de UX (opcional, complementar):** PostHog (eventos de produto/uso), *não substitui* telemetria técnica.

> **Nota:** UX Analytics (PostHog) é para comportamento do utilizador e funis; OTel é para saúde, performance e debugging técnico.

---

### 10.3. Modelo de Dados Unificado (mínimo comum)

#### 10.3.1. Resource Attributes (definidos 1 vez por app/serviço)

Estes atributos devem ser aplicados a nível de **Resource** (inicialização do SDK/agent), e não repetidos manualmente por operação:

- `service.name` (ex: `mobile-app`, `web-app`, `bff-orders`)
- `service.version` (versão do build/release)
- `deployment.environment` (dev/qa/prod)
- `team.name` / `owner` (opcional, se existir standard)
- Para **frontend**: `device.platform` (ios/android/web), `app.build_number` / `app.bundle_id` (quando disponível).
- Para **backend**: `cloud.provider`, `k8s.cluster.name`, `k8s.namespace.name` (via auto-instrumentação/collector).

#### 10.3.2. Span Attributes (padrão de correlação)

O mínimo para correlação e troubleshooting deve vir automaticamente:

- HTTP: `http.method`, `http.url`/`url.full` (com sanitização), `http.status_code`.
- Rede: latência, timeouts, retries (quando suportado pela instrumentação).
- Erros: `exception.type`, `exception.message` (sem PII), `exception.stacktrace` (quando permitido).

**Regra:** atributos de infraestrutura/ambiente = automáticos; atributos de negócio = manuais e poucos.

---

### 10.4. Instrumentação por Camada

#### 10.4.1. Frontend (Mobile React Native / Expo e Web)

**Obrigatório**
- Erros não tratados (crashes JS) e exceções capturadas.
- Medições de performance percebida: `app.startup` (cold start / time-to-interactive), `screen.load` / `route.change` (navegação).
- Chamadas de rede: HTTP para o BFF; GraphQL (se usado) com visibilidade de `operationName` (evitar logar a query completa).

**Preferir automático**
- Interceptors automáticos (HTTP client) para criar spans de rede.
- Hooks/listeners de navegação para spans de *screen/route*.
- Integração com *crash reporting* (Sentry/Crashlytics) correlacionando com `traceId` sempre que possível.

**Instrumentação manual apenas quando fizer sentido**
- Spans de negócio curtos e raros (ex: `order.checkout.submit`, `inventory.sync`) com 2–4 atributos no máximo (ex: `flow`, `feature`, `result`).

#### 10.4.2. Backend (BFF/APIs)

**Obrigatório**
- Auto-instrumentação OTel para: inbound requests (HTTP/GraphQL) e outbound calls (HTTP/gRPC/db/messaging quando suportado).
- **Propagação de contexto** (W3C Trace Context) a partir do *frontend*: o BFF deve aceitar e propagar `traceparent` / `tracestate`.
- **Logs estruturados** com correlação: `traceId` e `spanId` incluídos nos logs.

**Instrumentação manual (mínima)**
- Spans em pontos de agregação do BFF: `bff.aggregate` com atributos como `backend.count`, `cache.hit`, `degraded_mode` (sem PII).

---

### 10.5. Padrão de Instrumentação (recomendado)

#### 10.5.1. Frontend: OTel + instrumentação leve

```tsx
// observability/init.ts (conceitual)
import { Resource } from '@opentelemetry/resources';
import { SemanticResourceAttributes } from '@opentelemetry/semantic-conventions';

export function initObservability() {
  const resource = new Resource({
    [SemanticResourceAttributes.SERVICE_NAME]: 'mobile-app',
    [SemanticResourceAttributes.SERVICE_VERSION]: '1.0.0',
    'deployment.environment': 'prod',
    'device.platform': 'react-native',
  });

  // Provider/exporter para OTLP/HTTP apontando para o Collector
  // + integrações automáticas (HTTP client / navigation / errors) conforme SDK escolhido.
}
```

#### 10.5.2. Navegação: spans por ecrã/rota

```tsx
// navigationRef.ts (conceitual)
import { trace } from '@opentelemetry/api';
const tracer = trace.getTracer('frontend-tracer');

let currentSpan: any;

export function onRouteChange(routeName: string) {
  if (currentSpan) currentSpan.end();

  currentSpan = tracer.startSpan('screen.view', {
    attributes: { 'route.name': routeName },
  });
}
```

#### 10.5.3. Rede: spans automáticos + enriquecimento mínimo

```tsx
// api.ts (conceitual)
import axios from 'axios';

export const api = axios.create();

// O ideal é uma integração OTel pronta para Axios/fetch.
// Se não existir, manter apenas o essencial e evitar campos manuais redundantes.
```

---

### 10.6. Erros e Crash Reporting

**Obrigatório**
- Capturar erros JS não tratados e reportar.
- Sanitizar mensagens para evitar PII.
- Associar `traceId` quando possível (principalmente em flows com rede).

```tsx
// error-handler.ts (conceitual)
import { trace } from '@opentelemetry/api';
const tracer = trace.getTracer('frontend-tracer');

export function initGlobalErrorHandler() {
  // Em RN, ErrorUtils existe; na Web, usar window.onerror/unhandledrejection.
  // Implementação deve integrar com a ferramenta padrão de crash reporting.
}
```

---

### 10.7. O que deve ser coletado (KPI mínimo comum)

#### 10.7.1. Performance do Utilizador (Frontend)

- Cold start / TTI (quando aplicável).
- Tempo de render inicial por ecrã.
- Latência de chamadas ao BFF (P50/P95).
- Taxa de erros por ecrã/rota.
- (Opcional) FPS/jank se houver suporte técnico.

> **Segmentação por Dispositivo:** Métricas de performance (especialmente cold start e frame drops) devem ser segmentadas por modelo de dispositivo e versão do SO sempre que possível. Esta segmentação é crítica para priorizar otimizações, dado que dispositivos de gama baixa tendem a ser mais representativos na base de utilizadores móveis.

#### 10.7.2. Performance do Serviço (BFF/APIs)

- Latência inbound (P50/P95/P99).
- Taxa de erros (4xx/5xx) e timeouts.
- Retries/circuit breaker (se existir).
- Dependências downstream mais lentas.

#### 10.7.3. Fiabilidade

- Crash-free sessions (frontend).
- ANR / app not responding (mobile, se houver).
- SLO básico (ex: disponibilidade + latência P95).

---

### 10.8. Boas Práticas e Restrições

- **Não coletar PII** (nomes, emails, tokens, payloads completos).
- **Sanitizar URLs e queries** (especialmente GraphQL).
- **Sampling:** aplicar amostragem em produção (configurada no Collector/SDK) para controlar custo.
- **Versionamento:** garantir que `service.version` reflete o build para permitir comparar releases.
- **Correlações:** garantir propagação de contexto do frontend para o BFF (headers de trace).

---

> **Aplicação de Exemplo:** [sample-swrefarch-mobile-public](https://github.com/mcdigital-devplatforms/sample-swrefarch-mobile-public)
