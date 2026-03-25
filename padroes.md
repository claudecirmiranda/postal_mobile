# Guia de Melhores Práticas – React Native

Índice
------

- [Componentização e UI Nativa](#componentização-e-ui-nativa)
	- [Primitivos e Abstração](#primitivos-e-abstração)
	- [Performance em Listas (FlashList)](#performance-em-listas-flashlist)
	- [Gestão de Erros Globais (Error Boundaries)](#gestão-de-erros-globais-error-boundaries)
- [Notificações e Comunicação](#notificações-e-comunicação)
	- [Integração com Push Notifications](#integração-com-push-notifications)
- [Formulários, Input Handling e UX](#formulários-input-handling-e-ux)
	- [Tratamento de Teclado](#tratamento-de-teclado)
	- [Controller Pattern](#controller-pattern)
	- [Indicadores de Estado de Sincronização](#indicadores-de-estado-de-sincronização)
- [Performance e Bridge](#performance-e-bridge)
	- [Diretrizes de Otimização](#diretrizes-de-otimização)
		- [Minimizar Passagens na Bridge](#minimizar-passagens-na-bridge)
		- [Animações na UI Thread](#animações-na-ui-thread)
		- [Memoização Agressiva](#memoização-agressiva)
		- [Imagens](#imagens)
- [Navegação e Conteúdos](#navegação-e-conteúdos)
	- [Abertura de URLs (In-App Browser)](#abertura-de-urls-in-app-browser)
	- [Visualização de Documentos (PDF)](#visualização-de-documentos-pdf)
	- [Partilha Nativa](#partilha-nativa)
- [Funcionalidades Interativas](#funcionalidades-interativas)
	- [Leitura de QR Codes](#leitura-de-qr-codes)
	- [Feedback Háptico](#feedback-háptico)
- [Testes e Qualidade Mobile](#testes-e-qualidade-mobile)
	- [Estratégia de Testes](#estratégia-de-testes)
	- [Observabilidade de Interações UI](#observabilidade-de-interações-ui)
	- [Gestão de Dependências](#gestão-de-dependências)

## Componentização e UI Nativa

### Primitivos e Abstração

Evitar o uso direto de `View` e `Text` com estilos *inline* repetitivos. Criar primitivos que incorporem os *Design Tokens*.

> **Acessibilidade:** Componentes customizados devem expor propriedades de acessibilidade (`accessibilityLabel`, `role`, `accessibilityHint`) compatíveis com VoiceOver (iOS) e TalkBack (Android). Nunca omitir estas propriedades em componentes interativos ou que transmitam informação semântica.

**Exemplo de Primitivo (Typography):**

```tsx
import React from "react";
import { Text, TextProps, StyleSheet } from "react-native";
import { theme } from "@/shared/theme";

interface TypographyProps extends TextProps {
  variant?: "h1" | "body" | "caption";
  color?: keyof typeof theme.colors;
}

export const Typography: React.FC<TypographyProps> = ({
  children,
  variant = "body",
  color = "textPrimary",
  style,
  ...props
}) => {
  return (
    <Text
      style={[
        styles.base,
        styles[variant],
        { color: theme.colors[color] },
        style,
      ]}
      {...props}
    >
      {children}
    </Text>
  );
};

const styles = StyleSheet.create({
  base: { fontFamily: theme.fonts.regular },
  h1: { fontSize: 24, fontFamily: theme.fonts.bold },
  body: { fontSize: 16 },
  caption: { fontSize: 12, color: theme.colors.textSecondary },
});
```

### Performance em Listas (FlashList)

Para renderização de coleções, o uso de `FlashList` é mandatório devido à sua arquitetura de reciclagem de *views*.

**Diretriz:** Nunca utilizar `.map()` para renderizar listas *scrolláveis*.

```tsx
import { FlashList } from "@shopify/flash-list";
import { OrderCard } from "./OrderCard";

export const OrderList = ({ data, isLoading }) => {
  return (
    <FlashList
      data={data}
      renderItem={({ item }) => <OrderCard order={item} />}
      estimatedItemSize={120} // Crítico para performance
      onEndReachedThreshold={0.5}
      keyExtractor={(item) => item.id}
      refreshing={isLoading}
    />
  );
};
```

### Gestão de Erros Globais (Error Boundaries)

Componentes que possam falhar em tempo de execução devem ser envolvidos por um `ErrorBoundary` para evitar que erros isolados derrubem a aplicação inteira.

```tsx
import React from "react";
import { View, Text } from "react-native";

interface State {
  hasError: boolean;
}

export class ErrorBoundary extends React.Component<React.PropsWithChildren, State> {
  state: State = { hasError: false };

  static getDerivedStateFromError(): State {
    return { hasError: true };
  }

  componentDidCatch(error: Error, info: React.ErrorInfo) {
    // Reportar ao sistema de observabilidade (ex: Sentry)
    console.error("ErrorBoundary caught:", error, info);
  }

  render() {
    if (this.state.hasError) {
      return (
        <View>
          <Text accessibilityRole="alert">
            Ocorreu um erro inesperado. Tente novamente.
          </Text>
        </View>
      );
    }
    return this.props.children;
  }
}
```

> **Diretriz:** Envolver cada ecrã ou módulo de feature com um `ErrorBoundary` dedicado para garantir isolamento de falhas. O fallback UI deve oferecer uma ação de retry sempre que possível.

---

## Notificações e Comunicação

A aplicação deve ser capaz de receber e exibir informações relevantes ao utilizador mesmo quando em segundo plano ou fechada.

### Integração com Push Notifications

- Utilização dos SDKs oficiais do **MarketingCloud** (Salesforce) para gestão de campanhas e notificações transacionais.
- O registo do dispositivo (`deviceToken`) deve ser renovado a cada login e associado ao ID do utilizador no backend.

**Badging**

- Implementação de lógica local para incrementar/decrementar o contador de notificações não lidas no ícone da aplicação (*App Icon Badge*).
- Sincronização do estado de leitura com o servidor para garantir consistência entre dispositivos.

---

## Formulários, Input Handling e UX

### Tratamento de Teclado

Todo ecrã que contenha inputs deve implementar estratégias de *Keyboard Avoidance*.

- Uso de `KeyboardAvoidingView` (com behavior `padding` no iOS e `height` no Android).
- Uso de bibliotecas como `react-native-keyboard-aware-scroll-view` para formulários longos.

### Controller Pattern

O uso de inputs não controlados (refs) é limitado no React Native. Deve-se priorizar o `Controller` do React Hook Form.

```tsx
<Controller
  control={control}
  name="email"
  render={({ field: { onChange, onBlur, value } }) => (
    <Input
      placeholder="E-mail"
      onBlur={onBlur}
      onChangeText={onChange} // Atenção: onChangeText, não onChange
      value={value}
      autoCapitalize="none"
      keyboardType="email-address"
    />
  )}
/>
```

> **Segurança:** Para campos que contenham dados sensíveis (passwords, PINs, códigos de segurança), garantir o uso de `secureTextEntry={true}` para impedir a visualização do valor e evitar a sua captura em *screenshots* do sistema. Nunca registar o valor destes campos em logs, mesmo em modo de desenvolvimento.

### Indicadores de Estado de Sincronização

Ecrãs que dependem de dados *offline-first* devem comunicar ao utilizador o estado atual da sincronização, evitando ambiguidade sobre a frescura dos dados apresentados.

```tsx
import { useSyncStatus } from "@/shared/hooks/useSyncStatus";

export const SyncStatusBanner = () => {
  const { status, pendingCount } = useSyncStatus();

  if (status === "synced") return null;

  return (
    <View
      accessibilityRole="alert"
      accessibilityLabel={
        status === "offline"
          ? "Sem ligação. Os dados podem não estar atualizados."
          : `A sincronizar ${pendingCount} operações pendentes.`
      }
    >
      <Text>
        {status === "offline"
          ? "Offline – dados podem não estar atualizados"
          : `A sincronizar (${pendingCount} pendentes)…`}
      </Text>
    </View>
  );
};
```

> **Diretriz:** O banner de sincronização deve ser não intrusivo (não bloquear a UI) mas visível o suficiente para que o utilizador compreenda que está a trabalhar com dados locais. Em caso de conflito detetado, apresentar um indicador claro com opção de revisão manual quando aplicável.

---

## Performance e Bridge

O principal *bottleneck* do React Native é a comunicação assíncrona entre a *thread* JS e a *thread* Nativa (Bridge).

### Diretrizes de Otimização

#### Minimizar Passagens na Bridge

Evitar enviar grandes volumes de JSON (strings Base64, listas de grande dimensão) através da bridge em tempo real.

**Abordagem errada** (Base64 diretamente na bridge):

```tsx
// ❌ Não fazer: envia string de 5MB através da bridge
<Image source={{ uri: `data:image/png;base64,${umaStringGiganteDe5MB}` }} />
```

**Abordagem correta** (usar o sistema de ficheiros):

```tsx
import * as FileSystem from "expo-file-system";

// 1. Guardar o binário no disco nativo (diretório de cache)
const path = FileSystem.cacheDirectory + "minha-imagem.png";
await FileSystem.writeAsStringAsync(path, dadosBase64, {
  encoding: FileSystem.EncodingType.Base64,
});

// 2. Passar apenas o caminho (string curta) para o componente
// O componente nativo Image lê diretamente do disco.
<Image source={{ uri: path }} />;
```

> **Gestão de Cache:** Implementar uma política de limpeza do diretório de cache para evitar saturação do armazenamento do dispositivo. Recomenda-se uma estratégia LRU (*Least Recently Used*) ou baseada em TTL (ex: limpar ficheiros com mais de 7 dias). Usar `FileSystem.getInfoAsync` para verificar o tamanho do cache periodicamente e `FileSystem.deleteAsync` para remoção controlada.

#### Animações na UI Thread

Usar estritamente `react-native-reanimated` para animações e gestos. A `Animated` API nativa do React deve ser usada com `useNativeDriver: true`.

#### Memoização Agressiva

No mobile, re-renders são mais custosos do que na web. Componentes de lista (`renderItem`) devem ser sempre memoizados com `React.memo`.

#### Imagens

Usar `expo-image` ou `react-native-fast-image` para cache agressivo e decoding otimizado fora da *Main Thread*.

---

## Navegação e Conteúdos

A aplicação deve prover uma experiência de consumo de conteúdo fluida, mantendo o utilizador dentro do ecossistema da app sempre que possível.

### Abertura de URLs (In-App Browser)

Links externos devem ser abertos dentro da aplicação utilizando `expo-web-browser` (Expo) ou `react-native-inappbrowser-reborn` (CLI), garantindo que a sessão do utilizador seja preservada e oferecendo botões de navegação nativos (fechar, voltar).

### Visualização de Documentos (PDF)

**Requisito:** Renderização de ficheiros (faturas, contratos) integrada na *view*, evitando a quebra de fluxo de enviar o utilizador para um visualizador externo.

**Abordagem (Expo/CLI):** Utilização da biblioteca `react-native-pdf`.

- *Justificativa:* Diferente de soluções baseadas em WebView (que dependem de *wrappers* do Google Docs no Android, gerando riscos de privacidade), esta biblioteca renderiza o PDF nativamente.
- *No Expo:* Requer o uso de **Development Builds** (Prebuild) e Config Plugins, pois utiliza código nativo.

**Fallback iOS:**

Em cenários simples no iOS, o `react-native-webview` pode ser utilizado, pois o WebKit renderiza PDFs nativamente.

### Partilha Nativa

Fluxo: Download do binário → Armazenamento em Cache Local → Invocação da *Share Sheet*.

**Implementação Expo:**

- Utilizar `expo-file-system` para descarregar/guardar o ficheiro no diretório de cache (`FileSystem.cacheDirectory`).
- Invocar `expo-sharing` passando a URI local (`file://...`) e o mime-type correto (`application/pdf`).

**Implementação CLI:**

- Utilizar `react-native-blob-util` para gestão do sistema de ficheiros.
- Utilizar `react-native-share` para invocar a gaveta de partilha, permitindo maior granularidade (ex: partilhar diretamente para WhatsApp ou E-mail, se necessário).

---

## Funcionalidades Interativas

Recursos que utilizam o hardware do dispositivo para enriquecer a experiência do utilizador.

### Leitura de QR Codes

Utilização de `expo-camera` ou bibliotecas de visão computacional (`react-native-vision-camera`) para leitura rápida e precisa de códigos.

### Feedback Háptico

Implementação de feedback tátil (`expo-haptics`) em ações chave (sucesso em formulários, erros, *refresh* de listas) e mecânicas de gamificação para aumentar a imersão.

---

## Testes e Qualidade Mobile

### Estratégia de Testes

- **Unitários:** Jest. Foco em lógica de negócio e hooks (partilhados com web).
- **Componentes:** `@testing-library/react-native`. Foco em renderização, acessibilidade e interação do utilizador.
- **End-to-End (E2E):** Detox ou Maestro. Testes críticos de fluxo (Login, Checkout) a correr em emuladores.

**Exemplo de Teste de Componente (RNTL):**

```tsx
import { render, fireEvent, screen } from "@testing-library/react-native";
import { LoginForm } from "./LoginForm";

test("submits form with valid data", async () => {
  const onSubmit = jest.fn();
  render(<LoginForm onSubmit={onSubmit} />);

  fireEvent.changeText(
    screen.getByPlaceholderText("E-mail"),
    "teste@email.com"
  );
  fireEvent.press(screen.getByText("Entrar"));

  expect(onSubmit).toHaveBeenCalledWith({ email: "teste@email.com" });
});
```

### Observabilidade de Interações UI

Eventos de navegação e interações críticas do utilizador (ex: submissão de formulário, clique em CTA, abertura de ecrã) devem ser instrumentados para permitir correlação com traces de backend.

```tsx
import { trace } from "@opentelemetry/api";
const tracer = trace.getTracer("frontend-tracer");

export function trackUserAction(action: string, attributes?: Record<string, string>) {
  const span = tracer.startSpan(`ui.${action}`, { attributes });
  // O traceId deste span pode ser incluído em chamadas subsequentes ao BFF,
  // permitindo correlação ponta-a-ponta entre a interação do utilizador e o trace de backend.
  span.end();
}

// Exemplo de uso:
// trackUserAction("checkout.submit", { screenName: "CartScreen", itemCount: "3" });
```

> **Diretriz:** Instrumentar pelo menos os eventos de navegação entre ecrãs e as ações de negócio críticas (checkout, login, sincronização manual). Evitar instrumentar todos os cliques — focar nos pontos de valor que necessitam de correlação com o backend ou que alimentam métricas de produto.

### Gestão de Dependências

As bibliotecas definidas neste guia devem ser revistas periodicamente para evitar obsolescência e vulnerabilidades de segurança:

- **Frequência de revisão:** Mínimo trimestral, ou imediatamente após publicação de CVE que afete dependências críticas.
- **Processo:** Executar `npm audit` (ou equivalente) no pipeline de CI para detetar vulnerabilidades conhecidas. Atualizar dependências em branch dedicada com validação de testes antes de merge.
- **Critério de substituição:** Bibliotecas sem manutenção ativa há mais de 12 meses ou incompatíveis com a versão LTS corrente do React Native devem ser avaliadas para substituição e a decisão registada em ADR.
