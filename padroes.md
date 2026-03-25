# Guia de Arquitetura Web – Tipagem, Serviços, Features, Routing, Testes e Performance

Índice
------

- [Camada de Tipagem (Shared Types)](#camada-de-tipagem-shared-types)
- [Camada de Serviços de API (Shared)](#camada-de-serviços-de-api-shared)
	- [Configuração do Cliente HTTP](#configuração-do-cliente-http)
	- [Interceptadores (Interceptors)](#interceptadores-interceptors)
	- [Serviços de Domínio (Service Layer)](#serviços-de-domínio-service-layer)
- [Implementação de Feature (Exemplo: Orders)](#implementação-de-feature-exemplo-orders)
	- [Gestão de Estado de Servidor (React Query)](#gestão-de-estado-de-servidor-react-query)
	- [Componentes da Feature](#componentes-da-feature)
- [Gestão de Estado Global (Zustand)](#gestão-de-estado-global-zustand)
- [Validação e Formulários (Zod + React Hook Form)](#validação-e-formulários-zod-react-hook-form)
- [Roteamento e Guards de Autenticação](#roteamento-e-guards-de-autenticação)
	- [Estrutura de Rotas e Lazy Loading](#estrutura-de-rotas-e-lazy-loading)
	- [Route Guards](#route-guards)
- [Estratégia de Testes](#estratégia-de-testes)
	- [Testes Unitários (Lógica Pura)](#testes-unitários-lógica-pura)
	- [Testes de Componentes (Interação)](#testes-de-componentes-interação)
- [Performance por Defeito](#performance-por-defeito)
	- [Diretrizes Mandatórias](#diretrizes-mandatórias)
	- [Anti-Patterns de Performance](#anti-patterns-de-performance)
- [Estado Previsível, Rastreável e Sem Prop Drilling](#estado-previsível-rastreável-e-sem-prop-drilling)
	- [Eliminação de Prop Drilling](#eliminação-de-prop-drilling)

## Camada de Tipagem (Shared Types)

Definição de contratos de dados partilhados. Prioriza-se o uso de `interfaces` para modelos de dados e `types`/`enums` para uniões e constantes.

**Ficheiro:** `src/shared/types/models/order.types.ts`

```ts
export interface Order {
  id: string;
  customerId: string;
  items: OrderItem[];
  totalAmount: number;
  status: OrderStatus;
  createdAt: Date;
  updatedAt: Date;
}

export interface OrderItem {
  id: string;
  productId: string;
  productName: string;
  quantity: number;
  unitPrice: number;
  totalPrice: number;
}

export enum OrderStatus {
  PENDING = "PENDING",
  PROCESSING = "PROCESSING",
  COMPLETED = "COMPLETED",
  CANCELLED = "CANCELLED",
}
```

---

## Camada de Serviços de API (Shared)

Esta camada centraliza a comunicação HTTP, garantindo consistência no tratamento de requisições, erros e autenticação.

### Configuração do Cliente HTTP

Instanciação do cliente Axios com configurações base (BaseURL, Timeouts) e acoplamento de interceptadores.

**Ficheiro:** `src/shared/api/client/axios.config.ts`

```ts
import axios, { AxiosInstance } from "axios";
import { authInterceptor } from "./interceptors/auth.interceptor";
import { errorInterceptor } from "./interceptors/error.interceptor";
import { retryInterceptor } from "./interceptors/retry.interceptor";

const createUserApiClient = (): AxiosInstance => {
  const client = axios.create({
    baseURL: import.meta.env.VITE_API_URL || "http://localhost:3000/user/api",
    timeout: 10000,
    headers: {
      "Content-Type": "application/json",
    },
  });

  // Interceptors de Request
  client.interceptors.request.use(
    authInterceptor.onRequest,
    authInterceptor.onRequestError
  );

  // Interceptors de Response (erro e retry)
  client.interceptors.response.use(
    undefined,
    errorInterceptor.onResponseError
  );

  return client;
};

export const userApiClient = createUserApiClient();
```

> **Nota:** Os interceptores `errorInterceptor` e `retryInterceptor` são importados mas apenas o `authInterceptor` é registado no exemplo acima. Garantir que todos os interceptores necessários são acoplados ao cliente antes de usar em produção.

### Interceptadores (Interceptors)

Middleware para injeção de tokens, logging e tratamento padronizado de erros.

**Ficheiro:** `src/shared/api/client/interceptors/auth.interceptor.ts`

```ts
import { InternalAxiosRequestConfig, AxiosError } from "axios";
import { authService } from "@/infra/auth/auth.service";

export const authInterceptor = {
  onRequest: (config: InternalAxiosRequestConfig) => {
    const token = authService.getAccessToken();

    if (token && config.headers) {
      config.headers.Authorization = `Bearer ${token}`;
    }

    // Adiciona Correlation ID para rastreamento distribuído
    config.headers["X-Correlation-Id"] = crypto.randomUUID();

    return config;
  },

  onRequestError: (error: AxiosError) => {
    console.error("Request error:", error);
    return Promise.reject(error);
  },
};
```

> **Nota de tipagem:** `AxiosRequestConfig` foi substituído por `InternalAxiosRequestConfig` — tipo correto para o callback de interceptor de request a partir do Axios 1.x. O uso de `AxiosRequestConfig` neste contexto causaria erro de tipagem TypeScript.

### Serviços de Domínio (Service Layer)

Classes ou objetos que encapsulam as chamadas de rede, tipando estritamente entradas e saídas.

**Ficheiro:** `src/shared/api/services/order.service.ts`

```ts
import { userApiClient } from "../client";
import type {
  Order,
  CreateOrderDTO,
  ApiResponse,
  PaginatedResponse,
} from "@/shared/types";
import { OrderStatus } from "@/shared/types/models/order.types";

class OrderService {
  private readonly basePath = "/orders";

  async getOrders(params: {
    page?: number;
    pageSize?: number;
    status?: string;
  }): Promise<PaginatedResponse<Order>> {
    const { data } = await userApiClient.get<PaginatedResponse<Order>>(
      this.basePath,
      { params }
    );
    return data;
  }

  async getOrderById(id: string): Promise<Order> {
    const { data } = await userApiClient.get<ApiResponse<Order>>(
      `${this.basePath}/${id}`
    );
    return data.data;
  }

  async createOrder(orderData: CreateOrderDTO): Promise<Order> {
    const { data } = await userApiClient.post<ApiResponse<Order>>(
      this.basePath,
      orderData
    );
    return data.data;
  }

  async updateOrderStatus(id: string, status: OrderStatus): Promise<Order> {
    const { data } = await userApiClient.patch<ApiResponse<Order>>(
      `${this.basePath}/${id}/status`,
      { status }
    );
    return data.data;
  }

  async cancelOrder(id: string): Promise<void> {
    await userApiClient.delete(`${this.basePath}/${id}`);
  }
}

export const orderService = new OrderService();
```

> **Nota:** O tipo `OrderStatus` utilizado em `updateOrderStatus` deve ser importado explicitamente — não está disponível automaticamente no escopo da classe. O import foi adicionado acima.

---

## Implementação de Feature (Exemplo: Orders)

Demonstração prática da aplicação dos princípios num módulo de negócio.

### Gestão de Estado de Servidor (React Query)

**Ficheiro:** `src/features/orders/user-api/orders.queries.ts`

```ts
import { useQuery } from "@tanstack/react-query";
import { orderService } from "@/shared/api/services";

// Query Keys — factory pattern para consistência e type-safety
export const orderKeys = {
  all: ["orders"] as const,
  lists: () => [...orderKeys.all, "list"] as const,
  list: (filters: Record<string, unknown>) =>
    [...orderKeys.lists(), filters] as const,
  details: () => [...orderKeys.all, "detail"] as const,
  detail: (id: string) => [...orderKeys.details(), id] as const,
};

// Hook para listar pedidos com paginação
export const useOrders = (page = 1, pageSize = 10, status?: string) => {
  return useQuery({
    queryKey: orderKeys.list({ page, pageSize, status }),
    queryFn: () => orderService.getOrders({ page, pageSize, status }),
    staleTime: 5 * 60 * 1000,  // 5 minutos
    gcTime: 10 * 60 * 1000,    // 10 minutos (antigo cacheTime)
  });
};

// Hook para buscar um pedido específico
export const useOrder = (id: string) => {
  return useQuery({
    queryKey: orderKeys.detail(id),
    queryFn: () => orderService.getOrderById(id),
    enabled: !!id,
  });
};
```

> **Nota:** O tipo `any` no parâmetro de `list` foi substituído por `Record<string, unknown>` para manter type-safety. Evitar `any` em query keys — queries com chaves mal tipadas causam cache pollution difícil de rastrear.

**Ficheiro:** `src/features/orders/user-api/orders.mutations.ts`

```ts
import { useMutation, useQueryClient } from "@tanstack/react-query";
import { orderService } from "@/shared/api/services";
import type { CreateOrderDTO } from "@/shared/types";
import { toast } from "@/shared/components/feedback/Toast";
import { orderKeys } from "./orders.queries";

export const useCreateOrder = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (data: CreateOrderDTO) => orderService.createOrder(data),
    onMutate: async (_newOrder) => {
      // Cancelar queries em curso para evitar sobrescrever o optimistic update
      await queryClient.cancelQueries({ queryKey: orderKeys.lists() });
    },
    onError: (_err, _newOrder, _context) => {
      toast.error("Erro ao criar pedido.");
    },
    onSuccess: () => {
      // Invalida cache para forçar refetch da lista
      queryClient.invalidateQueries({ queryKey: orderKeys.lists() });
      toast.success("Pedido criado com sucesso!");
    },
  });
};
```

> **Notas:**
> - O import de `useQuery` foi removido — este ficheiro contém apenas mutations. Imports não utilizados devem ser eliminados.
> - Os parâmetros não utilizados nos callbacks (`err`, `newOrder`, `context`) foram prefixados com `_` para suprimir avisos do TypeScript sem remover a assinatura.
> - O `onMutate` foi completado com `cancelQueries` — padrão recomendado pelo TanStack Query para evitar race conditions.

### Componentes da Feature

Aqui são declarados os componentes da respetiva feature (orders), as interfaces, os seus comportamentos e chamadas.

**Ficheiro:** `src/features/orders/pages/OrdersPage.tsx`

```tsx
import React, { useState } from "react";
import { useNavigate } from "react-router-dom";
import { useOrders } from "../user-api/orders.queries";
import { OrderList } from "../components/OrderList";
import { OrderFilters } from "../components/OrderFilters";
import { Pagination } from "@/shared/components/ui/Pagination";
import { Button } from "@/shared/components/ui/Button";
import { PlusIcon } from "@/shared/components/icons";

export const OrdersPage: React.FC = () => {
  const navigate = useNavigate();
  const [page, setPage] = useState(1);
  const [status, setStatus] = useState<string>("");

  const { data, isLoading, error } = useOrders(page, 10, status);

  if (error) {
    return (
      <div role="alert" className="alert alert-error">
        Erro ao carregar pedidos. Por favor, tente novamente.
      </div>
    );
  }

  return (
    <div className="container mx-auto p-4">
      <div className="flex justify-between items-center mb-6">
        <h1 className="text-3xl font-bold">Meus Pedidos</h1>
        <Button
          onClick={() => navigate("/orders/new")}
          leftIcon={<PlusIcon />}
          aria-label="Criar novo pedido"
        >
          Novo Pedido
        </Button>
      </div>

      <OrderFilters onStatusChange={setStatus} currentStatus={status} />

      <OrderList
        orders={data?.items || []}
        isLoading={isLoading}
        onOrderClick={(order) => navigate(`/orders/${order.id}`)}
      />

      {data && data.total > 10 && (
        <Pagination
          currentPage={page}
          totalPages={Math.ceil(data.total / 10)}
          onPageChange={setPage}
        />
      )}
    </div>
  );
};
```

> **Acessibilidade:** `role="alert"` adicionado no bloco de erro para que leitores de ecrã anunciem a mensagem imediatamente. `aria-label` adicionado no botão com ícone para garantir descrição semântica adequada.

**Ficheiro:** `src/features/orders/components/OrderList/OrderList.tsx`

```tsx
import React, { memo } from "react";
import { OrderCard } from "../OrderCard";
import { LoadingState } from "@/shared/components/feedback/LoadingState";
import { EmptyState } from "@/shared/components/feedback/EmptyState";
import { ErrorBoundary } from "@/shared/components/feedback/ErrorBoundary";
import type { Order } from "@/shared/types";

interface OrderListProps {
  orders: Order[];
  isLoading?: boolean;
  onOrderClick?: (order: Order) => void;
}

export const OrderList = memo<OrderListProps>(
  ({ orders, isLoading, onOrderClick }) => {
    if (isLoading) {
      return <LoadingState message="A carregar pedidos..." />;
    }

    if (!orders || orders.length === 0) {
      return (
        <EmptyState
          title="Nenhum pedido encontrado"
          description="Ainda não existem pedidos realizados."
          action={{
            label: "Fazer primeiro pedido",
            onClick: () => navigate("/products"),
          }}
        />
      );
    }

    return (
      <ErrorBoundary fallback={<p role="alert">Erro ao carregar lista de pedidos.</p>}>
        <div
          className="grid gap-4 md:grid-cols-2 lg:grid-cols-3"
          role="list"
          aria-label="Lista de pedidos"
        >
          {orders.map((order) => (
            <OrderCard
              key={order.id}
              order={order}
              onClick={() => onOrderClick?.(order)}
            />
          ))}
        </div>
      </ErrorBoundary>
    );
  }
);

OrderList.displayName = "OrderList";
```

> **Notas:**
> - `window.location.href` substituído por `navigate("/products")` — a navegação imperativa deve usar o router para preservar o histórico e evitar reload completo da página.
> - `role="list"` e `aria-label` adicionados na grid de pedidos para semântica de lista acessível.
> - O `fallback` do `ErrorBoundary` foi convertido de string para elemento JSX com `role="alert"`.

---

## Gestão de Estado Global (Zustand)

Implementação de *stores* para estados de cliente que necessitam ser acessíveis globalmente.

**Ficheiro:** `src/shared/store/zustand/orders.store.ts`

```ts
import { create } from "zustand";
import { devtools, persist } from "zustand/middleware";
import type { Order } from "@/shared/types";

interface OrdersState {
  // State
  selectedOrder: Order | null;
  filters: {
    status: string;
    dateRange: [Date | null, Date | null];
    search: string;
  };

  // Actions
  setSelectedOrder: (order: Order | null) => void;
  updateFilters: (filters: Partial<OrdersState["filters"]>) => void;
  resetFilters: () => void;
}

const initialFilters: OrdersState["filters"] = {
  status: "",
  dateRange: [null, null],
  search: "",
};

export const useOrdersStore = create<OrdersState>()(
  devtools(
    persist(
      (set) => ({
        // State
        selectedOrder: null,
        filters: initialFilters,

        // Actions
        setSelectedOrder: (order) =>
          set({ selectedOrder: order }, false, "setSelectedOrder"),

        updateFilters: (filters) =>
          set(
            (state) => ({
              filters: { ...state.filters, ...filters },
            }),
            false,
            "updateFilters"
          ),

        resetFilters: () =>
          set({ filters: initialFilters }, false, "resetFilters"),
      }),
      {
        name: "orders-storage",
        partialize: (state) => ({ filters: state.filters }),
      }
    )
  )
);
```

> **Nota:** O tipo de `initialFilters` foi corrigido — o cast `as [null, null]` foi removido e o tipo explícito `OrdersState["filters"]` foi adicionado à constante, o que é mais seguro e legível.

---

## Validação e Formulários (Zod + React Hook Form)

Utilização de Zod para definição de esquemas (*Single Source of Truth* para validação).

**Ficheiro:** `src/features/orders/schemas/order.schema.ts`

```ts
import { z } from "zod";

export const createOrderItemSchema = z.object({
  productId: z.string().uuid("ID do produto inválido"),
  quantity: z
    .number()
    .min(1, "Quantidade deve ser maior que zero")
    .max(100, "Quantidade máxima excedida"),
  unitPrice: z.number().positive("Preço deve ser positivo"),
});

export const createOrderSchema = z.object({
  customerId: z.string().uuid("ID do cliente inválido"),
  items: z
    .array(createOrderItemSchema)
    .min(1, "Pedido deve ter pelo menos um item")
    .max(50, "Limite de itens por pedido excedido"),
  deliveryAddress: z.object({
    street: z.string().min(3, "Endereço muito curto"),
    number: z.string().min(1, "Número obrigatório"),
    complement: z.string().optional(),
    city: z.string().min(2, "Cidade obrigatória"),
    state: z.string().length(2, "Estado deve ter 2 caracteres"),
    zipCode: z.string().regex(/^\d{8}$/, "CEP inválido"),
  }),
  paymentMethod: z.enum(["CREDIT_CARD", "PIX", "BOLETO"]),
  notes: z.string().max(500, "Observações muito longas").optional(),
});

export type CreateOrderInput = z.infer<typeof createOrderSchema>;
export type CreateOrderItemInput = z.infer<typeof createOrderItemSchema>;
```

**Ficheiro:** `src/features/orders/components/OrderForm/OrderForm.tsx`

```tsx
import React from "react";
import { useForm, useFieldArray } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import {
  createOrderSchema,
  type CreateOrderInput,
} from "../../schemas/order.schema";
import { useCreateOrder } from "../../user-api/orders.mutations";
import { Button } from "@/shared/components/ui/Button";
import { Input } from "@/shared/components/ui/Input";
import { Select } from "@/shared/components/ui/Select";

export const OrderForm: React.FC = () => {
  const createOrder = useCreateOrder();

  const {
    register,
    control,
    handleSubmit,
    formState: { errors, isSubmitting },
    reset,
  } = useForm<CreateOrderInput>({
    resolver: zodResolver(createOrderSchema),
    defaultValues: {
      items: [{ productId: "", quantity: 1, unitPrice: 0 }],
    },
  });

  const { fields, append, remove } = useFieldArray({
    control,
    name: "items",
  });

  const onSubmit = async (data: CreateOrderInput) => {
    try {
      await createOrder.mutateAsync(data);
      reset();
    } catch (error) {
      // Erro já tratado pelo onError da mutation (toast)
      console.error("Erro ao criar pedido:", error);
    }
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="space-y-6" noValidate>
      {/* Customer ID */}
      <div>
        <label htmlFor="customerId" className="block text-sm font-medium mb-1">
          ID do Cliente
        </label>
        <Input
          id="customerId"
          {...register("customerId")}
          error={errors.customerId?.message}
          placeholder="Digite o ID do cliente"
          aria-describedby={errors.customerId ? "customerId-error" : undefined}
        />
        {errors.customerId && (
          <span id="customerId-error" role="alert" className="text-sm text-red-600">
            {errors.customerId.message}
          </span>
        )}
      </div>

      {/* Items */}
      <div>
        <div className="flex justify-between items-center mb-2">
          <label className="block text-sm font-medium">Itens do Pedido</label>
          <Button
            type="button"
            variant="secondary"
            size="sm"
            onClick={() => append({ productId: "", quantity: 1, unitPrice: 0 })}
          >
            Adicionar Item
          </Button>
        </div>

        {fields.map((field, index) => (
          <div key={field.id} className="flex gap-2 mb-2">
            <Input
              {...register(`items.${index}.productId`)}
              placeholder="ID do Produto"
              error={errors.items?.[index]?.productId?.message}
              aria-label={`ID do produto, item ${index + 1}`}
            />
            <Input
              {...register(`items.${index}.quantity`, { valueAsNumber: true })}
              type="number"
              placeholder="Qtd"
              error={errors.items?.[index]?.quantity?.message}
              aria-label={`Quantidade, item ${index + 1}`}
            />
            <Button
              type="button"
              variant="danger"
              size="sm"
              onClick={() => remove(index)}
              disabled={fields.length === 1}
              aria-label={`Remover item ${index + 1}`}
            >
              Remover
            </Button>
          </div>
        ))}
      </div>

      {/* Payment Method */}
      <div>
        <label htmlFor="paymentMethod" className="block text-sm font-medium mb-1">
          Método de Pagamento
        </label>
        <Select
          id="paymentMethod"
          {...register("paymentMethod")}
          error={errors.paymentMethod?.message}
        >
          <option value="">Selecione...</option>
          <option value="CREDIT_CARD">Cartão de Crédito</option>
          <option value="PIX">PIX</option>
          <option value="BOLETO">Boleto</option>
        </Select>
      </div>

      {/* Submit */}
      <div className="flex gap-2">
        <Button type="submit" isLoading={isSubmitting || createOrder.isPending}>
          Criar Pedido
        </Button>
        <Button type="button" variant="secondary" onClick={() => reset()}>
          Limpar
        </Button>
      </div>
    </form>
  );
};
```

> **Notas:**
> - Erro de sintaxe corrigido: `{...register(`items[${index}].productId`)}` → `` `items.${index}.productId` `` — React Hook Form usa notação de ponto, não de colchetes, para `useFieldArray`.
> - O caminho do import de `useCreateOrder` foi corrigido de `../../userApi/orders.mutation` para `../../user-api/orders.mutations` (consistente com o restante do projeto).
> - `label` com `htmlFor` e `id` adicionados nos campos de formulário para associação semântica correta (acessibilidade).
> - `aria-label` adicionado nos inputs dinâmicos de itens (sem label visível associada).
> - `noValidate` adicionado no `<form>` para desativar validação nativa do browser e delegar inteiramente ao Zod/RHF.

---

## Roteamento e Guards de Autenticação

### Estrutura de Rotas e Lazy Loading

A configuração do router deve utilizar *Dynamic Imports* (Lazy Loading) para *Code Splitting*, garantindo que o JavaScript seja carregado apenas quando necessário.

**Ficheiro:** `src/app/router/AppRouter.tsx`

```tsx
import { Routes, Route, Navigate } from "react-router-dom";
import { lazy, Suspense } from "react";
import { AuthGuard } from "./guards/AuthGuard";
import { PublicRoute } from "./guards/PublicRoute";
import { RoleGuard } from "./guards/RoleGuard";
import { MainLayout } from "../layouts/MainLayout";
import { AuthLayout } from "../layouts/AuthLayout";
import { LoadingScreen } from "@/shared/components/feedback/LoadingScreen";

// Lazy loading das páginas
const LoginPage = lazy(() => import("@/features/auth/pages/LoginPage"));
const RegisterPage = lazy(() => import("@/features/auth/pages/RegisterPage"));
const ForgotPasswordPage = lazy(
  () => import("@/features/auth/pages/ForgotPasswordPage")
);
const DashboardPage = lazy(
  () => import("@/features/dashboard/pages/DashboardPage")
);
const OrdersPage = lazy(() => import("@/features/orders/pages/OrdersPage"));
const OrderDetailsPage = lazy(
  () => import("@/features/orders/pages/OrderDetailsPage")
);
const CreateOrderPage = lazy(
  () => import("@/features/orders/pages/CreateOrderPage")
);
const ProfilePage = lazy(() => import("@/features/profile/pages/ProfilePage"));
const SettingsPage = lazy(() => import("@/features/settings/pages/SettingsPage"));
const AdminDashboard = lazy(
  () => import("@/features/admin/pages/AdminDashboard")
);
const UsersManagement = lazy(
  () => import("@/features/admin/pages/UsersManagement")
);
const ReportsPage = lazy(() => import("@/features/admin/pages/ReportsPage"));
const NotFoundPage = lazy(() => import("@/shared/pages/NotFoundPage"));

export const AppRouter = () => {
  return (
    <Suspense fallback={<LoadingScreen />}>
      <Routes>
        {/* Rotas Públicas */}
        <Route element={<PublicRoute />}>
          <Route element={<AuthLayout />}>
            <Route path="/login" element={<LoginPage />} />
            <Route path="/register" element={<RegisterPage />} />
            <Route path="/forgot-password" element={<ForgotPasswordPage />} />
          </Route>
        </Route>

        {/* Rotas Protegidas */}
        <Route element={<AuthGuard />}>
          <Route element={<MainLayout />}>
            <Route path="/" element={<Navigate to="/dashboard" replace />} />
            <Route path="/dashboard" element={<DashboardPage />} />

            {/* Rotas de Pedidos */}
            <Route path="/orders">
              <Route index element={<OrdersPage />} />
              <Route path=":orderId" element={<OrderDetailsPage />} />
              <Route path="new" element={<CreateOrderPage />} />
            </Route>

            {/* Rotas de Perfil */}
            <Route path="/profile" element={<ProfilePage />} />
            <Route path="/settings" element={<SettingsPage />} />

            {/* Rotas com Permissões Específicas */}
            <Route element={<RoleGuard allowedRoles={["admin", "manager"]} />}>
              <Route path="/admin">
                <Route index element={<AdminDashboard />} />
                <Route path="users" element={<UsersManagement />} />
                <Route path="reports" element={<ReportsPage />} />
              </Route>
            </Route>
          </Route>
        </Route>

        {/* 404 */}
        <Route path="*" element={<NotFoundPage />} />
      </Routes>
    </Suspense>
  );
};
```

> **Notas:**
> - Todos os componentes usados em `<Route element={...}>` que estavam referenciados mas sem `lazy()` declarado foram adicionados (`ForgotPasswordPage`, `CreateOrderPage`, `SettingsPage`, `AdminDashboard`, `UsersManagement`, `ReportsPage`). Componentes não importados causam erro de compilação.
> - `RoleGuard` foi adicionado aos imports — estava a ser usado mas não importado.

### Route Guards

Middlewares de proteção de rota que verificam o estado de autenticação antes de renderizar componentes filhos.

**Ficheiro:** `src/app/router/guards/AuthGuard.tsx`

```tsx
import { Navigate, Outlet, useLocation } from "react-router-dom";
import { useAuthStore } from "@/shared/stores/auth.store";

export const AuthGuard = () => {
  const location = useLocation();
  const { isAuthenticated, checkAuth } = useAuthStore();

  if (!checkAuth() || !isAuthenticated) {
    return <Navigate to="/login" state={{ from: location.pathname }} replace />;
  }

  return <Outlet />;
};
```

> **Nota:** O nome do ficheiro foi corrigido de `PermissionGuard.tsx` para `AuthGuard.tsx` — o componente exportado chama-se `AuthGuard` e o ficheiro deve ter o mesmo nome para consistência e para que os imports automáticos funcionem corretamente.

---

## Estratégia de Testes

### Testes Unitários (Lógica Pura)

Focados em funções utilitárias, hooks e regras de negócio isoladas.

**Ficheiro:** `src/shared/utils/formatters/currency.test.ts`

```ts
import { describe, it, expect } from "vitest";
import { formatCurrency } from "./currency";

describe("formatCurrency", () => {
  it("should format BRL currency correctly", () => {
    expect(formatCurrency(1234.56, "BRL")).toBe("R$ 1.234,56");
  });

  it("should handle zero values", () => {
    expect(formatCurrency(0, "BRL")).toBe("R$ 0,00");
  });

  it("should handle negative values", () => {
    expect(formatCurrency(-100, "BRL")).toBe("-R$ 100,00");
  });
});
```

> **Nota:** A extensão do ficheiro foi corrigida de `.test.tsx` para `.test.ts` — este ficheiro não contém JSX, pelo que `.tsx` é desnecessário e pode gerar avisos de linting.

### Testes de Componentes (Interação)

Focados no comportamento do utilizador e acessibilidade, utilizando *Testing Library*.

**Ficheiro:** `src/shared/components/ui/Button/Button.test.tsx`

```ts
import { render, screen, fireEvent } from "@testing-library/react";
import { vi } from "vitest";
import { Button } from "./Button";

describe("Button", () => {
  it("renders correctly", () => {
    render(<Button>Click me</Button>);
    expect(screen.getByRole("button", { name: "Click me" })).toBeInTheDocument();
  });

  it("handles click events", () => {
    const handleClick = vi.fn();
    render(<Button onClick={handleClick}>Click me</Button>);

    fireEvent.click(screen.getByRole("button", { name: "Click me" }));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it("shows loading state and disables button", () => {
    render(<Button isLoading>Submit</Button>);
    expect(screen.getByRole("button")).toBeDisabled();
  });

  it("is accessible with screen readers", () => {
    render(<Button aria-label="Criar pedido">+</Button>);
    expect(
      screen.getByRole("button", { name: "Criar pedido" })
    ).toBeInTheDocument();
  });
});
```

> **Notas:**
> - `screen.getByText("Click me")` substituído por `screen.getByRole("button", { name: "Click me" })` — query mais robusta e alinhada com boas práticas de Testing Library (queries por role são preferidas).
> - Adicionado teste de acessibilidade com `aria-label` para validar componentes com ícones ou labels não textuais.

---

## Performance por Defeito

A performance não é uma etapa de otimização tardia — é um requisito funcional contínuo.

### Diretrizes Mandatórias

**Eliminação de Efeitos Colaterais no Render:**

Proibido realizar processamento pesado, *parsing* de dados ou formatações complexas dentro de `useEffect` apenas para derivar estado.

```ts
// ❌ Não fazer: useEffect desnecessário para derivar estado
useEffect(() => {
  const sorted = items.sort((a, b) => a.date.localeCompare(b.date));
  setSorted(sorted);
}, [items]);
```

```ts
// ✅ Correto: derivar diretamente no render com useMemo
const sorted = useMemo(
  () => [...items].sort((a, b) => a.date.localeCompare(b.date)),
  [items]
);
```

> **Atenção:** Nunca mutar o array original (`items.sort(...)` muta in-place). Usar sempre spread (`[...items]`) antes de ordenar.

**Gestão de Eventos de Alta Frequência:**

Eventos como `onChange`, `onScroll` ou `onResize` devem utilizar técnicas de *Debounce* ou *Throttle*. Evitar armazenar cada *keystroke* no estado global (Redux/Context), preferindo estado local ou *uncontrolled components*.

**Memoização:**

- `React.memo`: utilizar em componentes de apresentação puros que recebem props primitivas.
- `useCallback`: utilizar estritamente para funções passadas como props para filhos memoizados.
- `useMemo`: utilizar quando o custo computacional da derivação justificar (evitar uso excessivo — memoização tem custo próprio).

**Carregamento Assíncrono (Lazy Loading):**

```ts
// Segmentação de bundles por rota e componentes pesados
const OrdersPage = lazy(() => import("@/features/orders/pages/OrdersPage"));
const HeavyChart = lazy(() => import("@/shared/components/charts/HeavyChart"));
```

Aplicar a componentes pesados como gráficos, mapas e tabelas com grande volume de dados.

### Anti-Patterns de Performance

- Chamadas de API (`fetch`) manuais dentro de `useEffect` — usar TanStack Query.
- Duplicação de estado (manter a mesma informação sincronizada em servidor e cliente sem necessidade).
- Definição de objetos, arrays ou funções literais dentro do render de componentes que são dependências de `useEffect` (causam re-renders infinitos).

---

## Estado Previsível, Rastreável e Sem Prop Drilling

O estado deve ser fácil de entender, fácil de rastrear e impossível de manter duplicado.

### Eliminação de Prop Drilling

O repasse de props através de múltiplos níveis de componentes (*Prop Drilling*) indica uma falha de design ou necessidade de composição.

**Cenário inadequado (alto acoplamento):**

```tsx
// ❌ Prop drilling — cria acoplamento e re-renders em cascata
<Home user={user} />
// Home → Dashboard → Sidebar → UserAvatar (todos recebem e repassam user)
```

**Solução arquitetural (Context Composition):**

**1. Definição do Contexto:**

```tsx
import { createContext, useContext, useState, ReactNode } from "react";

type User = { id: string; name: string };

const UserContext = createContext<User | null>(null);

export const UserProvider = ({ children }: { children: ReactNode }) => {
  const [user] = useState<User>({ id: "1", name: "Matheus" });

  return <UserContext.Provider value={user}>{children}</UserContext.Provider>;
};

// Hook para consumir de forma limpa — lança erro se usado fora do Provider
export const useUser = () => {
  const context = useContext(UserContext);
  if (context === null) {
    throw new Error("useUser deve ser usado dentro de <UserProvider>");
  }
  return context;
};
```

**2. Provider registado apenas no nível raiz:**

```tsx
// src/app/providers/index.tsx
export function AppProviders({ children }: { children: ReactNode }) {
  return <UserProvider>{children}</UserProvider>;
}
```

**3. Consumo desacoplado:**

```tsx
// src/features/dashboard/components/UserInfo.tsx
import { useUser } from "@/shared/context/UserContext";

export function UserInfo() {
  const user = useUser();
  return <span>Bem-vindo, {user.name}</span>;
}

// src/app/pages/DashboardPage.tsx
import { UserInfo } from "@/features/dashboard/components/UserInfo";

export function DashboardPage() {
  return (
    <div>
      <h1>Dashboard</h1>
      <UserInfo /> {/* acede o utilizador sem passar props */}
    </div>
  );
}
```

> **Nota:** O hook `useUser` foi reforçado com verificação de contexto nulo e lançamento de erro descritivo — padrão recomendado para detetar usos fora do Provider em tempo de desenvolvimento, em vez de falhar silenciosamente com `null`.
