Índice
------

- [Histórico de Revisões](#histórico-de-revisões)
- [Ideia](#ideia)
    - [Exemplo na solução](#exemplo-na-solução)
- [**MVVM**](#mvvm)
- [**Princípio Offline First**](#princípio-offline-first)
    - [**Conceito de Offline-First**](#conceito-de-offline-first)
    - [**Benefícios para React Native**](#benefícios-para-react-native)
    - [**Componentes Essenciais da Arquitetura Offline-First**](#componentes-essenciais-da-arquitetura-offline-first)
        - [**Base de dados local**](#base-de-dados-local)
    - [**Camada de acesso a dados (repositories)**](#camada-de-acesso-a-dados-repositories)
    - [**Fila de operações pendentes (Pending Queue)**](#fila-de-operações-pendentes-pending-queue)
    - [**Serviço de sincronização**](#serviço-de-sincronização)
    - [**Detecção de rede**](#detecção-de-rede)
    - [**Fluxo Operacional do Offline-First**](#fluxo-operacional-do-offline-first)
    - [**Estratégias de Resolução de Conflitos**](#estratégias-de-resolução-de-conflitos)
    - [**Boas Práticas**](#boas-práticas)
    - [**Integração com Apollo Client**](#integração-com-apollo-client)
    - [Exemplos de código](#exemplos-de-código)
- [**GraphQL**](#graphql)
    - [HotChocolate.AspNetCore](#hotchocolateaspnetcore)
    - [Arquitetura do GraphQL no Backend](#arquitetura-do-graphql-no-backend)
    - [Configuração Básica do Servidor](#configuração-básica-do-servidor)
    - [Queries (Leituras)](#queries-leituras)
    - [Mutations (Escritas)](#mutations-escritas)
    - [Types (Tipos GraphQL)](#types-tipos-graphql)
    - [Autorização com HotChocolate](#autorização-com-hotchocolate)
    - [Integração com Entity Framework Core](#integração-com-entity-framework-core)
    - [Ferramentas de Teste e Playground](#ferramentas-de-teste-e-playground)
    - [Considerações de Segurança](#considerações-de-segurança)
- [Event Driven Architecture](#event-driven-architecture)
    - [**Visão Geral**](#visão-geral)
    - [**Conceitos Fundamentais**](#conceitos-fundamentais)
        - [**Evento**](#evento)
        - [**Produtor de Eventos**](#produtor-de-eventos)
        - [**Consumidor de Eventos**](#consumidor-de-eventos)
        - [**Event Broker / Event Bus**](#event-broker-event-bus)
        - [**Fluxo Básico de Funcionamento**](#fluxo-básico-de-funcionamento)
        - [**Tipos de Arquitetura Orientada a Eventos**](#tipos-de-arquitetura-orientada-a-eventos)
            - [**Event Notification**](#event-notification)
            - [**Event-Carried State Transfer**](#event-carried-state-transfer)
            - [**Event Sourcing**](#event-sourcing)
        - [**Benefícios da EDA**](#benefícios-da-eda)
        - [**Desafios e Cuidados**](#desafios-e-cuidados)
            - [**Consistência Eventual**](#consistência-eventual)
            - [**Idempotência**](#idempotência)
            - [**Ordenação de Eventos**](#ordenação-de-eventos)
            - [**Versionamento de Eventos**](#versionamento-de-eventos)
        - [**EDA e Microserviços**](#eda-e-microserviços)
        - [**Quando Usar Event-Driven Architecture**](#quando-usar-event-driven-architecture)
        - [**Conclusão**](#conclusão)
- [Processamento Assíncrono no Frontend Web](#processamento-assíncrono-no-frontend-web)
    - [**Exemplos em .Net Blazor:**](#exemplos-em-net-blazor)
- [**Client-side Composition**](#client-side-composition)
    - [**Exemplo:**](#exemplo)
- [**Modular Monolith**](#modular-monolith)
    - [Exemplo:](#exemplo-1)
- [**Clean Architecture**](#clean-architecture)
    - [**Visão Geral**](#visão-geral-1)
    - [**Princípio Fundamental**](#princípio-fundamental)
    - [**Estrutura em Camadas**](#estrutura-em-camadas)
    - [**Entities (Domínio)**](#entities-domínio)
        - [**Responsabilidade**](#responsabilidade)
    - [**Use Cases (Application Layer)**](#use-cases-application-layer)
        - [**Responsabilidade**](#responsabilidade-1)
    - [**Interface Adapters**](#interface-adapters)
        - [**Responsabilidade**](#responsabilidade-2)
    - [**Frameworks & Drivers**](#frameworks-drivers)
        - [**Responsabilidade**](#responsabilidade-3)
    - [**Inversão de Dependência**](#inversão-de-dependência)
    - [**Clean Architecture vs MVC Tradicional**](#clean-architecture-vs-mvc-tradicional)
    - [**Clean Architecture e Hexagonal Architecture**](#clean-architecture-e-hexagonal-architecture)
    - [**Clean Architecture e Microserviços**](#clean-architecture-e-microserviços)
    - [**Benefícios da Clean Architecture**](#benefícios-da-clean-architecture)
    - [**Desafios e Armadilhas**](#desafios-e-armadilhas)
        - [**Overengineering**](#overengineering)
        - [**Curva de Aprendizado**](#curva-de-aprendizado)
        - [**Excesso de Abstrações**](#excesso-de-abstrações)
    - [**Quando Usar Clean Architecture**](#quando-usar-clean-architecture)
    - [**Conclusão**](#conclusão-1)
- [**CQRS (Command Query Responsibility Segregation)**](#cqrs-command-query-responsibility-segregation)
    - [**Visão Geral**](#visão-geral-2)
    - [**Conceitos Fundamentais**](#conceitos-fundamentais-1)
        - [**Command**](#command)
        - [**Query**](#query)
        - [**Command Model (Write Model)**](#command-model-write-model)
        - [**Query Model (Read Model)**](#query-model-read-model)
    - [**Fluxo Básico de CQRS**](#fluxo-básico-de-cqrs)
    - [**CQRS Simples vs CQRS Completo**](#cqrs-simples-vs-cqrs-completo)
        - [**CQRS Simples**](#cqrs-simples)
        - [**CQRS Completo**](#cqrs-completo)
    - [**CQRS e Event-Driven Architecture**](#cqrs-e-event-driven-architecture)
    - [**CQRS e Event Sourcing**](#cqrs-e-event-sourcing)
    - [**Benefícios do CQRS**](#benefícios-do-cqrs)
    - [**Desafios e Armadilhas**](#desafios-e-armadilhas-1)
        - [**Complexidade**](#complexidade)
        - [**Consistência Eventual**](#consistência-eventual-1)
        - [**Overengineering**](#overengineering-1)
    - [**CQRS vs CRUD Tradicional**](#cqrs-vs-crud-tradicional)
    - [**CQRS e Clean Architecture**](#cqrs-e-clean-architecture)
    - [**Quando Usar CQRS**](#quando-usar-cqrs)
    - [**Exemplo de Uso Prático**](#exemplo-de-uso-prático)
    - [**Conclusão**](#conclusão-2)
- [**Design for Failure**](#design-for-failure)
- [**Infraestrutura como Código (IaC)**](#infraestrutura-como-código-iac)

# Histórico de Revisões

| **Versão** | **Data**   | **Autor(es)** | **Resumo das Mudanças** |
| ---------- | ---------- | ------------- | ----------------------- |
| 1.0        | 19/12/2025 | William Alves | Criação do documento.   |



```adf 
{"type":"extension","attrs":{"layout":"default","extensionType":"com.atlassian.confluence.macro.core","extensionKey":"toc","parameters":{"macroParams":{"minLevel":{"value":"1"},"maxLevel":{"value":"6"},"include":{"value":""},"outline":{"value":"false"},"indent":{"value":""},"style":{"value":"none"},"exclude":{"value":"Histórico de Revisões"},"type":{"value":"list"},"class":{"value":""},"printable":{"value":"true"}},"macroMetadata":{"macroId":{"value":"4ac9a4fc-3efc-4997-b403-83d5616496fa"},"schemaVersion":{"value":"1"},"title":"Table of Contents"}},"localId":"f0580f92-f6d0-4988-8bd6-666812ab7552"}}
```

```adf 
{"type":"heading","attrs":{"level":1,"localId":"1a0509a3-78f7-4926-9ceb-0352c857fe2b"},"content":[{"text":"Backend-For-Frontend(BFF)","type":"text","marks":[{"type":"annotation","attrs":{"annotationType":"inlineComment","id":"4a32556c-bee2-4e25-b3c2-1ebb95856546"}}]}]}
```


# Ideia

Um backend feito sob medida para um frontend específico.

Ele não é o “grande backend de negócio”, mas sim uma construção mais simples que:

- simplifica contratos,
- agrega dados de integrações (lógica de agregação),
- aplica regras de segurança/validação pensando na UI.

Ele expõe rotas:

- /api/catalog/...
- /api/orders/...

e, esconde qualquer complexidade dos dados (como acesso ao Retek, ou, autenticação/autorização em diferentes IDPs).

## Exemplo na solução

Arquivo: Bff/BackendServices/CatalogBackendService.cs

``` 
using System.Net.Http.Json;
using Microsoft.Extensions.Options;
using Shared.Dtos;
namespace Bff.BackendServices;
public interface ICatalogBackendService
{
    Task<IReadOnlyList<CatalogItemDto>> GetCatalogAsync(CancellationToken ct = default);
}
public sealed class CatalogBackendService : ICatalogBackendService
{
    private readonly HttpClient _http;
    private readonly CatalogApiOptions _options;
    public CatalogBackendService(HttpClient http, IOptions<CatalogApiOptions> options)
    {
        _http = http;
        _options = options.Value;
    }
    public async Task<IReadOnlyList<CatalogItemDto>> GetCatalogAsync(CancellationToken ct = default)
    {
        // Exemplo: GET https://catalog-api.../api/catalog/items
        var response = await _http.GetFromJsonAsync<IReadOnlyList<CatalogItemDto>>(
            "/api/catalog/items",
            ct);
        return response ?? Array.Empty<CatalogItemDto>();
    }
}
public sealed class CatalogApiOptions
{
    public string BaseUrl { get; set; } = string.Empty;
}
```

Arquivo: Bff/BackendServices/InventoryBackendService.cs

``` 
using System.Net.Http.Json;
using Microsoft.Extensions.Options;
using Shared.Dtos;
namespace Bff.BackendServices;
public interface IInventoryBackendService
{
    Task<IReadOnlyList<InventoryItemDto>> GetInventoryAsync(CancellationToken ct = default);
}
public sealed class InventoryBackendService : IInventoryBackendService
{
    private readonly HttpClient _http;
    private readonly InventoryApiOptions _options;
    public InventoryBackendService(HttpClient http, IOptions<InventoryApiOptions> options)
    {
        _http = http;
        _options = options.Value;
    }
    public async Task<IReadOnlyList<InventoryItemDto>> GetInventoryAsync(CancellationToken ct = default)
    {
        // Exemplo: GET https://inventory-api.../api/inventory
        var response = await _http.GetFromJsonAsync<IReadOnlyList<InventoryItemDto>>(
            "/api/inventory",
            ct);
        return response ?? Array.Empty<InventoryItemDto>();
    }
}
public sealed class InventoryApiOptions
{
    public string BaseUrl { get; set; } = string.Empty;
}
```

Arquivo: Bff/Program.cs

``` 
using Bff.BackendServices;
using Microsoft.AspNetCore.Http.HttpResults;
using Shared.Dtos;
var builder = WebApplication.CreateBuilder(args);
// CORS
builder.Services.AddCors(o => o.AddDefaultPolicy(p =>
    p.WithOrigins("http://localhost:5173", "https://localhost:5173")
     .AllowAnyHeader()
     .AllowAnyMethod()
     .AllowCredentials()));
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();
// Bind das configurações de APIs
builder.Services.Configure<CatalogApiOptions>(
    builder.Configuration.GetSection("CatalogApi"));
builder.Services.Configure<InventoryApiOptions>(
    builder.Configuration.GetSection("InventoryApi"));
// HttpClient tipado para cada backend
builder.Services.AddHttpClient<ICatalogBackendService, CatalogBackendService>((sp, http) =>
{
    var opts = sp.GetRequiredService<Microsoft.Extensions.Options.IOptions<CatalogApiOptions>>().Value;
    http.BaseAddress = new Uri(opts.BaseUrl);
});
builder.Services.AddHttpClient<IInventoryBackendService, InventoryBackendService>((sp, http) =>
{
    var opts = sp.GetRequiredService<Microsoft.Extensions.Options.IOptions<InventoryApiOptions>>().Value;
    http.BaseAddress = new Uri(opts.BaseUrl);
});
var app = builder.Build();
app.UseCors();
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}
// ===============================
// Endpoint direto: só catálogo
// (pass-through simples)
// GET /api/catalog
// ===============================
app.MapGet("/api/catalog", async Task<Ok<IReadOnlyList<CatalogItemDto>>> (
    ICatalogBackendService catalogService,
    CancellationToken ct) =>
{
    var catalog = await catalogService.GetCatalogAsync(ct);
    return TypedResults.Ok(catalog);
})
.WithName("Catalog_List");
// ============================================
// Endpoint com lógica de agregação (stitching)
// GET /api/catalog/with-stock
// o endpoint anterior usava Task.WhenAll sem tratamento de falha
// parcial — se o serviço de inventário falhasse, o endpoint falhava completamente mesmo com
// o catálogo disponível. Implementada degradação graciosa: se o inventário estiver
// indisponível, retorna o catálogo com stock marcado como desconhecido (StockAvailable=false,
// AvailableQuantity=0) e um indicador explícito "stockUnavailable" no contrato de resposta.
// Esta decisão deve ser documentada como ADR se ainda não o estiver.
// ============================================
app.MapGet("/api/catalog/with-stock", async Task<Ok<CatalogWithStockResponse>> (
    ICatalogBackendService catalogService,
    IInventoryBackendService inventoryService,
    CancellationToken ct) =>
{
    var catalog = await catalogService.GetCatalogAsync(ct);
 
    IReadOnlyList<InventoryItemDto> inventory = Array.Empty<InventoryItemDto>();
    var stockUnavailable = false;
 
    try
    {
        inventory = await inventoryService.GetInventoryAsync(ct);
    }
    catch
    {
        // Degradação graciosa: catálogo retornado sem dados de stock
        stockUnavailable = true;
    }
 
    var inventoryById = inventory
        .GroupBy(i => i.CatalogItemId)
        .ToDictionary(g => g.Key, g => g.First());
 
    var result = new List<CatalogWithStockDto>(catalog.Count);
    foreach (var item in catalog)
    {
        inventoryById.TryGetValue(item.Id, out var stock);
        var stitched = new CatalogWithStockDto(
            Id: item.Id,
            Name: item.Name,
            Price: item.Price,
            InStock: stock?.InStock ?? false,
            AvailableQuantity: stock?.AvailableQuantity ?? 0,
            WarehouseLocation: stock?.WarehouseLocation ?? "N/A"
        );
        result.Add(stitched);
    }
 
    return TypedResults.Ok(new CatalogWithStockResponse(
        Items: result,
        StockUnavailable: stockUnavailable
    ));
})
.WithName("Catalog_WithStock");
// (opcional) GET /api/catalog/with-stock/{id} igual ao exemplo anterior, só mudando a origem dos dados
app.Run();
```

---

# **MVVM**

O padrão MVVM (Model–View–ViewModel) é uma das abordagens arquiteturais mais utilizadas no desenvolvimento de interfaces modernas, especialmente em aplicações que exigem separação clara de responsabilidades, testabilidade e previsibilidade de estado. Em aplicações React Native, o MVVM ajuda a organizar o fluxo entre UI, lógica e dados, reduzindo acoplamento e tornando o projeto mais escalável.

**Objetivos do MVVM**

O padrão MVVM visa resolver desafios recorrentes no desenvolvimento mobile:

- separar claramente apresentação e lógica;
- evitar que ecrãs tenham regra de negócio embutida;
- permitir testabilidade de fluxos sem depender da UI;
- padronizar comunicação entre dados e interface;
- facilitar manutenção e evolução do código.

**Componentes do MVVM**

O MVVM é composto por três elementos principais: Model, View e ViewModel. Cada um desempenha responsabilidades específicas e isoladas.

- **Model**

O Model representa os dados de entidades de negócio e persistência local. Exemplos:

-entidades (Product, Order, CartItem);

-repositórios para acesso a base local (SQLite);

-serviços remotos (GraphQL).

O Model nunca deve conhecer a camada de UI.

**Entidade Produto**

``` 
export class Product {
  constructor({ id, name, price, stock }) {
    this.id = id;
    this.name = name;
    this.price = price;
    this.stock = stock;
  }
}
```

**Repositório de Produto**

``` 
import * as SQLite from "expo-sqlite";

const db = SQLite.openDatabase("app.db");

export const productRepository = {
  init() {
    db.transaction(tx => {
      tx.executeSql(`
        CREATE TABLE IF NOT EXISTS products (
          id INTEGER PRIMARY KEY,
          name TEXT,
          price REAL,
          stock INTEGER
        );
      `);
    });
  },

  getAll() {
    return new Promise((resolve, reject) => {
      db.transaction(tx => {
        tx.executeSql(
          "SELECT * FROM products;",
          [],
          (_, { rows }) => resolve(rows._array),
          (_, error) => reject(error)
        );
      });
    });
  },

  saveProducts(products) {
    db.transaction(tx => {
      products.forEach(p => {
        tx.executeSql(
          `INSERT OR REPLACE INTO products (id, name, price, stock)
           VALUES (?, ?, ?, ?)`,
          [p.id, p.name, p.price, p.stock]
        );
      });
    });
  }
};
```

- **View**

A View corresponde aos ecrãs e componentes visuais — no React Native, arquivos .js ou .tsx que exibem informações. Ela:

-não deve conter lógica de negócio;

-observa dados expostos pelo ViewModel;

-dispara eventos (toques, scroll, botões).

Views apenas refletem o estado fornecido pelo ViewModel.

**View de Catálogo**

``` 
import React, { useEffect } from "react";
import { View, Text, FlatList, ActivityIndicator } from "react-native";
import { useDispatch, useSelector } from "react-redux";
import { loadCatalog } from "../viewmodels/catalogViewModel";

export default function CatalogScreen() {
  const dispatch = useDispatch();
  const { products, loading } = useSelector(state => state.catalog);

  useEffect(() => {
    dispatch(loadCatalog());
  }, []);

  if (loading) {
    return <ActivityIndicator size="large" style={{ marginTop: 40 }} />;
  }

  return (
    <View style={{ padding: 16 }}>
      <FlatList
        data={products}
        keyExtractor={item => item.id.toString()}
        renderItem={({ item }) => (
          <View
            style={{
              padding: 12,
              borderBottomWidth: 1,
              borderColor: "#ddd"
            }}
          >
            <Text style={{ fontSize: 18 }}>{item.name}</Text>
            <Text style={{ color: "#888" }}>€ {item.price}</Text>
            <Text style={{ marginTop: 4 }}>Stock: {item.stock}</Text>
          </View>
        )}
      />
    </View>
  );
}
```

- **ViewModel**

O ViewModel é o núcleo da arquitetura. Ele:

-contém a lógica da ecrã;

-interage com Models e serviços;

-mantém estado observável (Redux, Zustand ou Context API);

-expõe dados já preparados para a View (ex.: preço formatado);

-não conhece a View diretamente — comunica-se por bindings ou listeners.

Em React Native, o ViewModel é geralmente implementado como:

-hooks personalizados (useCatalogViewModel);

-slices do Redux Toolkit;

-serviços que encapsulam lógica de ecrã.

**ViewModel de Catálogo**

``` 
import { createSlice, createAsyncThunk } from "@reduxjs/toolkit";
import { productRepository } from "../models/productRepository";
import { gql } from "@apollo/client";
import client from "../graphqlClient";
 
// GraphQL Query
const GET_PRODUCTS = gql`
  query {
    products {
      id
      name
      price
      stock
    }
  }
`;
 
// Carrega primeiro da base local → depois tenta sincronizar com API
export const loadCatalog = createAsyncThunk(
  "catalog/loadCatalog",
  async () => {
    const localProducts = await productRepository.getAll();
 
    // retorno offline-first (UI já mostra algo)
    let result = { products: localProducts, fromServer: false };
 
    try {
      const { data } = await client.query({ query: GET_PRODUCTS });
 
      if (data?.products) {
        productRepository.saveProducts(data.products);
        result = { products: data.products, fromServer: true };
      }
    } catch {
      console.log("Sem internet. Exibindo dados locais.");
    }
 
    return result;
  }
);
 
const catalogSlice = createSlice({
  name: "catalog",
  initialState: {
    products: [],
    loading: false,
    lastSync: null,
    error: null, // Adicionado: campo "error" no estado inicial para suportar o caso rejected
  },
  reducers: {},
  extraReducers: builder => {
    builder
      .addCase(loadCatalog.pending, state => {
        state.loading = true;
        state.error = null; // limpa erro anterior ao iniciar nova tentativa
      })
      .addCase(loadCatalog.fulfilled, (state, action) => {
        state.products = action.payload.products;
        state.loading = false;
        state.error = null;
 
        if (action.payload.fromServer) {
          state.lastSync = new Date().toISOString();
        }
      })
      // Adicionado: caso "rejected" — sem este caso, a UI não consegue
      // distinguir "a carregar" de "falhou a carregar", impedindo feedback adequado ao utilizador.
      .addCase(loadCatalog.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error?.message ?? "Erro desconhecido ao carregar catálogo.";
      });
  }
});
 
export default catalogSlice.reducer;
```

- **MVVM em React Native + Redux**

Em ambientes React Native, Redux Toolkit funciona de forma natural como ViewModel porque:

-faz gestão do estado global e local;

-mantém previsibilidade (funções puras);

-permite testes unitários isolados;

-é compatível com offline-first (persist + rehydration).

Cada slice do Redux pode ser visto como um ViewModel. Exemplo:

-catalogSlice → estado da listagem;

-cartSlice → lógica do carrinho;

-orderSlice → criação de pedidos.

Views apenas despacham actions e consomem estado.

- **Fluxo Operacional MVVM**

Um fluxo típico:

1. View é carregada e solicita dados ao ViewModel;
2. ViewModel obtém dados do Model (SQLite, cache, etc.);
3. ViewModel prepara os dados (ex.: formatar preços);
4. View observa e renderiza automaticamente;
5. Utilizador interage;
6. ViewModel processa ações e atualiza Models;
7. Quando online, o Model sincroniza com backend;
8. ViewModel recebe atualizações e atualiza View.



- **Boas Práticas**

-Nunca coloque lógica de negócio em componentes React;

-Mantenha ViewModel livre de dependências da UI;

-Mantenha Models puros e reutilizáveis;

-Evite acoplamento circular (View ↔ ViewModel);

-Cada View deve ter seu próprio ViewModel dedicado;

-Mantenha ViewModels pequenos e específicos;

-Iterações de utilizador devem sempre chamar métodos do ViewModel.


---

# **Princípio Offline First**

O princípio de “offline-first” tem se tornado um dos pilares modernos para o desenvolvimento de aplicações mobile, especialmente em cenários onde a disponibilidade da rede é intermitente ou simplesmente não confiável. Em vez de assumir que a conexão está sempre disponível — como muitos aplicativos tradicionais fazem — o offline-first parte do pressuposto oposto: a aplicação deve funcionar totalmente sem internet. A conectividade passa a ser tratada como um bônus, e não como uma dependência fundamental.

Em dispositivos móveis, onde quedas de rede, alternância entre Wi-Fi e dados móveis e latências elevadas são comuns, esse princípio traz benefícios claros:

- maior confiabilidade na experiência do utilizador;
- uso contínuo da aplicação mesmo em ambientes com rede ruim ou inexistente;
- redução do número de erros causados por timeouts e falhas de conexão;
- carregamento inicial mais rápido, pois os dados podem vir do armazenamento local;
- percepção de performance superior e menor taxa de abandono do utilizador.

## **Conceito de Offline-First**

No modelo offline-first, o dispositivo e o armazenamento local são tratados como a “primeira fonte de verdade” para a aplicação. Isso significa que o fluxo padrão de acesso a dados passa a ser centrado no dispositivo, e não no servidor remoto. 

De forma simplificada, a aplicação:

- lê dados primeiro do armazenamento local (base ou cache persistente);
- interage com a base local para criar, atualizar ou remover registros;
- registra operações de escrita em uma fila de sincronização quando a rede não está disponível;
- sincroniza alterações com o servidor quando a conectividade é restabelecida.

Esse fluxo garante que o utilizador não seja impedido de utilizar o aplicativo por falta de conexão. O backend deixa de ser um ponto único de bloqueio e passa a atuar como fonte de consolidação, integração e sincronização dos dados.

## **Benefícios para React Native**

Aplicações desenvolvidas com React Native — incluindo aquelas que utilizam o ecossistema Expo beneficiam-se significativamente de uma arquitetura offline-first. Como o código JavaScript é executado em um ambiente com recursos limitados e sujeito a múltiplos contextos de rede, ter uma estratégia robusta de uso de dados locais traz vantagens importantes: 

- operações de leitura tornam-se extremamente rápidas, pois usam a base local e não dependem da latência do servidor;
- a interface do utilizador reage de forma imediata, sem aguardar round-trips de rede;
- o backend é menos sobrecarregado, já que diversas operações são consolidadas e sincronizadas em lote;
- torna-se natural usar “optimistic UI”, onde alterações aparecem instantaneamente na ecrã antes mesmo da confirmação do servidor;
- a aplicação passa a ser mais resiliente a quedas de rede, reconexões e atrasos.

## **Componentes Essenciais da Arquitetura Offline-First**

### **Base de dados local**

A base de dados local é o núcleo técnico de uma solução offline-first. É nele que a aplicação guarda os dados necessários para operar mesmo sem qualquer comunicação com o servidor. Em projetos React Native/Expo, algumas opções comuns incluem:

- expo-sqlite: fornece acesso a uma base SQLite local de forma integrada ao ambiente Expo, sem necessidade de configuração nativa adicional;
- AsyncStorage: adequado para armazenar pares chave/valor simples, flags e pequenas estruturas de dados, mas não substitui uma base relacional ou de documentos;
- Realm: base orientada a objetos, com recursos avançados e bom desempenho, útil em cenários de dados complexos;
- WatermelonDB ou soluções similares: focadas em sincronização pesada e grandes volumes de dados.

A base local pode manter cópias completas ou parciais dos dados necessários ao funcionamento d aplicação, como catálogo de produtos, carrinho do utilizador, histórico de pedidos, configurações, preferências e qualquer outro conjunto de informações crítico para a experiência offline.

``` 
import * as SQLite from "expo-sqlite";

export const db = SQLite.openDatabase("app.db");

export function initDb() {
  db.transaction(tx => {
    tx.executeSql(`
      CREATE TABLE IF NOT EXISTS products (
        id INTEGER PRIMARY KEY,
        name TEXT,
        price REAL,
        stock INTEGER
      );
    `);

    tx.executeSql(`
      CREATE TABLE IF NOT EXISTS pending_operations (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        type TEXT,
        payload TEXT,
        createdAt TEXT
      );
    `);
  });
}
```

## **Camada de acesso a dados (repositories)**

Os arquivos JavaScript ou TypeScript responsáveis por acessar a base local funcionam como uma camada de repositório. Essa camada tem o objetivo de encapsular a lógica de persistência, oferecendo funções claras para operações de leitura e gravação a base de dados local.

Ao centralizar o acesso ao armazenamento local nesses repositórios, o restante da aplicação pode trabalhar com uma API mais limpa e desacoplada dos detalhes de implementação da BD.

``` 
import { db } from "../storage/db";
 
// Nota de segurança: os payloads são serializados como JSON em texto puro no SQLite.
// Em dispositivos comprometidos ou com acesso físico, dados de pedidos (itens, quantidades,
// valores) ficam expostos. Em alinhamento com a boa prática já documentada na secção "Boas
// Práticas" (evitar dados sensíveis em texto puro), considerar uma das seguintes abordagens
// em implementação real:
//   1. Cifrar o payload antes de persistir (ex.: AES-256 com chave derivada das credenciais do utilizador);
//   2. Armazenar apenas referências (IDs) no payload e reconstruir os dados no momento da sincronização;
//   3. Usar SQLCipher ou equivalente para cifrar toda a base SQLite.
export const pendingQueueRepository = {
  add(operation) {
    db.transaction(tx => {
      tx.executeSql(
        `INSERT INTO pending_operations (type, payload, createdAt)
         VALUES (?, ?, datetime('now'))`,
        [operation.type, JSON.stringify(operation.payload)]
      );
    });
  },
 
  getAll() {
    return new Promise(resolve => {
      db.transaction(tx => {
        tx.executeSql(
          "SELECT * FROM pending_operations ORDER BY id ASC;",
          [],
          (_, { rows }) => resolve(rows._array)
        );
      });
    });
  },
 
  remove(id) {
    db.transaction(tx => {
      tx.executeSql("DELETE FROM pending_operations WHERE id = ?;", [id]);
    });
  }
};
```

## **Fila de operações pendentes (Pending Queue)**

Alterações realizadas pelo utilizador enquanto a aplicação está offline não devem ser simplesmente descartadas. Para isso, utiliza-se uma fila de operações pendentes. Cada operação é registrada com informações que permitam sua execução posterior no servidor, como: 

- tipo da operação (por exemplo, criar pedido, atualizar endereço, adicionar item ao carrinho remoto);

- payload com os dados necessários para executar a operação no backend;

- data e hora da criação;

- estado da operação (pendente, sincronizada, falha, em retry).

Essa fila costuma ser armazenada na própria base local, garantindo que as operações não se percam mesmo que o aplicativo seja encerrado ou o dispositivo seja reiniciado.

``` 
import { db } from "../storage/db";

export const pendingQueueRepository = {
  add(operation) {
    db.transaction(tx => {
      tx.executeSql(
        `INSERT INTO pending_operations (type, payload, createdAt)
         VALUES (?, ?, datetime('now'))`,
        [operation.type, JSON.stringify(operation.payload)]
      );
    });
  },

  getAll() {
    return new Promise(resolve => {
      db.transaction(tx => {
        tx.executeSql(
          "SELECT * FROM pending_operations ORDER BY id ASC;",
          [],
          (_, { rows }) => resolve(rows._array)
        );
      });
    });
  },

  remove(id) {
    db.transaction(tx => {
      tx.executeSql("DELETE FROM pending_operations WHERE id = ?;", [id]);
    });
  }
};
```

## **Serviço de sincronização**

O serviço de sincronização é o componente responsável por orquestrar a comunicação entre o estado local e o backend.

Suas principais responsabilidades incluem:

- detectar quando a conectividade de rede está disponível;

- ler a fila de operações pendentes e enviá-las ao servidor, na ordem correta;

- aplicar políticas de repetição (retry) em caso de falhas temporárias;

- resolver conflitos entre dados locais e remotos, de acordo com regras definidas (por exemplo, server wins, client wins ou merge);

- atualizar a base local com as informações retornadas pelo backend após a sincronização.

``` 
// Pendingqueueprocessor
import * as Network from "expo-network";
import { pendingQueueRepository } from "../repositories/pendingQueueRepository";
import client from "../graphqlClient";
import { CREATE_ORDER } from "../mutations/orderMutations";

// Tipos
type OperationType = "CREATE_ORDER";

interface PendingOperation {
  id: string;
  type: OperationType;
  payload: string;
  retryCount?: number;
}

interface OperationResult {
  id: string;
  success: boolean;
  error?: unknown;
}

const MAX_RETRIES = 3;

// Dispatcher: mapeia cada tipo de operação para sua execução.
// Adicionar um novo tipo = adicionar uma entrada aqui, sem tocar no loop.
const operationHandlers: Record<OperationType, (payload: unknown) => Promise<void>> = {
  CREATE_ORDER: async (payload) => {
    await client.mutate({
      mutation: CREATE_ORDER,
      variables: payload as Record<string, unknown>,
    });
  },
};

function parsePayload(raw: string): unknown {
  try {
    return JSON.parse(raw);
  } catch {
    throw new Error(`Payload inválido (não é JSON): ${raw.slice(0, 80)}`);
  }
}

function isRecoverableError(err: unknown): boolean {
  // Erros de rede são recuperáveis; erros de validação/payload, não.
  if (err instanceof Error) {
    const msg = err.message.toLowerCase();
    return msg.includes("network") || msg.includes("timeout") || msg.includes("failed to fetch");
  }
  return false;
}

async function processOperation(op: PendingOperation): Promise<OperationResult> {
  const handler = operationHandlers[op.type];

  if (!handler) {
    // Tipo desconhecido: remove da fila para não bloquear para sempre.
    __DEV__ && console.warn(`[Queue] Tipo de operação desconhecido: ${op.type}. Removendo.`);
    await pendingQueueRepository.remove(op.id);
    return { id: op.id, success: false, error: new Error(`Tipo desconhecido: ${op.type}`) };
  }

  const retries = op.retryCount ?? 0;

  if (retries >= MAX_RETRIES) {
    __DEV__ && console.warn(`[Queue] Operação ${op.id} atingiu limite de retentativas. Descartando.`);
    await pendingQueueRepository.remove(op.id);
    return { id: op.id, success: false, error: new Error("Limite de retentativas atingido") };
  }

  try {
    const payload = parsePayload(op.payload);
    await handler(payload);
    await pendingQueueRepository.remove(op.id);
    return { id: op.id, success: true };
  } catch (err) {
    __DEV__ && console.warn(`[Queue] Falha ao sincronizar operação ${op.id}:`, err);

    if (!isRecoverableError(err)) {
      // Erro irrecuperável (payload corrompido, erro 400 etc): remove para não bloquear a fila.
      await pendingQueueRepository.remove(op.id);
    } else {
      // Erro recuperável: incrementa contador de retentativas.
      await pendingQueueRepository.incrementRetry(op.id);
    }

    return { id: op.id, success: false, error: err };
  }
}

export async function processPendingQueue(): Promise<void> {
  const network = await Network.getNetworkStateAsync();
  if (!network.isConnected) return;

  const operations = await pendingQueueRepository.getAll();
  if (operations.length === 0) return;

  // Processa todas em paralelo e coleta os resultados — uma falha não bloqueia as demais.
  const results = await Promise.allSettled(
    operations.map((op) => processOperation(op as PendingOperation))
  );

  if (__DEV__) {
    const failed = results.filter((r) => r.status === "rejected").length;
    if (failed > 0) console.warn(`[Queue] ${failed} operação(ões) falharam neste ciclo.`);
  }
}
```

**Testes unitários (essencial)**

```
// Pendingqueueprocessor.test
import { processPendingQueue } from "../services/pendingQueueProcessor";
import { pendingQueueRepository } from "../repositories/pendingQueueRepository";
import client from "../graphqlClient";
import * as Network from "expo-network";

// Mocks
jest.mock("expo-network");
jest.mock("../repositories/pendingQueueRepository");
jest.mock("../graphqlClient", () => ({ mutate: jest.fn() }));

const mockNetwork = Network.getNetworkStateAsync as jest.Mock;
const mockGetAll = pendingQueueRepository.getAll as jest.Mock;
const mockRemove = pendingQueueRepository.remove as jest.Mock;
const mockIncrementRetry = pendingQueueRepository.incrementRetry as jest.Mock;
const mockMutate = client.mutate as jest.Mock;

const connected = { isConnected: true };
const disconnected = { isConnected: false };

const makeOp = (overrides = {}) => ({
  id: "op-1",
  type: "CREATE_ORDER",
  payload: JSON.stringify({ items: [{ productId: "p1", quantity: 2 }] }),
  retryCount: 0,
  ...overrides,
});

beforeEach(() => jest.clearAllMocks());

// ─── Testes ───────────────────────────────────────────────────────────────────

describe("processPendingQueue", () => {
  it("não faz nada quando sem conexão", async () => {
    mockNetwork.mockResolvedValue(disconnected);

    await processPendingQueue();

    expect(pendingQueueRepository.getAll).not.toHaveBeenCalled();
  });

  it("não faz nada quando a fila está vazia", async () => {
    mockNetwork.mockResolvedValue(connected);
    mockGetAll.mockResolvedValue([]);

    await processPendingQueue();

    expect(client.mutate).not.toHaveBeenCalled();
  });

  it("executa a mutation e remove da fila em caso de sucesso", async () => {
    mockNetwork.mockResolvedValue(connected);
    mockGetAll.mockResolvedValue([makeOp()]);
    mockMutate.mockResolvedValue({ data: { createOrder: { id: "order-1", total: 100 } } });
    mockRemove.mockResolvedValue(undefined);

    await processPendingQueue();

    expect(client.mutate).toHaveBeenCalledTimes(1);
    expect(pendingQueueRepository.remove).toHaveBeenCalledWith("op-1");
  });

  it("não aborta o loop quando uma operação falha — as demais continuam", async () => {
    const op1 = makeOp({ id: "op-1" });
    const op2 = makeOp({ id: "op-2" });
    mockNetwork.mockResolvedValue(connected);
    mockGetAll.mockResolvedValue([op1, op2]);
    mockMutate
      .mockRejectedValueOnce(new Error("network timeout")) // op-1 falha
      .mockResolvedValueOnce({ data: {} });               // op-2 sucede
    mockIncrementRetry.mockResolvedValue(undefined);
    mockRemove.mockResolvedValue(undefined);

    await processPendingQueue();

    // op-2 deve ter sido processada mesmo com op-1 falhando
    expect(client.mutate).toHaveBeenCalledTimes(2);
    expect(pendingQueueRepository.remove).toHaveBeenCalledWith("op-2");
    expect(pendingQueueRepository.remove).not.toHaveBeenCalledWith("op-1");
  });

  it("incrementa retryCount em erros de rede (recuperáveis)", async () => {
    mockNetwork.mockResolvedValue(connected);
    mockGetAll.mockResolvedValue([makeOp()]);
    mockMutate.mockRejectedValue(new Error("network error"));
    mockIncrementRetry.mockResolvedValue(undefined);

    await processPendingQueue();

    expect(pendingQueueRepository.incrementRetry).toHaveBeenCalledWith("op-1");
    expect(pendingQueueRepository.remove).not.toHaveBeenCalled();
  });

  it("remove da fila em erros irrecuperáveis (ex: payload inválido)", async () => {
    mockNetwork.mockResolvedValue(connected);
    mockGetAll.mockResolvedValue([makeOp({ payload: "INVALID_JSON{{{" })]);
    mockRemove.mockResolvedValue(undefined);

    await processPendingQueue();

    // Payload inválido = erro irrecuperável = deve ser removido
    expect(pendingQueueRepository.remove).toHaveBeenCalledWith("op-1");
    expect(pendingQueueRepository.incrementRetry).not.toHaveBeenCalled();
  });

  it("descarta operação que atingiu o limite de retentativas", async () => {
    mockNetwork.mockResolvedValue(connected);
    mockGetAll.mockResolvedValue([makeOp({ retryCount: 3 })]);
    mockRemove.mockResolvedValue(undefined);

    await processPendingQueue();

    expect(pendingQueueRepository.remove).toHaveBeenCalledWith("op-1");
    expect(client.mutate).not.toHaveBeenCalled();
  });

  it("remove operações de tipo desconhecido sem travar a fila", async () => {
    mockNetwork.mockResolvedValue(connected);
    mockGetAll.mockResolvedValue([makeOp({ type: "UNKNOWN_OP" })]);
    mockRemove.mockResolvedValue(undefined);

    await processPendingQueue();

    expect(pendingQueueRepository.remove).toHaveBeenCalledWith("op-1");
    expect(client.mutate).not.toHaveBeenCalled();
  });
});
```

**Mutations**
```
// Ordermutations
import { gql } from "@apollo/client";

export const CREATE_ORDER = gql`
  mutation CreateOrder($items: [OrderItemInput!]!) {
    createOrder(items: $items) {
      id
      total
    }
  }
`;
```

## **Detecção de rede**

Para implementar o comportamento offline-first de forma eficiente, a aplicação precisa saber quando está online ou offline. No ecossistema Expo, é comum usar o pacote expo-network para consultar o estado da conexão e reagir a mudanças. Com isso, torna-se possível disparar a sincronização  automaticamente assim que a rede estiver novamente disponível.

No React Native (sem Expo), pode-se usar a biblioteca @react-native-community/netinfo para esse fim.

## **Fluxo Operacional do Offline-First**

Do ponto de vista do utilizador, o fluxo ideal de uma aplicação offline-first deve ser fluido e transparente.

Um cenário típico pode ser descrito da seguinte forma:

1. O utilizador abre o aplicativo em um contexto sem internet;
2. A ecrã inicial é carregada a partir da base de dados local, exibindo o catálogo de produtos e demais informações disponíveis;
3. O utilizador adiciona itens ao carrinho, cria pedidos ou altera dados de perfil;
4. Cada operação é gravada no armazenamento local e registrada na fila de operações pendentes;
5. Quando a rede volta, o serviço de sincronização detecta a conectividade e:

a) envia as operações pendentes ao backend;

b) recebe a resposta do servidor e ajusta a base local conforme necessário;

c) atualiza a interface para refletir o estado sincronizado.

Esse modelo garante que a experiência do utilizador seja contínua, com o mínimo de frustração possível, ainda que a infraestrutura de rede não seja ideal.

## **Estratégias de Resolução de Conflitos**

Quando tanto o cliente quanto o servidor fazem alterações em um mesmo registro em momentos diferentes, podem surgir conflitos de sincronização. Algumas estratégias comuns para lidar com esses conflitos são:

-Server Wins: o servidor sempre prevalece. É a estratégia mais simples, porém pode descartar alterações recentes feitas no cliente offline.

-Client Wins: as mudanças feitas no cliente têm prioridade sobre as do servidor. Funciona bem em aplicações pessoais com poucos utilizadores concorrentes.

-Merge: uma lógica personalizada tenta combinar as informações conflitantes. Essa abordagem é mais complexa, mas pode ser necessária em domínios sensíveis. 

A escolha da estratégia adequada depende do domínio de negócio, da criticidade dos dados e dos requisitos de consistência da aplicação.

## **Boas Práticas**

A adoção do offline-first vai além da implementação técnica; envolve também um conjunto de boas práticas que fortalecem a solução:

-Evitar o armazenamento de dados sensíveis em texto puro no SQLite ou em outras bases locais;

-Garantir que a fila de operações pendentes seja persistida para sobreviver a reinícios da aplicação e do dispositivo;

-Implementar logs internos para facilitar a depuração de problemas de sincronização;

-Tratar falhas de sincronização com mensagens amigáveis e estados de UI claros (por exemplo, indicar itens ainda não sincronizados);

-Criar uma UI que reflita o estado online/offline e informe ao utilizador quando certas funcionalidades dependem de conexão;

-Evitar operações destrutivas irreversíveis enquanto o aplicativo estiver offline, especialmente em contextos multiutilizador.

## **Integração com Apollo Client**

O Apollo Client, amplamente utilizado em aplicações React e React Native para consumo de APIs GraphQL, pode ser configurado de forma amigável ao paradigma offline-first. Algumas práticas úteis incluem:

-uso de InMemoryCache para armazenar resultados de queries no cliente;persistência desse cache em armazenamento local (por exemplo, AsyncStorage) para evitar recarregar dados após reiniciar o aplicativo;

-configuração de políticas de busca (fetchPolicy) como cache-first ou cache-and-network, priorizando dados locais;

-uso de mutations otimistas (optimisticResponse) para atualizar a UI imediatamente, antes da confirmação do servidor.

Quando combinado com a base local, a fila de operações pendentes e o serviço de sincronização, o Apollo torna-se parte importante de uma arquitetura robusta e escalável de dados.

## Exemplos de código

**ViewModel offline-first**

``` 
mport { createAsyncThunk, createSlice } from "@reduxjs/toolkit";
import { gql } from "@apollo/client";
import { productRepository } from "../repositories/productRepository";
import client from "../graphqlClient";
 
const GET_PRODUCTS = gql`
  query {
    products {
      id
      name
      price
      stock
    }
  }
`;
 
export const loadCatalog = createAsyncThunk("catalog/load", async () => {
  const local = await productRepository.getAll();
 
  let result = { items: local, synced: false };
 
  try {
    const { data } = await client.query({ query: GET_PRODUCTS });
 
    if (data?.products) {
      await productRepository.saveAll(data.products);
      result = { items: data.products, synced: true };
    }
  } catch (err) {
    console.log("Offline — usando dados locais");
  }
 
  return result;
});
 
const catalogSlice = createSlice({
  name: "catalog",
  initialState: { items: [], loading: false, lastSync: null, error: null }, // Adicionado campo "error"
  extraReducers: builder => {
    builder.addCase(loadCatalog.pending, state => {
      state.loading = true;
      state.error = null;
    });
    builder.addCase(loadCatalog.fulfilled, (state, action) => {
      state.items = action.payload.items;
      state.loading = false;
      state.error = null;
 
      if (action.payload.synced)
        state.lastSync = new Date().toISOString();
    });
    // Adicionado: caso "rejected" para persistir estado de erro no store
    // e permitir que a View exiba feedback adequado em cenários de falha.
    builder.addCase(loadCatalog.rejected, (state, action) => {
      state.loading = false;
      state.error = action.error?.message ?? "Erro desconhecido ao carregar catálogo.";
    });
  }
});
 
export default catalogSlice.reducer;
```

**View offline-first**

``` 
import React, { useEffect } from "react";
import { View, Text, ActivityIndicator, FlatList } from "react-native";
import { useDispatch, useSelector } from "react-redux";
import { loadCatalog } from "../viewmodels/catalogViewModel";

export default function CatalogScreen() {
  const dispatch = useDispatch();
  const { items, loading } = useSelector(state => state.catalog);

  useEffect(() => {
    dispatch(loadCatalog());
  }, []);

  if (loading) return <ActivityIndicator size="large" />;

  return (
    <FlatList
      data={items}
      keyExtractor={i => i.id.toString()}
      renderItem={({ item }) => (
        <View style={{ padding: 16, borderBottomWidth: 1 }}>
          <Text style={{ fontSize: 18 }}>{item.name}</Text>
          <Text style={{ color: "#888" }}>€ {item.price}</Text>
          <Text>Estoque: {item.stock}</Text>
        </View>
      )}
    />
  );
}
```

**Enfileiramento**

``` 
import { pendingQueueRepository } from "../repositories/pendingQueueRepository";
import client from "../graphqlClient";
import { gql } from "@apollo/client";
import * as Network from "expo-network";

const CREATE_ORDER = gql`
  mutation CreateOrder($items: [OrderItemInput!]!) {
    createOrder(items: $items) {
      id
      total
    }
  }
`;

export async function createOrder(items) {
  const network = await Network.getNetworkStateAsync();

  if (!network.isConnected) {
    pendingQueueRepository.add({
      type: "CREATE_ORDER",
      payload: { items }
    });
    return { offline: true };
  }

  const result = await client.mutate({
    mutation: CREATE_ORDER,
    variables: { items }
  });

  return result.data.createOrder;
}
```

# **GraphQL**

GraphQL é uma linguagem de consulta e um runtime para APIs, permitindo que o cliente solicite exatamente os dados necessários, de forma tipada e eficiente. Ao contrário de REST, GraphQL:

- evita overfetching (dados demais)
- evita underfetching (dados de menos)
- consolida múltiplos endpoints em uma única interface
- oferece introspecção e tipagem forte
- funciona muito bem com apps mobile e offline-first

Para a Arquiteturas Mobile, GraphQL é ideal porque:

- reduz tráfego de dados
- permite sincronização seletiva
- integra naturalmente com Apollo Client no app
- melhora a performance em cenários offline/online

## HotChocolate.AspNetCore

HotChocolate é um framework moderno de GraphQL para .NET, com:

- suporte completo ao padrão GraphQL
- auto-binding entre classes .NET e o schema
- integração direta com [http://ASP.NET](http://ASP.NET)  Core
- suporte a DataLoaders
- suporte a filtros, paginação, sorting
- integração com Entity Framework Core
- sistema de autorização integrado
- resolvers fortemente tipados
- boa performance e extensibilidade

O HotChocolate oferece:

- um servidor GraphQL leve e rápido
- integração nativa com autenticação e autorização via JWT
- pipeline simples de queries e mutations
- geração automática do schema a partir de tipos .NET

## Arquitetura do GraphQL no Backend

A arquitetura do servidor GraphQL é composta por:

- schema
- queries
- mutations, tipos (types)
- resolvers
- autorização com atributos [Authorize]
- integração com EF Core
- configuração via DI (Dependency Injection)
- pipeline de middleware do HotChocolate

O backend expõe um endpoint único HTTP:

- /graphql

Em ambiente de desenvolvimento também expõe a interface Banana Cake Pop para exploração interativa do schema.

## Configuração Básica do Servidor

Um exemplo simplificado de configuração usando Minimal APIs no .NET 8:

- Registrar o DbContext com EF Core.
- Adicionar o servidor GraphQL com serviços.
- Registrar os tipos de Query e Mutation.
- Ativar autorização, filtros, sorting e projeções.
- Mapear o endpoint /graphql.
- Mapear o Banana Cake Pop em ambiente de desenvolvimento.

``` 
using HotChocolate.AspNetCore;
using HotChocolate.AspNetCore.Playground;
using HotChocolate;
using HotChocolate.Execution;
using HotChocolate.Types;
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);

// Database
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseNpgsql(builder.Configuration.GetConnectionString("DefaultConnection"))
);

// GraphQL Server
builder.Services
    .AddGraphQLServer()
    .AddQueryType<Query>()
    .AddMutationType<Mutation>()
    .AddType<ProductType>()
    .AddAuthorization()
    .AddFiltering()
    .AddSorting()
    .AddProjections();

var app = builder.Build();

app.MapGraphQL("/graphql");

// Banana Cake Pop (dev)
if (app.Environment.IsDevelopment())
{
    app.MapBananaCakePop("/graphql-ui");
}

app.Run();
```

## Queries (Leituras)

A Query é a raiz das operações de leitura. 

Exemplo:

``` 
public class Query
{
    [Authorize] // requer login
    public IQueryable<Product> GetProducts([Service] AppDbContext dbContext)
        => dbContext.Products;

    [Authorize(Roles = new[] { "Admin" })]
    public Product? GetProduct(int id, [Service] AppDbContext dbContext)
        => dbContext.Products.FirstOrDefault(p => p.Id == id);
}
```

Essas queries podem:

- exigir autenticação com [Authorize]
- restringir acesso por roles (ex.: Admin)
- usar IQueryable e EF Core para permitir filtros, sorting e paginação server-side.

## Mutations (Escritas)

Mutations representam operações de escrita. 

Exemplos:

``` 
public class Mutation
{
    [Authorize(Roles = new[] { "Admin" })]
    public async Task<Product> AddProduct(
        AddProductInput input,
        [Service] AppDbContext dbContext)
    {
        var product = new Product
        {
            Name = input.Name,
            Price = input.Price,
            Description = input.Description,
            Stock = input.Stock
        };

        dbContext.Products.Add(product);
        await dbContext.SaveChangesAsync();

        return product;
    }

    [Authorize]
    public async Task<Order> CreateOrder(
        CreateOrderInput input,
        [Service] AppDbContext dbContext)
    {
        var order = new Order
        {
            UserId = input.UserId,
            Items = input.Items,
            Total = input.Total
        };

        dbContext.Orders.Add(order);
        await dbContext.SaveChangesAsync();

        return order;
    }
}
```

Características:

- recebem inputs tipados.
- usam o DbContext para persistir entidades.
- retornam o objeto criado ou atualizado.
- podem ter regras de autorização por papéis ou policies.

## Types (Tipos GraphQL)

- Types permitem controlar como as entidades .NET aparecem no schema GraphQL.

Exemplo:

``` 
public class ProductType : ObjectType<Product>
{
    protected override void Configure(IObjectTypeDescriptor<Product> descriptor)
    {
        descriptor.Field(p => p.Id);
        descriptor.Field(p => p.Name);
        descriptor.Field(p => p.Price);
        descriptor.Field(p => p.Description);
        descriptor.Field(p => p.Stock);
    }
}
```

Benefícios:

- controle granular do schema.
- desacoplamento entre modelo de domínio e contrato público.
- suporte a campos computados.

## Autorização com HotChocolate

HotChocolate oferece autorização nativa através de atributos:

- Por atributo global: exige utilizador autenticado.

``` 
[Authorize]
```

- Por Role: limita acesso a um papel específico.

``` 
[Authorize(Roles = new[] { "Admin" })]
```

- Por policy: aplica policy customizada.

``` 
[Authorize(Policy = "OrdersRead")]
```

A integração é feita com o sistema de autenticação/autorização do [http://ASP.NET](http://ASP.NET)  Core, usando:

- JWT bearer tokens
- Claims
- Roles
- Policies

Assim, o servidor GraphQL aplica as regras de segurança diretamente nos resolvers.

## Integração com Entity Framework Core

HotChocolate integra-se bem com EF Core, oferecendo:

- projeções automáticas
- paginação server-side
- filtros e ordenação com UseFiltering/UseSorting
- DataLoaders para evitar problemas de N+1 consultas

Isso permite consultas eficientes, mesmo em cenários complexos de catálogo e pedidos.

Exemplo:

``` 
descriptor.Field("products")
          .ResolveWith<QueryResolvers>(x => x.GetProductsAsync(default!, default!))
          .UseDbContext<AppDbContext>()
          .UseFiltering()
          .UseSorting()
          .UsePaging();
```

## Ferramentas de Teste e Playground

Durante o desenvolvimento é possível habilitar o Banana Cake Pop:

- interface gráfica acessível em /graphql-ui
- permite testar queries, mutations e subscriptions
- fornece autocomplete e documentação do schema
- ajuda no onboarding de desenvolvedores e debug

## Considerações de Segurança

Ao expor um endpoint GraphQL:

- exigir HTTPS sempre
- exigir autenticação para operações sensíveis
- limitar introspection em produção, se necessário
- filtrar e sanitizar mensagens de erro
- usar rate limiting e logging de segurança


---

# Event Driven Architecture

## **Visão Geral**

A **Arquitetura Orientada a Eventos (Event-Driven Architecture – EDA)** é um estilo arquitetural em que os componentes de um sistema se comunicam por meio de **eventos**, em vez de chamadas diretas e síncronas. Um **evento** representa algo que *já aconteceu* no domínio do negócio (por exemplo: *PedidoCriado*, *PagamentoConfirmado*, *EstoqueAtualizado*).

Nesse modelo, os produtores de eventos **não conhecem** os consumidores. Eles apenas publicam eventos em um barramento (event bus, broker ou streaming platform), e os consumidores reagem a esses eventos de forma assíncrona. Isso promove **baixo acoplamento**, **alta escalabilidade** e **resiliência**.

## **Conceitos Fundamentais**

### **Evento**

Um evento é um fato imutável que descreve uma mudança de estado relevante no sistema.

- É sempre algo no passado
- Não deve ser alterado após publicado
- Deve conter apenas informações necessárias para os consumidores

Exemplos:

- UserRegistered
- OrderPlaced
- PaymentFailed

### **Produtor de Eventos**

Componente responsável por **emitir eventos** quando algo relevante ocorre.

- Não sabe quem vai consumir o evento
- Publica o evento em um broker ou stream

Exemplo: o serviço de pedidos publica OrderPlaced.

### **Consumidor de Eventos**

Componente que **escuta e reage** a eventos.

- Pode haver múltiplos consumidores para o mesmo evento
- Cada consumidor executa sua própria lógica

Exemplo:

- Serviço de estoque reduz quantidade
- Serviço de e-mail envia confirmação
- Serviço de faturamento gera nota fiscal

### **Event Broker / Event Bus**

Infraestrutura responsável por transportar os eventos entre produtores e consumidores.

Exemplos comuns:

- Apache Kafka
- RabbitMQ
- Azure Event Hubs / Service Bus
- AWS SNS/SQS
- Google Pub/Sub

### **Fluxo Básico de Funcionamento**

1. Um evento ocorre no sistema (ex: pedido criado)
2. O produtor publica o evento no broker
3. O broker distribui o evento
4. Um ou mais consumidores recebem o evento
5. Cada consumidor executa sua ação de forma independente

Esse fluxo elimina dependências diretas e permite evolução independente dos serviços.

### **Tipos de Arquitetura Orientada a Eventos**

#### **Event Notification**

O evento serve apenas como **notificação**.

- Contém dados mínimos
- Consumidores buscam mais informações se necessário

Uso comum em integrações simples.

#### **Event-Carried State Transfer**

O evento carrega **todo o estado necessário** para o consumidor.

- Reduz chamadas adicionais
- Aumenta o tamanho do evento

Muito usado em microserviços distribuídos.

#### **Event Sourcing**

Os eventos são a **fonte da verdade** do sistema.

- O estado atual é reconstruído a partir da sequência de eventos
- Permite auditoria completa e time-travel

Exemplo:

- AccountCreated
- MoneyDeposited
- MoneyWithdrawn

O saldo é calculado a partir desses eventos.

### **Benefícios da EDA**

- **Desacoplamento**: produtores e consumidores evoluem de forma independente
- **Escalabilidade horizontal**: consumidores podem ser escalados facilmente
- **Resiliência**: falhas em um consumidor não afetam os outros
- **Extensibilidade**: novos comportamentos podem ser adicionados sem alterar produtores
- **Alta performance** em sistemas assíncronos e distribuídos

### **Desafios e Cuidados**

#### **Consistência Eventual**

- O sistema não é imediatamente consistente
- Estados intermediários são normais
- É necessário aceitar e modelar essa realidade

#### **Idempotência**

- Consumidores devem suportar eventos duplicados
- Cada evento deve ser processado de forma segura mais de uma vez

#### **Ordenação de Eventos**

- Nem todos os brokers garantem ordem global
- É comum garantir ordem apenas por chave (ex: orderId)

#### **Versionamento de Eventos**

- Eventos evoluem ao longo do tempo
- Nunca quebrar consumidores existentes
- Estratégias comuns:
- Versionar o nome do evento
- Tornar campos opcionais
- Usar schemas (Avro, Protobuf, JSON Schema)

### **EDA e Microserviços**

EDA é extremamente comum em arquiteturas de **microserviços**, pois:

- Evita comunicação síncrona excessiva
- Reduz efeito cascata de falhas
- Facilita integração entre domínios

Exemplo prático:

- Serviço de Pedido publica OrderPlaced
- Serviço de Pagamento reage
- Serviço de Estoque reage
- Serviço de Analytics reage
Nenhum deles depende diretamente do outro.

### **Quando Usar Event-Driven Architecture**

EDA é indicada quando:

- O sistema é distribuído
- Há necessidade de escalar consumidores independentemente
- O domínio é orientado a eventos de negócio
- A latência síncrona não é crítica
- É desejável alta extensibilidade

Não é indicada para:

- Sistemas simples e monolíticos
- Fluxos que exigem consistência forte imediata
- Times sem maturidade em sistemas distribuídos

### **Conclusão**

A Arquitetura Orientada a Eventos é um pilar moderno para sistemas escaláveis, resilientes e desacoplados. Quando bem aplicada, ela permite que sistemas cresçam em complexidade sem crescimento proporcional de acoplamento. Porém, exige maturidade técnica, boa observabilidade e entendimento profundo de consistência eventual.

Usada corretamente, EDA transforma eventos de simples mensagens em **primeiros cidadãos do domínio**, refletindo de forma natural como o negócio realmente funciona.

# Processamento Assíncrono no Frontend Web

Toda ação do utilizador que envolve I/O (ex.: criar pedido, salvar, atualizar) deve ser tratada 
como uma operação assíncrona, ligada a um async Task no evento do componente
(@onclick="PlaceFakeAsync"), com flags de estado (IsPlacing) para controlar UI (botões desabilitados, textos “Processando…”).

- Um botão chama um método async:
- @onclick="PlaceFakeAsync"
- Dentro do método:
- seta um flag (IsPlacing = true),
- faz a chamada assíncrona para o BFF (await Service.PlaceAsync(req)),
- trata erro/sucesso,
- atualiza mensagens de status e/ou recarrega a lista (await LoadOrdersAsync()),
- no finally, volta IsPlacing = false.

Ideia central:

> **Não bloquear a UI enquanto o utilizador executa uma ação remota**, evitando múltiplos cliques e dando feedback claro sobre o que está acontecendo.

## **Exemplos em .Net Blazor:**

- AsyncOrderService.cs fazendo chamadas HTTP assíncronas.
- OrdersPage.razor com:
- carregamento inicial async em OnInitializedAsync;
- auto-refresh assíncrono que roda em loop em background no lado do cliente enquanto a página está aberta;
- cancelamento com CancellationTokenSource;
- e ações de UI (Fazer pedido) que disparam processamento assíncrono no cliente;

Frontend/Features/Order/AsyncOrderService.cs

``` 
using System.Net.Http.Json;
using Shared.Dtos;

namespace Frontend.Features.Orders;

public sealed class OrdersService
{
    private readonly HttpClient _http;

    public OrdersService(HttpClient http)
    {
        _http = http;
    }

    /// <summary>
    /// Lista pedidos existentes.
    /// GET /api/orders
    /// </summary>
    public async Task<IReadOnlyList<PlaceOrderResponse>> ListAsync(CancellationToken ct = default)
    {
        var res = await _http.GetFromJsonAsync<OrdersListResponse>("/api/orders", ct);
        return res?.Orders ?? Array.Empty<PlaceOrderResponse>();
    }

    /// <summary>
    /// Envia um novo pedido para o BFF.
    /// POST /api/orders/place
    /// </summary>
    public async Task<PlaceOrderResponse?> PlaceAsync(PlaceOrderRequest req, CancellationToken ct = default)
    {
        var res = await _http.PostAsJsonAsync("/api/orders/place", req, ct);
        if (!res.IsSuccessStatusCode) return null;

        return await res.Content.ReadFromJsonAsync<PlaceOrderResponse>(cancellationToken: ct);
    }
}
```

Frontend/Features/Orders/OrdersPage.razor

``` 
@page "/orders"
@inherits OrdersPageBase

<h3>Pedidos</h3>

<div class="toolbar">
    <button @onclick="PlaceFakeAsync" disabled="@IsPlacing">
        @(IsPlacing ? "Criando pedido..." : "Fazer pedido de teste")
    </button>

    <button @onclick="ToggleAutoRefreshAsync">
        @(AutoRefreshEnabled ? "Parar auto-refresh" : "Iniciar auto-refresh (5s)")
    </button>
</div>

<StatusMessage Message="StatusMessage"
               ErrorMessage="ErrorMessage"
               IsLoading="IsLoading" />

@if (!IsLoading && string.IsNullOrWhiteSpace(ErrorMessage))
{
    <OrdersList Items="Orders" />
}

<style>
.toolbar {
    display: flex;
    gap: 0.5rem;
    margin-bottom: 0.75rem;
}
button[disabled] {
    opacity: 0.7;
    cursor: not-allowed;
}
</style>
```

OrdersPage.razor.cs (Partial class)

``` 
using System.Threading;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Components;
using Shared.Dtos;
using Frontend.Features.Orders;
 
namespace Frontend.Features.Orders;
 
public class OrdersPageBase : ComponentBase, IAsyncDisposable
{
    [Inject] protected OrdersService Service { get; set; } = default!;
 
    protected IReadOnlyList<PlaceOrderResponse> Orders { get; set; } =
        Array.Empty<PlaceOrderResponse>();
 
    protected bool IsLoading { get; set; }
    protected bool IsPlacing { get; set; }
    protected string? ErrorMessage { get; set; }
    protected string? StatusMessage { get; set; }
 
    protected bool AutoRefreshEnabled { get; set; }
    private CancellationTokenSource? _autoRefreshCts;
    private PeriodicTimer? _autoRefreshTimer;
 
    protected override async Task OnInitializedAsync()
    {
        await LoadOrdersAsync();
    }
 
    protected async Task LoadOrdersAsync()
    {
        IsLoading = true;
        ErrorMessage = null;
 
        try
        {
            Orders = await Service.ListAsync();
            StatusMessage = $"Última atualização: {DateTime.Now:T}";
        }
        catch (Exception ex)
        {
            Orders = Array.Empty<PlaceOrderResponse>();
            ErrorMessage = $"Erro ao carregar pedidos: {ex.Message}";
        }
        finally
        {
            IsLoading = false;
            await InvokeAsync(StateHasChanged);
        }
    }
 
    protected async Task PlaceFakeAsync()
    {
        if (IsPlacing) return;
 
        IsPlacing = true;
        ErrorMessage = null;
 
        try
        {
            var req = new PlaceOrderRequest(new[]
            {
                new OrderLineDto(Guid.NewGuid(), 2, 10m),
                new OrderLineDto(Guid.NewGuid(), 1, 25.5m)
            });
 
            var placed = await Service.PlaceAsync(req);
            if (placed is null)
            {
                ErrorMessage = "Falha ao criar pedido.";
            }
            else
            {
                StatusMessage = $"Pedido criado: {placed.OrderId}, Total {placed.Total:C}";
                await LoadOrdersAsync();
            }
        }
        catch (Exception ex)
        {
            ErrorMessage = $"Erro ao criar pedido: {ex.Message}";
        }
        finally
        {
            IsPlacing = false;
        }
    }
 
    protected async Task ToggleAutoRefreshAsync()
    {
        if (AutoRefreshEnabled)
        {
            AutoRefreshEnabled = false;
            StatusMessage = "Auto-refresh desativado.";
            _autoRefreshCts?.Cancel();
            _autoRefreshTimer?.Dispose();
            await InvokeAsync(StateHasChanged);
            return;
        }
 
        AutoRefreshEnabled = true;
        StatusMessage = "Auto-refresh ativado (5s).";
        _autoRefreshCts?.Cancel();
        _autoRefreshCts = new CancellationTokenSource();
        _autoRefreshTimer?.Dispose();
        _autoRefreshTimer = new PeriodicTimer(TimeSpan.FromSeconds(5));
 
        _ = RunAutoRefreshLoopAsync(_autoRefreshCts.Token);
        await InvokeAsync(StateHasChanged);
    }
 
    private bool _isRefreshing; // Adicionado: flag para controlo de concorrência no auto-refresh
 
    private async Task RunAutoRefreshLoopAsync(CancellationToken ct)
    {
        try
        {
            if (_autoRefreshTimer is null)
                return;
 
            // Corrigido: sem este controlo, se uma chamada a LoadOrdersAsync
            // demorar mais de 5 segundos, múltiplas invocações ficam em curso simultaneamente,
            // causando condições de corrida no estado da UI e chamadas redundantes ao BFF.
            // O flag _isRefreshing garante que cada iteração aguarda a conclusão da anterior.
            while (await _autoRefreshTimer.WaitForNextTickAsync(ct))
            {
                if (_isRefreshing) continue;
 
                _isRefreshing = true;
                try
                {
                    await LoadOrdersAsync();
                }
                finally
                {
                    _isRefreshing = false;
                }
            }
        }
        catch (OperationCanceledException)
        {
            // esperado
        }
    }
 
    public async ValueTask DisposeAsync()
    {
        AutoRefreshEnabled = false;
        _autoRefreshCts?.Cancel();
        _autoRefreshTimer?.Dispose();
        _autoRefreshCts?.Dispose();
        await Task.CompletedTask;
    }
}
```
 
```adf 
{"type":"paragraph","attrs":{"localId":"30f90260-71e5-4caf-8227-463eee796d5a"},"content":[{"text":"Explicação adicional:","type":"text","marks":[{"type":"underline"}]}]}
```

```adf 
{"type":"paragraph","attrs":{"localId":"30f90260-71e5-4caf-8227-463eee796d5a"},"content":[{"text":"Explicação adicional:","type":"text","marks":[{"type":"underline"}]}]}
```

- OnInitializedAsync carrega os pedidos **sem travar a UI.**
- PlaceFakeAsync faz:
- chamada assíncrona para o BFF,
- recarrega listagem de forma assíncrona,
- mostra mensagens de status.
- ToggleAutoRefreshAsync:
- inicia um loop assíncrono em background no cliente (RunAutoRefreshLoopAsync),
- usa PeriodicTimer + CancellationToken,
- enquanto estiver ligado, a página vai atualizando automaticamente a cada 5 segundos sem interação do utilizador.
- DisposeAsync garante que o loop assíncrono pare quando a página for destruída/navegação mudar.


---

# **Client-side Composition**

O ecrã é montado no cliente, compondo componentes menores (layout + menu + features), em vez de vir “pronto” do servidor.

## **Exemplo:**

**Layout + composição de componentes:**

Shared/MainLayout.razor:

``` 
<div class="page">
  <aside>
    <Shared.NavMenu />
  </aside>
  <main>
    @Body
  </main>
</div>
```

Shared/NavMenu.razor:

``` 
<nav>
  <h4>Modular Monolith</h4>
  <a href="/">Home</a>
  <a href="/catalog">Catálogo</a>
  <a href="/orders">Pedidos</a>
</nav>
```

E uma página de feature:

``` 
@page "/catalog"
@inject CatalogService Service

<h3>Catálogo</h3>
<!-- conteúdo daqui pra baixo -->
```

Como funciona na prática:

1. O Router decide qual página renderizar (CatalogPage, OrdersPage, etc.).
2. O MainLayout envolve essa página com sidebar e header.
3. O NavMenu é outro componente, plugado no layout.
4. Tudo isso é **combinado no cliente **pelo runtime do Blazor.

Isso é client-side composition: montar a UI final no browser, compondo partes menores.

# **Modular Monolith**

Um único frontend SPA (um bundle, um deploy), mas organizado internamente em módulos de negócio bem isolados com módulos verticalizados, por domínio:

-Features/Catalog → tudo de catálogo

-Features/Orders → tudo de pedidos

-Features/Admin → tudo de administração

Cada módulo tem:

- suas páginas (.razor com @page "/catalog" etc.),
- seus serviços (ex.: CatalogService, OrdersService),
- seus componentes,
- opcionalmente sua navegação (links de menu, ícones, etc.).

Mas tudo corre na mesma SPA, no mesmo Program.cs, no mesmo App.razor.

## Exemplo:

Frontend/Program.cs

``` 
using Microsoft.AspNetCore.Components.WebAssembly.Hosting;
using Frontend.Features.Catalog;
using Frontend.Features.Orders;

var builder = WebAssemblyHostBuilder.CreateDefault(args);

builder.RootComponents.Add<App>("#app");

// HttpClient para chamar o BFF (URL de exemplo)
var bffBase = builder.Configuration["BffBaseUrl"] ?? "http://localhost:5005";
builder.Services.AddScoped(sp => new HttpClient { BaseAddress = new Uri(bffBase) });

// Registrar módulos de frontend
CatalogModule.RegisterServices(builder.Services);
OrdersModule.RegisterServices(builder.Services);

await builder.Build().RunAsync();
```

Frontend/App.razor

``` 
@using Microsoft.AspNetCore.Components.Routing

<Router AppAssembly="@typeof(App).Assembly">
    <Found Context="routeData">
        <RouteView RouteData="@routeData" DefaultLayout="@typeof(Shared.MainLayout)" />
        <FocusOnNavigate RouteData="@routeData" Selector="h1" />
    </Found>
    <NotFound>
        <Shared.MainLayout>
            <p role="alert">Página não encontrada.</p>
        </Shared.MainLayout>
    </NotFound>
</Router>
```

Features/Catalog/CatalogModule.cs

``` 
// Frontend/Features/Catalog/CatalogModule.cs
using Microsoft.Extensions.DependencyInjection;

namespace Frontend.Features.Catalog;

public static class CatalogModule
{
    public static void RegisterServices(IServiceCollection services)
    {
        services.AddScoped<CatalogService>();
    }
}
```

Features/Catalog/CatalogService.cs

``` 
// Frontend/Features/Catalog/CatalogService.cs
using System.Net.Http.Json;

namespace Frontend.Features.Catalog;

public sealed class CatalogService
{
    private readonly HttpClient _http;

    public CatalogService(HttpClient http)
    {
        _http = http;
    }

    public async Task<IReadOnlyList<CatalogItemDto>> ListAsync(CancellationToken ct = default)
    {
        // Chamando BFF, mas poderia ser qualquer fonte
        var items = await _http.GetFromJsonAsync<IReadOnlyList<CatalogItemDto>>(
            "/api/catalog/items",
            ct);

        return items ?? Array.Empty<CatalogItemDto>();
    }
}

public sealed record CatalogItemDto(Guid Id, string Name, decimal Price);
```

Features/Catalog/Pages/CatalogPage.razor

``` 
@page "/catalog"
@inject Frontend.Features.Catalog.CatalogService Service

<h3>Catálogo</h3>

@if (_isLoading)
{
    <p>Carregando...</p>
}
else if (!string.IsNullOrWhiteSpace(_errorMessage))
{
    <p style="color:red">@_errorMessage</p>
}
else if (_items is null || _items.Count == 0)
{
    <p>Nenhum item encontrado.</p>
}
else
{
    <table class="catalog-table">
        <thead>
        <tr>
            <th>Nome</th>
            <th>Preço</th>
        </tr>
        </thead>
        <tbody>
        @foreach (var i in _items)
        {
            <tr>
                <td>@i.Name</td>
                <td>@i.Price.ToString("C")</td>
            </tr>
        }
        </tbody>
    </table>
}

@code {
    private IReadOnlyList<CatalogItemDto>? _items;
    private bool _isLoading;
    private string? _errorMessage;

    protected override async Task OnInitializedAsync()
    {
        _isLoading = true;
        _errorMessage = null;

        try
        {
            _items = await Service.ListAsync();
        }
        catch (Exception ex)
        {
            _items = Array.Empty<CatalogItemDto>();
            _errorMessage = $"Erro ao carregar catálogo: {ex.Message}";
        }
        finally
        {
            _isLoading = false;
        }
    }
}

<style>
.catalog-table {
    border-collapse: collapse;
    min-width: 320px;
}
.catalog-table th,
.catalog-table td {
    border: 1px solid #ddd;
    padding: .4rem .6rem;
}
.catalog-table thead {
    background-color: #f5f5f5;
}
</style>
```

# **Clean Architecture** 

## **Visão Geral**

**Clean Architecture** é um estilo arquitetural proposto por **Robert C. Martin (Uncle Bob)** cujo objetivo principal é criar sistemas **independentes de frameworks**, **testáveis**, **manuteníveis** e **fáceis de evoluir ao longo do tempo**.

A ideia central é organizar o software de forma que **as regras de negócio sejam o núcleo do sistema**, enquanto detalhes técnicos (UI, base de dados, frameworks, mensageria) fiquem na periferia.

Em Clean Architecture, **políticas de alto nível não dependem de detalhes de baixo nível**. Ao contrário, os detalhes é que dependem das regras de negócio.

## **Princípio Fundamental**

> **Dependências sempre apontam para dentro.**

Isso significa que:

- A camada de domínio **não conhece** base de dados
- Não conhece frameworks web
- Não conhece filas, brokers ou UI
- Não conhece ORM, REST, GraphQL ou gRPC

Ela conhece apenas **regras de negócio puras**.

## **Estrutura em Camadas**

Visualmente, a Clean Architecture é representada como **círculos concêntricos**:

1. **Entities**
2. **Use Cases (Application)**
3. **Interface Adapters**
4. **Frameworks & Drivers**

Cada camada externa depende apenas das camadas internas.

## **Entities (Domínio)**

### **Responsabilidade**

- Representar as **regras de negócio mais estáveis**
- Conter invariantes do domínio
- Ser completamente independente de tecnologia

Exemplos:

- Order
- Customer
- Account

Características:

- Código puro
- Sem anotações de framework
- Fácil de testar
- Alta longevidade

## **Use Cases (Application Layer)**

### **Responsabilidade**

- Orquestrar regras de negócio
- Representar **casos de uso do sistema**
- Coordenar entidades e portas (interfaces)

Exemplos:

- CreateOrder
- TransferMoney
- RegisterUser

Aqui ficam:

- Regras de aplicação
- Fluxos
- Validações de negócio que não pertencem a uma única entidade

Os use cases **não sabem**:

- Como os dados são persistidos
- Como a API é exposta
- Quem chama o caso de uso

## **Interface Adapters**

### **Responsabilidade**

- Adaptar o mundo externo para o modelo interno
- Converter dados entre formatos externos e internos

Inclui:

- Controllers (REST, GraphQL, gRPC)
- Presenters / ViewModels
- Repositórios (implementações)
- Gateways

Essa camada:

- Implementa interfaces definidas nos Use Cases
- Traduz DTOs ↔ Entidades

## **Frameworks & Drivers**

### **Responsabilidade**

- Lidar com detalhes técnicos
- Infraestrutura e ferramentas

Exemplos:

- Spring Boot / [http://ASP.NET](http://ASP.NET)  Core
- React / Angular
- base de dados
- ORM (Hibernate, EF Core)
- Brokers (Kafka, RabbitMQ)

Essa camada **é descartável**.

Se amanhã você trocar Spring por outro framework, o domínio não muda.

## **Inversão de Dependência**

Clean Architecture se apoia fortemente no **Dependency Inversion Principle (DIP)**:

- Interfaces são definidas nas camadas internas
- Implementações ficam nas camadas externas

Exemplo:

- OrderRepository definido no domínio ou application
- JpaOrderRepository implementado na infraestrutura

Isso garante:

- Testes sem BD
- Troca fácil de tecnologia
- Baixo acoplamento

## **Clean Architecture vs MVC Tradicional**

| Aspecto | MVC Tradicional (Data-Centric) | Clean Architecture |
| --- | --- | --- |
| **Núcleo (Domínio)** | Dependente de frameworks e bancos de dados. | **Independente**de detalhes técnicos (Plain Objects). |
| **Direção das Dependências** | **Acoplamento Direto**(Domínio → Infraestrutura). | **Inversão de Dependência**(Infraestrutura → Domínio). |
| **Lógica de Negócio** | Muitas vezes fragmentada entre Controllers e Services. | **Centralizada** e protegida em Entidades e Casos de Uso. |
| **Testabilidade** | **Moderada** (frequentemente exige mocks de DB ou integração). | **Alta** (testes unitários puros, sem IO ou infra). |
| **Portabilidade** | Difícil (trocar o DB ou Framework exige refatorar o Core). | **Facilitada** (o Core não conhece o mundo externo). |


## **Clean Architecture e Hexagonal Architecture**

Clean Architecture e **Arquitetura Hexagonal (Ports and Adapters)** são conceitos muito próximos:

- Ambas isolam o domínio
- Ambas usam portas e adaptadores
- Diferença principal é **ênfase e representação**

Hexagonal foca mais na ideia de portas/adaptadores

Clean Architecture foca mais em **políticas vs detalhes**

Na prática, muitos sistemas combinam as duas.

## **Clean Architecture e Microserviços**

Clean Architecture funciona muito bem em:

- Monólitos bem estruturados
- Monólitos modulares
- Microserviços

Em microserviços:

- Cada serviço tem seu próprio domínio isolado
- Facilita testes
- Reduz dependência entre times
- Evita que o framework “vaze” para o core

## **Benefícios da Clean Architecture**

- **Testabilidade extrema**
- **Baixo acoplamento**
- **Alta coesão**
- **Evolução segura**
- **Framework-agnostic**
- **Código orientado ao negócio**

## **Desafios e Armadilhas**

### **Overengineering**

- Pode ser exagero para sistemas pequenos
- Requer disciplina

### **Curva de Aprendizado**

- Times acostumados com MVC tendem a misturar camadas

### **Excesso de Abstrações**

- Interfaces sem necessidade real
- Camadas artificiais

## **Quando Usar Clean Architecture**

Indicada quando:

- O sistema é grande ou tende a crescer
- Regras de negócio são complexas
- Longevidade é importante
- Há múltiplas interfaces (API, batch, eventos)

Não indicada quando:

- Projeto é pequeno e descartável
- Time é muito júnior sem acompanhamento
- Time-to-market é extremamente curto

## **Conclusão**

Clean Architecture não é sobre pastas ou frameworks, mas sobre **proteção do domínio**.

Ela força decisões arquiteturais que colocam o negócio no centro e empurram detalhes técnicos para as bordas.

Quando bem aplicada, resulta em sistemas mais fáceis de testar, manter e evoluir — mesmo após anos de mudanças tecnológicas.


---

# **CQRS (Command Query Responsibility Segregation)**

## **Visão Geral**

**CQRS (Command Query Responsibility Segregation)** é um padrão arquitetural que propõe a **separação explícita entre operações de escrita (Commands)** e **operações de leitura (Queries)** em um sistema.

Enquanto arquiteturas tradicionais usam o mesmo modelo para ler e escrever dados, o CQRS reconhece que **leitura e escrita têm necessidades muito diferentes** e, por isso, devem ser tratadas de forma distinta.

O princípio central é simples:

> **Commands mudam estado. Queries apenas leem estado. Nunca os dois.**

## **Conceitos Fundamentais**

### **Command**

Um **Command** representa uma intenção de mudança no sistema.

- Expressa *o intuito do utilizador*
- Sempre tenta alterar o estado
- Não retorna dados (no máximo um status ou ID)

Exemplos:

- CreateOrder
- CancelOrder
- UpdateCustomerAddress

Características:

- Verbos no imperativo
- Validação forte
- Executados de forma transacional

### **Query**

Uma **Query** representa uma solicitação de leitura.

- Nunca altera estado
- Retorna dados
- Pode ser altamente otimizada para leitura

Exemplos:

- GetOrderById
- ListOrdersByCustomer
- SearchProducts

### **Command Model (Write Model)**

O **modelo de escrita** é responsável por:

- Garantir regras de negócio
- Manter consistência
- Executar validações

Normalmente:

- Usa entidades ricas
- Segue princípios de DDD
- Prioriza integridade sobre performance

### **Query Model (Read Model)**

O **modelo de leitura** é responsável por:

- Entregar dados rapidamente
- Atender necessidades específicas da UI
- Suportar consultas complexas

Normalmente:

- Usa DTOs simples
- Pode ter modelos diferentes por ecrã
- Pode usar BDs ou índices diferentes


---

## **Fluxo Básico de CQRS**

1. O cliente envia um **Command**
2. O Command Handler valida e executa regras de negócio
3. O estado é persistido
4. Um ou mais eventos são publicados
5. O Read Model é atualizado (sincronamente ou não)
6. O cliente executa **Queries** contra o Read Model

Leitura e escrita seguem caminhos totalmente independentes.

## **CQRS Simples vs CQRS Completo**

### **CQRS Simples**

- Mesma base de dados
- Modelos diferentes para leitura e escrita
- Separação lógica, não física

Muito comum e recomendado como primeiro passo.

### **CQRS Completo**

- BDs separados
- Atualização do Read Model via eventos
- Consistência eventual

Mais complexo, porém altamente escalável.

## **CQRS e Event-Driven Architecture**

CQRS é frequentemente combinado com **Event-Driven Architecture**:

- Commands geram eventos
- Eventos atualizam Read Models
- Consumidores independentes processam projeções

Exemplo:

- OrderPlaced → atualiza visão de pedidos
- PaymentConfirmed → atualiza dashboard financeiro

Essa combinação é extremamente poderosa em sistemas distribuídos.

## **CQRS e Event Sourcing**

Em uma abordagem mais avançada:

- Commands geram eventos
- Eventos são a fonte da verdade
- Read Models são projeções desses eventos

Benefícios:

- Auditoria completa
- Time travel
- Rebuild de projeções

Custo:

- Complexidade elevada
- Requer maturidade técnica

## **Benefícios do CQRS**

- **Escalabilidade independente** de leitura e escrita
- **Modelos otimizados** para cada caso
- **Melhor desempenho** em cenários de leitura intensa
- **Clareza de intenção** no código
- **Facilidade de integração** com eventos

## **Desafios e Armadilhas**

### **Complexidade**

- Mais camadas
- Mais código
- Mais pontos de falha

### **Consistência Eventual**

- Leitura pode não refletir escrita imediatamente
- UI deve lidar com estados intermediários

### **Overengineering**

- Desnecessário para CRUD simples
- Pode atrasar o time-to-market

## **CQRS vs CRUD Tradicional**

| **Aspecto**         | **CRUD Tradicional** | **CQRS**     |
| ------------------- | -------------------- | ------------ |
| Modelo              | Único                | Separado     |
| Leitura             | Acoplada à escrita   | Independente |
| Escalabilidade      | Limitada             | Alta         |
| Complexidade        | Baixa                | Média/Alta   |
| Clareza de intenção | Média                | Alta         |

## **CQRS e Clean Architecture**

CQRS se encaixa naturalmente em **Clean Architecture**:

- Commands e Queries vivem na camada de Application
- Domínio protege regras de negócio
- Infraestrutura implementa handlers, repositórios e projeções

A separação reforça o isolamento do domínio e melhora testabilidade.

## **Quando Usar CQRS**

Indicado quando:

- Leitura é muito mais frequente que escrita
- Consultas são complexas
- Há necessidade de escalabilidade
- O domínio possui regras ricas

Não indicado quando:

- Aplicação é simples
- CRUD básico resolve o problema
- Time não tem experiência com sistemas distribuídos

## **Exemplo de Uso Prático**

Sistema de e-commerce:

- Commands:
- PlaceOrder
- CancelOrder
- Queries:
- GetOrderDetails
- ListOrdersByStatus

Modelo de escrita garante:

- Estoque
- Pagamento
- Regras de cancelamento

Modelo de leitura entrega:

- Listagens rápidas
- Dashboards
- Relatórios

## **Conclusão**

CQRS não é um requisito, é uma **opção estratégica**. Ele traz clareza, escalabilidade e flexibilidade, mas cobra um preço em complexidade.

Quando aplicado nos contextos certos — especialmente combinado com Event-Driven Architecture e Clean Architecture — o CQRS se torna uma poderosa ferramenta para construir sistemas robustos, performáticos e preparados para crescer.


---

# **Design for Failure**

A arquitetura é projetada para ser resiliente a falhas. O uso de filas de *dead-letter*, políticas de *retry* com *exponential backoff* e a natureza assíncrona da comunicação garantem que o sistema possa se recuperar de falhas temporárias sem perda de dados.


---

# **Infraestrutura como Código (IaC)**

Toda a infraestrutura é definida em código (Terraform), o que garante a reprodutibilidade, o controle de versão e a automação do provisionamento de ambientes.


---
