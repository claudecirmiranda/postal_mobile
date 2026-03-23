Índice
------

- [DAS Backend API Messaging — Documento de Apoio](#das-backend-api-messaging-documento-de-apoio)
- [4.3. Estratégia de Testes](#43-estratégia-de-testes)
	- [4.3.1. Testes Unitários](#431-testes-unitários)
	- [4.3.2. Testes de Integração](#432-testes-de-integração)
	- [4.3.3. Testes de Contrato](#433-testes-de-contrato)
	- [4.3.4. Testes de Performance](#434-testes-de-performance)
	- [4.3.5. Exemplos de Testes de Unidade](#435-exemplos-de-testes-de-unidade)
		- [4.3.5.1. Teste de Entidade de Domínio](#4351-teste-de-entidade-de-domínio)
		- [4.3.5.2. Teste de Caso de Uso (Application)](#4352-teste-de-caso-de-uso-application)
		- [4.3.5.3. Teste de Envio de Evento](#4353-teste-de-envio-de-evento)
	- [4.3.6. Testcontainers — Exemplos Práticos](#436-testcontainers-exemplos-práticos)
		- [4.3.6.1. Exemplo de Teste de Repositório com PostgreSQL Real](#4361-exemplo-de-teste-de-repositório-com-postgresql-real)
		- [4.3.6.2. Exemplo de Teste de Integração com Wolverine Tracking](#4362-exemplo-de-teste-de-integração-com-wolverine-tracking)
		- [4.3.6.3. Exemplo de Teste End-to-End](#4363-exemplo-de-teste-end-to-end)
	- [4.5.3. Circuit Breaker](#453-circuit-breaker)
	- [4.5.4. Outbox Pattern](#454-outbox-pattern)
- [5. Estrutura do Projeto — Exemplos de Implementação de Camadas](#5-estrutura-do-projeto-exemplos-de-implementação-de-camadas)
	- [5.1. Camada Domain — Núcleo do Sistema](#51-camada-domain-núcleo-do-sistema)
	- [5.2. Camada Application — Casos de Uso](#52-camada-application-casos-de-uso)
	- [5.3. Camada Infrastructure — Implementações Técnicas](#53-camada-infrastructure-implementações-técnicas)
	- [5.4. Camada API — Apresentação](#54-camada-api-apresentação)

# DAS Backend API Messaging — Documento de Apoio

> **Âmbito deste documento:** Exemplos práticos de implementação para o DAS Backend API Messaging. Consultar o documento principal para diretrizes arquiteturais, fronteiras de camadas e padrões de design.
>
> **Nota:** `Modelo` é um placeholder nos exemplos de código — substituir pelo nome do bounded context real (ex.: `Pedidos`, `Faturacao`, `Catalogo`).

## 4.3. Estratégia de Testes

### 4.3.1. Testes Unitários

- xUnit + FluentAssertions + Moq.
- Foco em:
  - Regras de domínio (Domain).
  - Casos de uso (Application).
  - Mocks para repositórios e serviços externos.

### 4.3.2. Testes de Integração

- Validar integração real com dependências técnicas:
  - EF Core + PostgreSQL (via Testcontainers).
  - API → Application → Infrastructure → DB.
  - (Quando aplicável) integração com mensageria via *test harness* / stubs / ambiente de testes.

> **Boas práticas com Testcontainers:** Cada teste deve ser responsável por criar e limpar os seus próprios dados. Evitar partilha de estado entre testes para garantir isolamento e repetibilidade.

### 4.3.3. Testes de Contrato

- **REST:** Pact (consumer-driven contracts) para contratos entre serviços.
- **Eventos:** validar schema/versão (ex.: JSON Schema/Avro/Protobuf) e regras de compatibilidade.

> **Versionamento de eventos em testes de contrato:** Ao evoluir o schema de um evento, incluir testes que validem compatibilidade retroativa — ou seja, que consumidores da versão anterior continuam a processar corretamente mensagens da versão nova. Schemas aditivos devem ser aceites sem erro; breaking changes devem ser detectadas em tempo de CI.

### 4.3.4. Testes de Performance

- k6/JMeter para cenários críticos (login, checkout, criação de pedido, etc.).
- Medir e acompanhar regressões:
  - Latência p95/p99 por endpoint/operação.
  - Throughput.
  - Taxa de erro e timeouts.

> **Critérios de aceitação:** Definir thresholds mínimos por cenário (ex.: p95 < 300 ms para criação de pedido, taxa de erro < 0,1%). Os testes de performance devem falhar o pipeline quando esses limites forem excedidos.

### 4.3.5. Exemplos de Testes de Unidade

#### 4.3.5.1. Teste de Entidade de Domínio

```csharp
using FluentAssertions;
using Modelo.Domain.Entities;
using Xunit;

public class OrderTests
{
    [Fact]
    public void Should_Create_Order_With_Valid_Data()
    {
        var customerId = Guid.NewGuid();
        decimal total = 150.75m;

        var order = new Order(customerId, total);

        order.Id.Should().NotBeEmpty();
        order.CustomerId.Should().Be(customerId);
        order.TotalAmount.Should().Be(total);
        order.CreatedAtUtc.Should().BeCloseTo(DateTime.UtcNow, TimeSpan.FromSeconds(2));
    }
}
```

#### 4.3.5.2. Teste de Caso de Uso (Application)

```csharp
using FluentAssertions;
using Wolverine;
using Modelo.Application.Commands.Orders;
using Modelo.Application.Handlers.Orders;
using Modelo.Application.Interfaces;
using Modelo.Domain.Entities;
using Modelo.Domain.Interfaces;
using Moq;
using Xunit;

public class CreateOrderCommandHandlerTests
{
    [Fact]
    public async Task Should_Create_Order_And_Publish_Event()
    {
        var repository = new Mock<IOrderRepository>();
        var unitOfWork = new Mock<IUnitOfWork>();
        var publisher = new Mock<IEventPublisher>();

        var handler = new CreateOrderCommandHandler(
            repository.Object,
            unitOfWork.Object,
            publisher.Object
        );

        var command = new CreateOrderCommand(Guid.NewGuid(), 250);

        var orderId = await handler.Handle(command, CancellationToken.None);

        repository.Verify(r => r.AddAsync(It.IsAny<Order>(), It.IsAny<CancellationToken>()), Times.Once);
        unitOfWork.Verify(u => u.CommitAsync(It.IsAny<CancellationToken>()), Times.Once);
        publisher.Verify(p => p.PublishAsync(
            "modelo.orders.order-created.v1",
            It.IsAny<OrderCreatedEvent>(),
            It.IsAny<CancellationToken>()),
            Times.Once
        );

        orderId.Should().NotBeEmpty();
    }
}
```

#### 4.3.5.3. Teste de Envio de Evento

```csharp
using System;
using System.Threading;
using System.Threading.Tasks;
using FluentAssertions;
using Moq;
using Xunit;
using Modelo.Application.Interfaces;

public class EventPublisherTests
{
    [Fact]
    public async Task Should_Publish_Event_To_Topic()
    {
        // Arrange
        var publisher = new Mock<IEventPublisher>(MockBehavior.Strict);

        var topic = "modelo.orders.order-created.v1";
        var evt = new
        {
            EventId = Guid.NewGuid(),
            OrderId = Guid.NewGuid(),
            OccurredAtUtc = DateTime.UtcNow
        };

        publisher
            .Setup(p => p.PublishAsync(topic, It.IsAny<object>(), It.IsAny<CancellationToken>()))
            .Returns(Task.CompletedTask);

        // Act
        Func<Task> act = async () =>
            await publisher.Object.PublishAsync(topic, evt, CancellationToken.None);

        // Assert
        await act.Should().NotThrowAsync();

        publisher.Verify(p =>
            p.PublishAsync(
                topic,
                It.Is<object>(x => x != null),
                It.IsAny<CancellationToken>()),
            Times.Once);

        publisher.VerifyNoOtherCalls();
    }
}
```

### 4.3.6. Testcontainers — Exemplos Práticos

A seguir, exemplos prontos para uso no padrão da arquitetura.

#### 4.3.6.1. Exemplo de Teste de Repositório com PostgreSQL Real

```csharp
using Xunit;
using Testcontainers.PostgreSql;
using Microsoft.EntityFrameworkCore;
using Modelo.Infrastructure.Context;
using Modelo.Infrastructure.Repositories;
using Modelo.Domain.Entities;

public class OrderRepositoryTests : IAsyncLifetime
{
    private readonly PostgreSqlContainer _pgContainer;
    private AppDbContext _context = default!;

    public OrderRepositoryTests()
    {
        _pgContainer = new PostgreSqlBuilder()
            .WithImage("postgres:16")
            .WithDatabase("testdb")
            .WithUsername("test")
            .WithPassword("test")
            .Build();
    }

    public async Task InitializeAsync()
    {
        await _pgContainer.StartAsync();

        var options = new DbContextOptionsBuilder<AppDbContext>()
            .UseNpgsql(_pgContainer.GetConnectionString())
            .Options;

        _context = new AppDbContext(options);
        await _context.Database.EnsureCreatedAsync();
    }

    public async Task DisposeAsync()
    {
        await _pgContainer.DisposeAsync();
    }

    [Fact]
    public async Task Should_Insert_And_Read_Order()
    {
        // Arrange — dados criados por este teste, limpos no DisposeAsync
        var repo = new OrderRepository(_context);
        var order = new Order(Guid.NewGuid(), 300);

        // Act
        await repo.AddAsync(order);
        await _context.SaveChangesAsync();
        var saved = await repo.GetByIdAsync(order.Id);

        // Assert
        Assert.NotNull(saved);
        Assert.Equal(order.TotalAmount, saved!.TotalAmount);
    }
}
```

> **Nota sobre segredos em CI/CD:** Em pipelines CI/CD, as credenciais dos containers de teste (ex.: `WithUsername`, `WithPassword`) devem ser injetadas via variáveis de ambiente ou secrets do pipeline — nunca hard-coded em repositórios. As credenciais acima são apenas para uso local/ilustrativo.

#### 4.3.6.2. Exemplo de Teste de Integração com Wolverine Tracking

```csharp
using Xunit;
using Wolverine;
using Wolverine.Tracking;
using Microsoft.Extensions.DependencyInjection;

public class WolverineTrackingTests : IAsyncLifetime
{
    private IServiceProvider _provider = default!;

    public async Task InitializeAsync()
    {
        var services = new ServiceCollection();

        services.AddWolverine(opts =>
        {
            // Exemplo: desabilitar roteamento convencional se quiser algo mais explícito
            opts.DisableConventionalLocalRouting();
        });

        _provider = services.BuildServiceProvider();

        await Task.CompletedTask;
    }

    public async Task DisposeAsync()
    {
        if (_provider is IDisposable d)
            d.Dispose();

        await Task.CompletedTask;
    }

    [Fact]
    public async Task OrderCreatedEvent_Should_Execute_Handler()
    {
        var bus = _provider.GetRequiredService<IMessageBus>();

        var evt = new OrderCreatedEvent
        {
            OrderId = Guid.NewGuid(),
            CustomerId = Guid.NewGuid(),
            TotalAmount = 150
        };

        var report = await bus.TrackAsync(async session =>
        {
            await session.Send(evt);
        });

        Assert.True(report.Executed.Any());
    }
}
```

#### 4.3.6.3. Exemplo de Teste End-to-End

> **Nota:** O exemplo abaixo é uma estrutura base para testes end-to-end com Testcontainers. O `Assert.True(true)` é apenas um placeholder — deve ser substituído pela validação real do fluxo completo (ex.: verificar que um pedido criado via API é persistido na base de dados e que o evento correspondente é publicado). Testes end-to-end superficiais criam falsa perceção de cobertura.

```csharp
public class FullIntegrationTests : IAsyncLifetime
{
    private readonly PostgreSqlContainer _pg =
        new PostgreSqlBuilder().WithDatabase("integration").Build();

    public async Task InitializeAsync()
    {
        await _pg.StartAsync();
        // Aqui deve ser configurado o host completo da aplicação (ex.: WebApplicationFactory)
        // e, quando aplicável, stubs ou test harness do broker Solace
    }

    public async Task DisposeAsync()
    {
        await _pg.DisposeAsync();
    }

    [Fact]
    public async Task Full_Flow_Should_Work()
    {
        // TODO: implementar validação real do fluxo API → Application → Repository
        // Exemplo mínimo esperado:
        // 1. POST /api/orders com payload válido
        // 2. Verificar resposta 201 com ID gerado
        // 3. Verificar registo na base de dados via repositório
        // 4. Verificar publicação do evento (via test harness ou mock de IEventPublisher)
        Assert.Fail("Substituir por implementação real do teste end-to-end.");
    }
}
```

---

### 4.5.3. Circuit Breaker

Exemplo de Circuit Breaker com retry e timeout utilizando **Polly**.

> **Nota:** Polly é utilizado aqui como exemplo ilustrativo do padrão Circuit Breaker. A implementação pode utilizar qualquer mecanismo que garanta os mesmos comportamentos de resiliência (estados Closed/Open/Half-Open, fail fast, métricas expostas).

```csharp
using Polly;
using Polly.CircuitBreaker;
using Polly.Timeout;
using System;
using System.Net.Http;
using System.Threading;
using System.Threading.Tasks;

public class ExternalApiClient
{
    private readonly HttpClient _httpClient;
    private readonly AsyncPolicy<HttpResponseMessage> _policy;

    public ExternalApiClient(HttpClient httpClient)
    {
        _httpClient = httpClient;

        var timeoutPolicy =
            Policy.TimeoutAsync<HttpResponseMessage>(
                TimeSpan.FromSeconds(2)
            );

        var retryPolicy =
            Policy<HttpResponseMessage>
                .Handle<HttpRequestException>()
                .OrResult(r => !r.IsSuccessStatusCode)
                .WaitAndRetryAsync(
                    retryCount: 3,
                    sleepDurationProvider: attempt =>
                        TimeSpan.FromMilliseconds(200 * Math.Pow(2, attempt))
                );

        var circuitBreakerPolicy =
            Policy<HttpResponseMessage>
                .Handle<HttpRequestException>()
                .OrResult(r => !r.IsSuccessStatusCode)
                .CircuitBreakerAsync(
                    handledEventsAllowedBeforeBreaking: 5,
                    durationOfBreak: TimeSpan.FromSeconds(30),
                    onBreak: (outcome, breakDelay) =>
                    {
                        Console.WriteLine($"Circuit OPEN for {breakDelay.TotalSeconds}s");
                    },
                    onReset: () =>
                    {
                        Console.WriteLine("Circuit CLOSED again");
                    },
                    onHalfOpen: () =>
                    {
                        Console.WriteLine("Circuit HALF-OPEN");
                    });

        _policy = Policy.WrapAsync(
            retryPolicy,
            circuitBreakerPolicy,
            timeoutPolicy
        );
    }

    public async Task<string> GetDataAsync(CancellationToken ct)
    {
        var response = await _policy.ExecuteAsync(
            async token =>
            {
                return await _httpClient.GetAsync(
                    "https://api.externa.com/data",
                    token
                );
            },
            ct
        );

        response.EnsureSuccessStatusCode();
        return await response.Content.ReadAsStringAsync();
    }
}
```

Uso em consumidor de mensageria com fallback para DLQ:

```csharp
public async Task HandleMessageAsync(string message)
{
    try
    {
        await _policy.ExecuteAsync(async () =>
        {
            await CallExternalServiceAsync(message);
        });
    }
    catch (BrokenCircuitException)
    {
        // Fail fast → enviar para retry topic ou DLQ
        await SendToDlqAsync(message);
    }
}
```

### 4.5.4. Outbox Pattern

Exemplo ilustrativo do Outbox Pattern.

> **Nota sobre concorrência no dispatcher:** Em ambientes com múltiplas instâncias do worker a correr em paralelo, a mesma mensagem pode ser lida e enviada duplicadamente se não houver coordenação. Para evitar este cenário, utilizar `SELECT ... FOR UPDATE SKIP LOCKED` no PostgreSQL (ou equivalente no SGBD escolhido) ao consultar os registos pendentes. Alternativamente, usar uma fila distribuída para coordenar o dispatch. O exemplo abaixo omite este mecanismo por simplicidade — adaptar para produção.

```csharp
// Modelo da tabela outbox
class OutboxMessage
{
    public Guid Id { get; init; } = Guid.NewGuid();
    public string Topic { get; init; } = default!;
    public string Payload { get; init; } = default!;  // JSON do evento
    public DateTime CreatedAt { get; init; } = DateTime.UtcNow;
    public DateTime? SentAt { get; set; }
}

// 1) Escrita atómica: grava domínio + outbox na mesma transação
async Task CreateOrderAsync(Order order, DbContext db)
{
    using var tx = await db.Database.BeginTransactionAsync();

    db.Add(order); // grava o agregado
    db.Add(new OutboxMessage
    {
        Topic = "orders.created",
        Payload = JsonSerializer.Serialize(new { order.Id, order.Total })
    });

    await db.SaveChangesAsync();
    await tx.CommitAsync(); // ou rollback em caso de erro
}

// 2) Dispatcher assíncrono: lê outbox e envia para o broker
// ATENÇÃO: em múltiplas instâncias, usar SELECT FOR UPDATE SKIP LOCKED
// para evitar processamento duplicado do mesmo registo.
async Task DispatchOutboxAsync(DbContext db, IMessageBus bus, CancellationToken ct)
{
    var pending = await db.Set<OutboxMessage>()
        .Where(m => m.SentAt == null)
        .OrderBy(m => m.CreatedAt)
        .Take(100)
        .ToListAsync(ct);

    foreach (var msg in pending)
    {
        try
        {
            await bus.PublishAsync(msg.Topic, msg.Payload, ct);
            msg.SentAt = DateTime.UtcNow;
        }
        catch
        {
            // Falhou → será reprocessado na próxima iteração (retry/backoff/log)
            // Nota: se o publish tiver sucesso mas o SaveChanges falhar,
            // a mensagem será reenviada. Os consumidores devem ser idempotentes.
        }
    }

    await db.SaveChangesAsync(ct);
}

// 3) Worker simples (loop periódico via BackgroundService na mesma aplicação)
async Task RunWorkerAsync(CancellationToken ct)
{
    while (!ct.IsCancellationRequested)
    {
        await DispatchOutboxAsync(db, bus, ct);
        await Task.Delay(TimeSpan.FromSeconds(5), ct); // intervalo de polling
    }
}
```

## 5. Estrutura do Projeto — Exemplos de Implementação de Camadas

### 5.1. Camada Domain — Núcleo do Sistema

Ficheiro: `Modelo.Domain/Entities/Order.cs`

```csharp
namespace Modelo.Domain.Entities;

public sealed class Order
{
    public Guid Id { get; private set; }
    public Guid CustomerId { get; private set; }
    public decimal TotalAmount { get; private set; }
    public DateTime CreatedAtUtc { get; private set; }

    private Order() { }

    public Order(Guid customerId, decimal totalAmount)
    {
        Id = Guid.NewGuid();
        CustomerId = customerId;
        TotalAmount = totalAmount;
        CreatedAtUtc = DateTime.UtcNow;
    }
}
```

**Contratos de repositório e unit of work**

Ficheiro: `Modelo.Domain/Interfaces/IRepository.cs`

```csharp
namespace Modelo.Domain.Interfaces;

public interface IRepository<T> where T : class
{
    Task<T?> GetByIdAsync(Guid id, CancellationToken ct = default);
    Task AddAsync(T entity, CancellationToken ct = default);
    void Update(T entity);
    void Remove(T entity);
}
```

Ficheiro: `Modelo.Domain/Interfaces/IUnitOfWork.cs`

```csharp
namespace Modelo.Domain.Interfaces;

public interface IUnitOfWork : IAsyncDisposable
{
    Task<int> SaveChangesAsync(CancellationToken ct = default);
}
```

**Evento de domínio**

Ficheiro: `Modelo.Domain/Events/OrderCreatedEvent.cs`

```csharp
namespace Modelo.Domain.Events;

public sealed class OrderCreatedEvent
{
    public Guid EventId { get; init; } = Guid.NewGuid();
    public DateTime OccurredAtUtc { get; init; } = DateTime.UtcNow;
    public string SchemaVersion { get; init; } = "v1";
    public string Source { get; init; } = "modelo.api";

    public Guid OrderId { get; init; }
    public Guid CustomerId { get; init; }
    public decimal TotalAmount { get; init; }
    public string Currency { get; init; } = "BRL";
}
```

### 5.2. Camada Application — Casos de Uso

**Command de criação de pedido**

Ficheiro: `Modelo.Application/Commands/Orders/CreateOrderCommand.cs`

```csharp
using Wolverine;

namespace Modelo.Application.Commands.Orders;

/// <summary>
/// Comando de criação de pedido.
/// </summary>
public sealed record CreateOrderCommand(Guid CustomerId, decimal TotalAmount);
```

**Handler do caso de uso**

Ficheiro: `Modelo.Application/Handlers/Orders/CreateOrderCommandHandler.cs`

```csharp
using Wolverine;
using Modelo.Application.Commands.Orders;
using Modelo.Application.Interfaces;
using Modelo.Domain.Entities;
using Modelo.Domain.Events;
using Modelo.Domain.Interfaces;

namespace Modelo.Application.Handlers.Orders;

/// <summary>
/// Handler de aplicação para criação de pedidos.
/// Descoberto automaticamente pelo Wolverine através do método Handle.
/// </summary>
public sealed class CreateOrderCommandHandler
{
    private readonly IOrderRepository _orders;
    private readonly IUnitOfWork _uow;
    private readonly IEventPublisher _publisher;

    public CreateOrderCommandHandler(
        IOrderRepository orders,
        IUnitOfWork uow,
        IEventPublisher publisher)
    {
        _orders = orders;
        _uow = uow;
        _publisher = publisher;
    }

    /// <summary>
    /// Executa a criação do pedido e publica o evento de integração
    /// OrderCreatedEvent após o commit da UoW.
    /// </summary>
    public async Task<Guid> Handle(CreateOrderCommand request, CancellationToken ct)
    {
        var order = new Order(request.CustomerId, request.TotalAmount);

        await _orders.AddAsync(order, ct);
        await _uow.CommitAsync(ct);

        var evt = new OrderCreatedEvent
        {
            OrderId = order.Id,
            CustomerId = order.CustomerId,
            TotalAmount = order.TotalAmount
        };

        await _publisher.PublishAsync(
            "modelo.orders.order-created.v1",
            evt,
            ct);

        return order.Id;
    }
}
```

**Abstração de mensageria usada pela Application**

Ficheiro: `Modelo.Application/Interfaces/IEventPublisher.cs`

```csharp
namespace Modelo.Application.Interfaces;

public interface IEventPublisher
{
    Task PublishAsync<TEvent>(string topic, TEvent @event, CancellationToken ct = default);
}
```

### 5.3. Camada Infrastructure — Implementações Técnicas

**DbContext + UoW**

Ficheiro: `Modelo.Infrastructure/Context/AppDbContext.cs`

```csharp
using Microsoft.EntityFrameworkCore;
using Modelo.Domain.Entities;
using Modelo.Domain.Interfaces;

namespace Modelo.Infrastructure.Context;

public class AppDbContext : DbContext, IUnitOfWork
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }

    public DbSet<Order> Orders => Set<Order>();

    public async Task<int> SaveChangesAsync(CancellationToken ct = default)
        => await base.SaveChangesAsync(ct);
}
```

**Repositório de Order**

Ficheiro: `Modelo.Infrastructure/Repositories/OrderRepository.cs`

```csharp
using Microsoft.EntityFrameworkCore;
using Modelo.Domain.Entities;
using Modelo.Domain.Interfaces;
using Modelo.Infrastructure.Context;

namespace Modelo.Infrastructure.Repositories;

public class OrderRepository : IOrderRepository
{
    private readonly AppDbContext _context;

    public OrderRepository(AppDbContext context)
    {
        _context = context;
    }

    public async Task<Order?> GetByIdAsync(Guid id, CancellationToken ct = default)
        => await _context.Orders.FirstOrDefaultAsync(o => o.Id == id, ct);

    public async Task AddAsync(Order entity, CancellationToken ct = default)
        => await _context.Orders.AddAsync(entity, ct);

    public void Update(Order entity) => _context.Orders.Update(entity);
    public void Remove(Order entity) => _context.Orders.Remove(entity);
}
```

**Registo de infraestrutura e mensageria**

Ficheiro: `Modelo.Infrastructure/IoC/DependencyInjection.cs`

```csharp
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Modelo.Application.Interfaces;
using Modelo.Domain.Interfaces;
using Modelo.Infrastructure.Context;
using Modelo.Infrastructure.Messaging.Solace;
using Modelo.Infrastructure.Repositories;

namespace Modelo.Infrastructure.IoC;

public static class DependencyInjection
{
    public static IServiceCollection AddInfrastructure(
        this IServiceCollection services,
        IConfiguration configuration)
    {
        services.AddDbContext<AppDbContext>(options =>
            options.UseNpgsql(configuration.GetConnectionString("DefaultConnection")));

        services.AddScoped<IOrderRepository, OrderRepository>();
        services.AddScoped<IUnitOfWork>(sp => sp.GetRequiredService<AppDbContext>());

        // Publicadores (pode alternar via configuração, se necessário)
        // services.AddSingleton<IEventPublisher, SolaceEventPublisher>();

        // Consumers (Hosted Services)
        services.AddHostedService<SolaceOrderCreatedConsumer>();

        return services;
    }
}
```

### 5.4. Camada API — Apresentação

**Program.cs**

Ficheiro: `Modelo.Api/Program.cs`

```csharp
using Wolverine;
using Modelo.Application.Commands.Orders;
using Modelo.Infrastructure.IoC;
using Microsoft.Extensions.Logging;

var builder = WebApplication.CreateBuilder(args);

// Logging nativo (Microsoft.Extensions.Logging)
builder.Logging.ClearProviders();
builder.Logging.AddConsole();
builder.Logging.AddDebug();

builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

builder.Services.AddWolverine(typeof(CreateOrderCommand).Assembly);
builder.Services.AddInfrastructure(builder.Configuration);

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.MapControllers();

app.Run();
```

**Controller**

Ficheiro: `Modelo.Api/Controllers/OrdersController.cs`

```csharp
using Wolverine;
using Microsoft.AspNetCore.Mvc;
using Modelo.Application.Commands.Orders;
using Modelo.Application.Queries.Orders;

namespace Modelo.Api.Controllers;

[ApiController]
[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    private readonly IMessageBus _bus;

    public OrdersController(IMessageBus bus)
    {
        _bus = bus;
    }

    /// <summary>
    /// Cria um novo pedido.
    /// </summary>
    /// <param name="command">Dados do pedido a ser criado.</param>
    /// <param name="ct">Token de cancelamento da requisição.</param>
    [HttpPost]
    [ProducesResponseType(StatusCodes.Status201Created)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    public async Task<IActionResult> Create(
        [FromBody] CreateOrderCommand command,
        CancellationToken ct)
    {
        if (!ModelState.IsValid)
            return ValidationProblem(ModelState);

        var id = await _bus.InvokeAsync<Guid>(command, ct);

        return CreatedAtAction(nameof(GetById), new { id }, new { id });
    }

    /// <summary>
    /// Obtém uma lista paginada de pedidos.
    /// </summary>
    [HttpGet]
    [ProducesResponseType(typeof(GetOrdersPagedResult), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    public async Task<IActionResult> GetPaged(
        [FromQuery] int page = 1,
        [FromQuery] int pageSize = 20,
        CancellationToken ct = default)
    {
        if (page < 1 || pageSize < 1 || pageSize > 200)
            return BadRequest(new { error = "Parâmetros de paginação inválidos." });

        var query = new GetOrdersQuery(page, pageSize);
        var result = await _bus.InvokeAsync<GetOrdersPagedResult>(query, ct);
        return Ok(result);
    }

    /// <summary>
    /// Obtém os detalhes de um pedido por identificador.
    /// </summary>
    /// <param name="id">Identificador do pedido.</param>
    [HttpGet("{id:guid}")]
    [ProducesResponseType(StatusCodes.Status200OK)]
    public IActionResult GetById(Guid id)
    {
        // TODO: consultar Application/Domain (ex.: via Query enviada ao Wolverine)
        return Ok(new { id });
    }
}
```
