Here's a comprehensive expansion of your ASP.NET learning note:

---

# 🌐 ASP.NET Core Mastery

## 📅 Learning Goals & Milestones

### Short-term Goals (1-3 months)
- [ ] Understand ASP.NET Core fundamentals
- [ ] Build first Web API
- [ ] Master dependency injection
- [ ] Implement authentication & authorization
- [ ] Learn Entity Framework Core basics
- [ ] Deploy first application to cloud

### Medium-term Goals (3-6 months)
- [ ] Master middleware pipeline
- [ ] Implement clean architecture
- [ ] Build microservices
- [ ] Learn advanced EF Core
- [ ] Implement caching strategies
- [ ] Master async/await patterns
- [ ] Build real-time features with SignalR

### Long-term Goals (6-12 months)
- [ ] Design scalable API architectures
- [ ] Master performance optimization
- [ ] Implement CQRS and Event Sourcing
- [ ] Build production-ready applications
- [ ] Contribute to ASP.NET Core open source
- [ ] Achieve expertise in cloud deployment (Azure/AWS)

---

## 🎯 Core Learning Resources

### Foundation & Introduction

#### ASP.NET Core Documentation (Official)
https://docs.microsoft.com/en-us/aspnet/core/?view=aspnetcore-5.0
- **Type:** Official Microsoft documentation
- **Level:** All levels
- **Coverage:** 
  - Fundamentals
  - Web API development
  - MVC patterns
  - Razor Pages
  - Security
  - Performance
  - Deployment
- **Study approach:** Use as primary reference, bookmark key sections
- **Latest version:** Keep updated with ASP.NET Core 8.0/9.0

**Key documentation sections:**
1. **Fundamentals** - Startup, middleware, configuration
2. **Web API** - RESTful services, routing, model binding
3. **Security** - Authentication, authorization, data protection
4. **Data Access** - Entity Framework Core
5. **Performance** - Caching, response compression
6. **Hosting & Deployment** - IIS, Docker, Azure

**Documentation study plan:**
- Week 1-2: Fundamentals
- Week 3-4: Web API basics
- Week 5-6: Data access & EF Core
- Week 7-8: Security & authentication
- Ongoing: Advanced topics as needed

---

## 🛣️ Routing & API Design

### Attribute Routing in Web API
https://docs.microsoft.com/en-us/aspnet/web-api/overview/web-api-routing-and-actions/attribute-routing-in-web-api-2
- **Focus:** Modern routing approach
- **Level:** Beginner to intermediate
- **Use cases:**
  - RESTful API design
  - Route constraints
  - Route templates
  - Versioning strategies

**Key concepts:**

#### Basic Attribute Routing
```csharp
[Route("api/[controller]")]
[ApiController]
public class ProductsController : ControllerBase
{
    [HttpGet]
    public IActionResult GetAll() { }
    
    [HttpGet("{id}")]
    public IActionResult GetById(int id) { }
    
    [HttpPost]
    public IActionResult Create([FromBody] Product product) { }
    
    [HttpPut("{id}")]
    public IActionResult Update(int id, [FromBody] Product product) { }
    
    [HttpDelete("{id}")]
    public IActionResult Delete(int id) { }
}
```

#### Advanced Routing Patterns
```csharp
// Route constraints
[HttpGet("{id:int:min(1)}")]

// Multiple routes
[HttpGet("active")]
[HttpGet("status/active")]

// Custom route names
[HttpGet("{id}", Name = "GetProduct")]

// Route templates with defaults
[HttpGet("{category=all}")]
```

#### API Versioning
```csharp
// URL versioning
[Route("api/v1/[controller]")]

// Header versioning
[HttpGet]
[MapToApiVersion("1.0")]

// Query string versioning
[Route("api/[controller]")]
[ApiVersion("1.0")]
[ApiVersion("2.0")]
```

**Best practices:**
- Use consistent naming conventions (plural nouns)
- Follow RESTful principles
- Keep routes simple and predictable
- Use route constraints for validation
- Plan versioning strategy early

---

## ⚠️ Error Handling & Consistency

### Consistent Error Responses
https://offby2.com/posts/004-consistent-error-response-format-asp-net-core-web-api/
- **Focus:** Standardized error handling
- **Level:** Intermediate
- **Benefits:**
  - Predictable API responses
  - Better client-side error handling
  - Professional API design
  - Debugging efficiency

**Implementation patterns:**

#### Standard Error Response Model
```csharp
public class ErrorResponse
{
    public int StatusCode { get; set; }
    public string Message { get; set; }
    public string Details { get; set; }
    public DateTime Timestamp { get; set; }
    public string Path { get; set; }
    public List<ValidationError> Errors { get; set; }
}

public class ValidationError
{
    public string Field { get; set; }
    public string Message { get; set; }
}
```

#### Global Exception Handler
```csharp
public class GlobalExceptionHandler : IExceptionHandler
{
    private readonly ILogger<GlobalExceptionHandler> _logger;

    public GlobalExceptionHandler(ILogger<GlobalExceptionHandler> logger)
    {
        _logger = logger;
    }

    public async ValueTask<bool> TryHandleAsync(
        HttpContext httpContext,
        Exception exception,
        CancellationToken cancellationToken)
    {
        _logger.LogError(exception, "An error occurred");

        var response = new ErrorResponse
        {
            StatusCode = GetStatusCode(exception),
            Message = exception.Message,
            Timestamp = DateTime.UtcNow,
            Path = httpContext.Request.Path
        };

        httpContext.Response.StatusCode = response.StatusCode;
        await httpContext.Response.WriteAsJsonAsync(response, cancellationToken);

        return true;
    }

    private int GetStatusCode(Exception exception) =>
        exception switch
        {
            NotFoundException => StatusCodes.Status404NotFound,
            ValidationException => StatusCodes.Status400BadRequest,
            UnauthorizedAccessException => StatusCodes.Status401Unauthorized,
            _ => StatusCodes.Status500InternalServerError
        };
}
```

#### Custom Exception Middleware
```csharp
public class ExceptionMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<ExceptionMiddleware> _logger;

    public ExceptionMiddleware(RequestDelegate next, ILogger<ExceptionMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        try
        {
            await _next(context);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Unhandled exception occurred");
            await HandleExceptionAsync(context, ex);
        }
    }

    private async Task HandleExceptionAsync(HttpContext context, Exception exception)
    {
        context.Response.ContentType = "application/json";
        context.Response.StatusCode = (int)HttpStatusCode.InternalServerError;

        var response = new ErrorResponse
        {
            StatusCode = context.Response.StatusCode,
            Message = "An error occurred processing your request.",
            Details = exception.Message
        };

        await context.Response.WriteAsJsonAsync(response);
    }
}
```

#### ProblemDetails (RFC 7807)
```csharp
// Use built-in ProblemDetails
services.AddProblemDetails();

// Custom problem details
public class CustomProblemDetails : ProblemDetails
{
    public string TraceId { get; set; }
    public Dictionary<string, object> Extensions { get; set; }
}
```

**Error handling strategy:**
- Use global exception handlers
- Implement custom exceptions for domain logic
- Log errors appropriately (different levels)
- Don't expose sensitive information
- Return appropriate HTTP status codes
- Include correlation IDs for tracking

---

## 💉 Dependency Injection Deep Dive

### Customize Dependency Injection
https://girishgodage.in/blog/customize-dependency-injection
- **Focus:** Advanced DI patterns
- **Level:** Intermediate to advanced
- **Topics:**
  - Service lifetimes
  - Custom implementations
  - Factory patterns
  - Decorators

**DI Fundamentals:**

#### Service Lifetimes
```csharp
// Transient: New instance every time
services.AddTransient<IEmailService, EmailService>();

// Scoped: One instance per request
services.AddScoped<IOrderRepository, OrderRepository>();

// Singleton: Single instance for application lifetime
services.AddSingleton<ICacheService, CacheService>();
```

#### Constructor Injection
```csharp
public class ProductService
{
    private readonly IProductRepository _repository;
    private readonly ILogger<ProductService> _logger;
    private readonly IMapper _mapper;

    public ProductService(
        IProductRepository repository,
        ILogger<ProductService> logger,
        IMapper mapper)
    {
        _repository = repository;
        _logger = logger;
        _mapper = mapper;
    }
}
```

#### Factory Pattern
```csharp
// Register factory
services.AddSingleton<IServiceFactory, ServiceFactory>();

public interface IServiceFactory
{
    IPaymentService CreatePaymentService(PaymentType type);
}

public class ServiceFactory : IServiceFactory
{
    private readonly IServiceProvider _provider;

    public ServiceFactory(IServiceProvider provider)
    {
        _provider = provider;
    }

    public IPaymentService CreatePaymentService(PaymentType type)
    {
        return type switch
        {
            PaymentType.CreditCard => _provider.GetRequiredService<CreditCardService>(),
            PaymentType.PayPal => _provider.GetRequiredService<PayPalService>(),
            _ => throw new ArgumentException("Invalid payment type")
        };
    }
}
```

#### Decorator Pattern
```csharp
// Original service
services.AddScoped<IOrderService, OrderService>();

// Decorator
services.Decorate<IOrderService, CachedOrderService>();
services.Decorate<IOrderService, LoggingOrderService>();

public class LoggingOrderService : IOrderService
{
    private readonly IOrderService _inner;
    private readonly ILogger<LoggingOrderService> _logger;

    public LoggingOrderService(IOrderService inner, ILogger<LoggingOrderService> logger)
    {
        _inner = inner;
        _logger = logger;
    }

    public async Task<Order> GetOrderAsync(int id)
    {
        _logger.LogInformation("Getting order {OrderId}", id);
        var result = await _inner.GetOrderAsync(id);
        _logger.LogInformation("Order {OrderId} retrieved", id);
        return result;
    }
}
```

#### Options Pattern
```csharp
// appsettings.json
{
  "EmailSettings": {
    "SmtpServer": "smtp.example.com",
    "Port": 587
  }
}

// Configuration class
public class EmailSettings
{
    public string SmtpServer { get; set; }
    public int Port { get; set; }
}

// Registration
services.Configure<EmailSettings>(
    configuration.GetSection("EmailSettings"));

// Usage
public class EmailService
{
    private readonly EmailSettings _settings;

    public EmailService(IOptions<EmailSettings> options)
    {
        _settings = options.Value;
    }
}
```

#### Named Services
```csharp
// Register multiple implementations
services.AddKeyedSingleton<INotificationService, EmailNotificationService>("email");
services.AddKeyedSingleton<INotificationService, SmsNotificationService>("sms");

// Usage
public class NotificationController : ControllerBase
{
    public NotificationController(
        [FromKeyedServices("email")] INotificationService emailService,
        [FromKeyedServices("sms")] INotificationService smsService)
    {
    }
}
```

**DI Best Practices:**
- Prefer constructor injection
- Avoid service locator anti-pattern
- Don't inject `IServiceProvider` directly
- Use appropriate lifetimes
- Keep constructors simple (max 3-5 dependencies)
- Use interfaces for testability

---

## 📦 Request/Response Wrappers

### JSON Wrapper Request/Response Pattern
https://gist.github.com/ricardo-miguel/5dd105937ca70fb7c91dc0efc4c807b9
- **Focus:** Standardized API responses
- **Level:** Intermediate
- **Benefits:**
  - Consistent response format
  - Metadata inclusion
  - Pagination support
  - Error handling integration

**Implementation:**

#### API Response Wrapper
```csharp
public class ApiResponse<T>
{
    public bool Success { get; set; }
    public T Data { get; set; }
    public string Message { get; set; }
    public List<string> Errors { get; set; }
    public ResponseMetadata Metadata { get; set; }

    public static ApiResponse<T> SuccessResponse(T data, string message = null)
    {
        return new ApiResponse<T>
        {
            Success = true,
            Data = data,
            Message = message ?? "Request completed successfully"
        };
    }

    public static ApiResponse<T> ErrorResponse(string message, List<string> errors = null)
    {
        return new ApiResponse<T>
        {
            Success = false,
            Message = message,
            Errors = errors ?? new List<string>()
        };
    }
}

public class ResponseMetadata
{
    public DateTime Timestamp { get; set; } = DateTime.UtcNow;
    public string RequestId { get; set; }
    public string Version { get; set; } = "1.0";
}
```

#### Pagination Wrapper
```csharp
public class PagedResponse<T> : ApiResponse<T>
{
    public int PageNumber { get; set; }
    public int PageSize { get; set; }
    public int TotalPages { get; set; }
    public int TotalRecords { get; set; }
    public bool HasNextPage => PageNumber < TotalPages;
    public bool HasPreviousPage => PageNumber > 1;

    public PagedResponse(T data, int pageNumber, int pageSize, int totalRecords)
    {
        Success = true;
        Data = data;
        PageNumber = pageNumber;
        PageSize = pageSize;
        TotalRecords = totalRecords;
        TotalPages = (int)Math.Ceiling(totalRecords / (double)pageSize);
    }
}
```

#### Result Filter for Automatic Wrapping
```csharp
public class ApiResponseWrapperFilter : IResultFilter
{
    public void OnResultExecuting(ResultExecutingContext context)
    {
        if (context.Result is ObjectResult objectResult)
        {
            var wrappedResponse = new ApiResponse<object>
            {
                Success = context.HttpContext.Response.StatusCode < 400,
                Data = objectResult.Value,
                Metadata = new ResponseMetadata
                {
                    RequestId = context.HttpContext.TraceIdentifier
                }
            };

            objectResult.Value = wrappedResponse;
        }
    }

    public void OnResultExecuted(ResultExecutedContext context) { }
}

// Register filter
services.AddControllers(options =>
{
    options.Filters.Add<ApiResponseWrapperFilter>();
});
```

#### Controller Usage
```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly IProductService _productService;

    public ProductsController(IProductService productService)
    {
        _productService = productService;
    }

    [HttpGet]
    public async Task<ActionResult<ApiResponse<IEnumerable<ProductDto>>>> GetAll(
        [FromQuery] PaginationParams paginationParams)
    {
        var products = await _productService.GetAllAsync(paginationParams);
        var totalCount = await _productService.GetTotalCountAsync();

        var response = new PagedResponse<IEnumerable<ProductDto>>(
            products,
            paginationParams.PageNumber,
            paginationParams.PageSize,
            totalCount
        );

        return Ok(response);
    }

    [HttpGet("{id}")]
    public async Task<ActionResult<ApiResponse<ProductDto>>> GetById(int id)
    {
        var product = await _productService.GetByIdAsync(id);
        
        if (product == null)
            return NotFound(ApiResponse<ProductDto>.ErrorResponse("Product not found"));

        return Ok(ApiResponse<ProductDto>.SuccessResponse(product));
    }

    [HttpPost]
    public async Task<ActionResult<ApiResponse<ProductDto>>> Create(
        [FromBody] CreateProductDto dto)
    {
        var product = await _productService.CreateAsync(dto);
        
        return CreatedAtAction(
            nameof(GetById),
            new { id = product.Id },
            ApiResponse<ProductDto>.SuccessResponse(product, "Product created successfully")
        );
    }
}
```

**Wrapper benefits:**
- Uniform response structure
- Easy client-side parsing
- Metadata for tracking and debugging
- Simplified error handling
- Support for pagination
- Versioning support

---

## 🏗️ Architecture Patterns

### Clean Architecture

**Project structure:**
```
Solution/
├── Domain/                 # Core business logic
│   ├── Entities/
│   ├── Enums/
│   ├── Exceptions/
│   ├── Interfaces/
│   └── ValueObjects/
├── Application/            # Use cases & business rules
│   ├── DTOs/
│   ├── Interfaces/
│   ├── Mappings/
│   ├── Services/
│   └── Validators/
├── Infrastructure/         # External concerns
│   ├── Data/
│   ├── Identity/
│   ├── Services/
│   └── Repositories/
├── API/                    # Web API
│   ├── Controllers/
│   ├── Filters/
│   ├── Middleware/
│   └── Program.cs
└── Tests/
    ├── Unit/
    ├── Integration/
    └── E2E/
```

**Layer responsibilities:**

#### Domain Layer
```csharp
// Entity
public class Product
{
    public int Id { get; private set; }
    public string Name { get; private set; }
    public decimal Price { get; private set; }
    public ProductStatus Status { get; private set; }

    public void UpdatePrice(decimal newPrice)
    {
        if (newPrice <= 0)
            throw new DomainException("Price must be positive");
        
        Price = newPrice;
    }

    public void Activate()
    {
        Status = ProductStatus.Active;
    }
}

// Value Object
public class Money : ValueObject
{
    public decimal Amount { get; }
    public string Currency { get; }

    public Money(decimal amount, string currency)
    {
        Amount = amount;
        Currency = currency;
    }

    protected override IEnumerable<object> GetEqualityComponents()
    {
        yield return Amount;
        yield return Currency;
    }
}
```

#### Application Layer
```csharp
// DTO
public class ProductDto
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
}

// Service interface
public interface IProductService
{
    Task<ProductDto> GetByIdAsync(int id);
    Task<IEnumerable<ProductDto>> GetAllAsync(PaginationParams pagination);
    Task<ProductDto> CreateAsync(CreateProductDto dto);
    Task UpdateAsync(int id, UpdateProductDto dto);
    Task DeleteAsync(int id);
}

// Service implementation
public class ProductService : IProductService
{
    private readonly IProductRepository _repository;
    private readonly IMapper _mapper;

    public ProductService(IProductRepository repository, IMapper mapper)
    {
        _repository = repository;
        _mapper = mapper;
    }

    public async Task<ProductDto> GetByIdAsync(int id)
    {
        var product = await _repository.GetByIdAsync(id);
        return _mapper.Map<ProductDto>(product);
    }
}

// Validator (FluentValidation)
public class CreateProductValidator : AbstractValidator<CreateProductDto>
{
    public CreateProductValidator()
    {
        RuleFor(x => x.Name)
            .NotEmpty()
            .MaximumLength(200);

        RuleFor(x => x.Price)
            .GreaterThan(0);
    }
}
```

#### Infrastructure Layer
```csharp
// Repository
public class ProductRepository : IProductRepository
{
    private readonly AppDbContext _context;

    public ProductRepository(AppDbContext context)
    {
        _context = context;
    }

    public async Task<Product> GetByIdAsync(int id)
    {
        return await _context.Products
            .AsNoTracking()
            .FirstOrDefaultAsync(p => p.Id == id);
    }

    public async Task<IEnumerable<Product>> GetAllAsync(
        int pageNumber, 
        int pageSize)
    {
        return await _context.Products
            .AsNoTracking()
            .Skip((pageNumber - 1) * pageSize)
            .Take(pageSize)
            .ToListAsync();
    }

    public async Task AddAsync(Product product)
    {
        await _context.Products.AddAsync(product);
        await _context.SaveChangesAsync();
    }
}

// DbContext
public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) 
        : base(options)
    {
    }

    public DbSet<Product> Products { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.ApplyConfigurationsFromAssembly(
            Assembly.GetExecutingAssembly());
    }
}
```

### CQRS Pattern

```csharp
// Command
public record CreateProductCommand(string Name, decimal Price) 
    : IRequest<ProductDto>;

// Command Handler
public class CreateProductCommandHandler 
    : IRequestHandler<CreateProductCommand, ProductDto>
{
    private readonly IProductRepository _repository;
    private readonly IMapper _mapper;

    public CreateProductCommandHandler(
        IProductRepository repository, 
        IMapper mapper)
    {
        _repository = repository;
        _mapper = mapper;
    }

    public async Task<ProductDto> Handle(
        CreateProductCommand request, 
        CancellationToken cancellationToken)
    {
        var product = new Product(request.Name, request.Price);
        await _repository.AddAsync(product);
        return _mapper.Map<ProductDto>(product);
    }
}

// Query
public record GetProductByIdQuery(int Id) : IRequest<ProductDto>;

// Query Handler
public class GetProductByIdQueryHandler 
    : IRequestHandler<GetProductByIdQuery, ProductDto>
{
    private readonly IProductRepository _repository;
    private readonly IMapper _mapper;

    public async Task<ProductDto> Handle(
        GetProductByIdQuery request, 
        CancellationToken cancellationToken)
    {
        var product = await _repository.GetByIdAsync(request.Id);
        return _mapper.Map<ProductDto>(product);
    }
}

// Controller usage
[HttpPost]
public async Task<IActionResult> Create([FromBody] CreateProductCommand command)
{
    var result = await _mediator.Send(command);
    return CreatedAtAction(nameof(GetById), new { id = result.Id }, result);
}
```

### Repository Pattern

```csharp
// Generic repository interface
public interface IRepository<T> where T : class
{
    Task<T> GetByIdAsync(int id);
    Task<IEnumerable<T>> GetAllAsync();
    Task<IEnumerable<T>> FindAsync(Expression<Func<T, bool>> predicate);
    Task AddAsync(T entity);
    Task AddRangeAsync(IEnumerable<T> entities);
    void Update(T entity);
    void Remove(T entity);
    void RemoveRange(IEnumerable<T> entities);
}

// Generic repository implementation
public class Repository<T> : IRepository<T> where T : class
{
    protected readonly DbContext _context;

    public Repository(DbContext context)
    {
        _context = context;
    }

    public async Task<T> GetByIdAsync(int id)
    {
        return await _context.Set<T>().FindAsync(id);
    }

    public async Task<IEnumerable<T>> GetAllAsync()
    {
        return await _context.Set<T>().ToListAsync();
    }

    public async Task<IEnumerable<T>> FindAsync(
        Expression<Func<T, bool>> predicate)
    {
        return await _context.Set<T>().Where(predicate).ToListAsync();
    }

    public async Task AddAsync(T entity)
    {
        await _context.Set<T>().AddAsync(entity);
    }

    public void Update(T entity)
    {
        _context.Set<T>().Update(entity);
    }

    public void Remove(T entity)
    {
        _context.Set<T>().Remove(entity);
    }
}

// Unit of Work
public interface IUnitOfWork : IDisposable
{
    IProductRepository Products { get; }
    IOrderRepository Orders { get; }
    Task<int> SaveChangesAsync();
}

public class UnitOfWork : IUnitOfWork
{
    private readonly AppDbContext _context;

    public UnitOfWork(AppDbContext context)
    {
        _context = context;
        Products = new ProductRepository(_context);
        Orders = new OrderRepository(_context);
    }

    public IProductRepository Products { get; }
    public IOrderRepository Orders { get; }

    public async Task<int> SaveChangesAsync()
    {
        return await _context.SaveChangesAsync();
    }

    public void Dispose()
    {
        _context.Dispose();
    }
}
```

---

## 🔐 Authentication & Authorization

### JWT Authentication

**Configuration:**
```csharp
// appsettings.json
{
  "JwtSettings": {
    "Secret": "your-256-bit-secret-key-here",
    "Issuer": "your-app",
    "Audience": "your-app-users",
    "ExpirationMinutes": 60
  }
}

// Settings class
public class JwtSettings
{
    public string Secret { get; set; }
    public string Issuer { get; set; }
    public string Audience { get; set; }
    public int ExpirationMinutes { get; set; }
}

// Program.cs configuration
builder.Services.Configure<JwtSettings>(
    builder.Configuration.GetSection("JwtSettings"));

builder.Services.AddAuthentication(options =>
{
    options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
})
.AddJwtBearer(options =>
{
    var jwtSettings = builder.Configuration
        .GetSection("JwtSettings")
        .Get<JwtSettings>();

    options.TokenValidationParameters = new TokenValidationParameters
    {
        ValidateIssuer = true,
        ValidateAudience = true,
        ValidateLifetime = true,
        ValidateIssuerSigningKey = true,
        ValidIssuer = jwtSettings.Issuer,
        ValidAudience = jwtSettings.Audience,
        IssuerSigningKey = new SymmetricSecurityKey(
            Encoding.UTF8.GetBytes(jwtSettings.Secret))
    };
});
```

**Token generation:**
```csharp
public class TokenService : ITokenService
{
    private readonly JwtSettings _jwtSettings;

    public TokenService(IOptions<JwtSettings> jwtSettings)
    {
        _jwtSettings = jwtSettings.Value;
    }

    public string GenerateToken(User user)
    {
        var claims = new[]
        {
            new Claim(ClaimTypes.NameIdentifier, user.Id.ToString()),
            new Claim(ClaimTypes.Name, user.Username),
            new Claim(ClaimTypes.Email, user.Email),
            new Claim(ClaimTypes.Role, user.Role)
        };

        var key = new SymmetricSecurityKey(
            Encoding.UTF8.GetBytes(_jwtSettings.Secret));
        
        var credentials = new SigningCredentials(
            key, 
            SecurityAlgorithms.HmacSha256);

        var token = new JwtSecurityToken(
            issuer: _jwtSettings.Issuer,
            audience: _jwtSettings.Audience,
            claims: claims,
            expires: DateTime.UtcNow.AddMinutes(_jwtSettings.ExpirationMinutes),
            signingCredentials: credentials
        );

        return new JwtSecurityTokenHandler().WriteToken(token);
    }

    public string GenerateRefreshToken()
    {
        var randomNumber = new byte[32];
        using var rng = RandomNumberGenerator.Create();
        rng.GetBytes(randomNumber);
        return Convert.ToBase64String(randomNumber);
    }
}
```

**Authentication controller:**
```csharp
[ApiController]
[Route("api/[controller]")]
public class AuthController : ControllerBase
{
    private readonly IAuthService _authService;
    private readonly ITokenService _tokenService;

    public AuthController(
        IAuthService authService, 
        ITokenService tokenService)
    {
        _authService = authService;
        _tokenService = tokenService;
    }

    [HttpPost("register")]
    public async Task<ActionResult<ApiResponse<AuthResponseDto>>> Register(
        [FromBody] RegisterDto dto)
    {
        var user = await _authService.RegisterAsync(dto);
        var token = _tokenService.GenerateToken(user);
        var refreshToken = _tokenService.GenerateRefreshToken();

        await _authService.SaveRefreshTokenAsync(user.Id, refreshToken);

        var response = new AuthResponseDto
        {
            Token = token,
            RefreshToken = refreshToken,
            User = user
        };

        return Ok(ApiResponse<AuthResponseDto>.SuccessResponse(response));
    }

    [HttpPost("login")]
    public async Task<ActionResult<ApiResponse<AuthResponseDto>>> Login(
        [FromBody] LoginDto dto)
    {
        var user = await _authService.AuthenticateAsync(dto.Username, dto.Password);
        
        if (user == null)
            return Unauthorized(ApiResponse<AuthResponseDto>
                .ErrorResponse("Invalid credentials"));

        var token = _tokenService.GenerateToken(user);
        var refreshToken = _tokenService.GenerateRefreshToken();

        await _authService.SaveRefreshTokenAsync(user.Id, refreshToken);

        var response = new AuthResponseDto
        {
            Token = token,
            RefreshToken = refreshToken,
            User = user
        };

        return Ok(ApiResponse<AuthResponseDto>.SuccessResponse(response));
    }

    [HttpPost("refresh")]
    public async Task<ActionResult<ApiResponse<AuthResponseDto>>> Refresh(
        [FromBody] RefreshTokenDto dto)
    {
        var user = await _authService.ValidateRefreshTokenAsync(dto.RefreshToken);
        
        if (user == null)
            return Unauthorized(ApiResponse<AuthResponseDto>
                .ErrorResponse("Invalid refresh token"));

        var token = _tokenService.GenerateToken(user);
        var newRefreshToken = _tokenService.GenerateRefreshToken();

        await _authService.SaveRefreshTokenAsync(user.Id, newRefreshToken);

        var response = new AuthResponseDto
        {
            Token = token,
            RefreshToken = newRefreshToken,
            User = user
        };

        return Ok(ApiResponse<AuthResponseDto>.SuccessResponse(response));
    }
}
```

###Authorization Policies

```csharp
// Program.cs
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("AdminOnly", policy => 
        policy.RequireRole("Admin"));
    
    options.AddPolicy("ModeratorOrAdmin", policy => 
        policy.RequireRole("Admin", "Moderator"));
    
    options.AddPolicy("MinimumAge", policy => 
        policy.Requirements.Add(new MinimumAgeRequirement(18)));
    
    options.AddPolicy("EditProduct", policy =>
        policy.Requirements.Add(new ProductEditRequirement()));
});

// Custom requirement
public class MinimumAgeRequirement : IAuthorizationRequirement
{
    public int MinimumAge { get; }

    public MinimumAgeRequirement(int minimumAge)
    {
        MinimumAge = minimumAge;
    }
}

// Custom handler
public class MinimumAgeHandler : AuthorizationHandler<MinimumAgeRequirement>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        MinimumAgeRequirement requirement)
    {
        var dateOfBirthClaim = context.User
            .FindFirst(c => c.Type == ClaimTypes.DateOfBirth);

        if (dateOfBirthClaim == null)
            return Task.CompletedTask;

        var dateOfBirth = Convert.ToDateTime(dateOfBirthClaim.Value);
        var age = DateTime.Today.Year - dateOfBirth.Year;

        if (age >= requirement.MinimumAge)
            context.Succeed(requirement);

        return Task.CompletedTask;
    }
}

// Usage in controller
[Authorize(Policy = "AdminOnly")]
[HttpDelete("{id}")]
public async Task<IActionResult> Delete(int id)
{
    // Only admins can access
}

[Authorize(Policy = "MinimumAge")]
[HttpGet("adult-content")]
public IActionResult GetAdultContent()
{
    // Only users 18+ can access
}
```

---

## 📊 Entity Framework Core

### DbContext Configuration

```csharp
public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) 
        : base(options)
    {
    }

    public DbSet<Product> Products { get; set; }
    public DbSet<Category> Categories { get; set; }
    public DbSet<Order> Orders { get; set; }
    public DbSet<OrderItem> OrderItems { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

        // Apply all configurations from assembly
        modelBuilder.ApplyConfigurationsFromAssembly(
            Assembly.GetExecutingAssembly());

        // Or apply individual configurations
        modelBuilder.ApplyConfiguration(new ProductConfiguration());
        modelBuilder.ApplyConfiguration(new CategoryConfiguration());

        // Global query filters
        modelBuilder.Entity<Product>()
            .HasQueryFilter(p => !p.IsDeleted);

        // Seed data
        modelBuilder.Entity<Category>().HasData(
            new Category { Id = 1, Name = "Electronics" },
            new Category { Id = 2, Name = "Books" }
        );
    }
}
```

### Entity Configuration

```csharp
public class ProductConfiguration : IEntityTypeConfiguration<Product>
{
    public void Configure(EntityTypeBuilder<Product> builder)
    {
        // Table name
        builder.ToTable("Products");

        // Primary key
        builder.HasKey(p => p.Id);

        // Properties
        builder.Property(p => p.Name)
            .IsRequired()
            .HasMaxLength(200);

        builder.Property(p => p.Price)
            .HasColumnType("decimal(18,2)")
            .IsRequired();

        builder.Property(p => p.Description)
            .HasMaxLength(1000);

        // Indexes
        builder.HasIndex(p => p.Name);
        builder.HasIndex(p => p.CategoryId);

        // Relationships
        builder.HasOne(p => p.Category)
            .WithMany(c => c.Products)
            .HasForeignKey(p => p.CategoryId)
            .OnDelete(DeleteBehavior.Restrict);

        // Owned types
        builder.OwnsOne(p => p.Address, address =>
        {
            address.Property(a => a.Street).HasMaxLength(200);
            address.Property(a => a.City).HasMaxLength(100);
        });

        // Ignore properties
        builder.Ignore(p => p.TemporaryData);

        // Value conversions
        builder.Property(p => p.Status)
            .HasConversion<string>();
    }
}
```

### Migrations

```bash
# Add migration
dotnet ef migrations add InitialCreate

# Update database
dotnet ef database update

# Remove last migration
dotnet ef migrations remove

# Generate SQL script
dotnet ef migrations script

# Update to specific migration
dotnet ef database update SpecificMigration

# Rollback
dotnet ef database update 0
```

### Query Patterns

```csharp
public class ProductRepository : IProductRepository
{
    private readonly AppDbContext _context;

    public ProductRepository(AppDbContext context)
    {
        _context = context;
    }

    // Basic queries
    public async Task<Product> GetByIdAsync(int id)
    {
        return await _context.Products
            .Include(p => p.Category)
            .FirstOrDefaultAsync(p => p.Id == id);
    }

    // Pagination
    public async Task<IEnumerable<Product>> GetPagedAsync(
        int pageNumber, 
        int pageSize)
    {
        return await _context.Products
            .OrderBy(p => p.Name)
            .Skip((pageNumber - 1) * pageSize)
            .Take(pageSize)
            .ToListAsync();
    }

    // Filtering with specifications
    public async Task<IEnumerable<Product>> FindAsync(
        Expression<Func<Product, bool>> predicate)
    {
        return await _context.Products
            .Where(predicate)
            .ToListAsync();
    }

    // Complex queries
    public async Task<IEnumerable<ProductDto>> GetProductsWithCategoryAsync()
    {
        return await _context.Products
            .Include(p => p.Category)
            .Select(p => new ProductDto
            {
                Id = p.Id,
                Name = p.Name,
                Price = p.Price,
                CategoryName = p.Category.Name
            })
            .ToListAsync();
    }

    // Aggregations
    public async Task<decimal> GetAveragePriceAsync()
    {
        return await _context.Products.AverageAsync(p => p.Price);
    }

    // Group by
    public async Task<Dictionary<string, int>> GetProductCountByCategoryAsync()
    {
        return await _context.Products
            .GroupBy(p => p.Category.Name)
            .Select(g => new { Category = g.Key, Count = g.Count() })
            .ToDictionaryAsync(x => x.Category, x => x.Count);
    }

    // Raw SQL
    public async Task<IEnumerable<Product>> GetProductsRawSqlAsync(decimal minPrice)
    {
        return await _context.Products
            .FromSqlRaw("SELECT * FROM Products WHERE Price >= {0}", minPrice)
            .ToListAsync();
    }

    // Stored procedure
    public async Task<IEnumerable<Product>> GetProductsByStoredProcAsync(int categoryId)
    {
        return await _context.Products
            .FromSqlRaw("EXEC GetProductsByCategory @p0", categoryId)
            .ToListAsync();
    }

    // Batch operations
    public async Task BulkUpdatePricesAsync(decimal percentage)
    {
        await _context.Products
            .Where(p => p.CategoryId == 1)
            .ExecuteUpdateAsync(setters => setters
                .SetProperty(p => p.Price, p => p.Price * percentage));
    }

    // Soft delete
    public async Task SoftDeleteAsync(int id)
    {
        await _context.Products
            .Where(p => p.Id == id)
            .ExecuteUpdateAsync(setters => setters
                .SetProperty(p => p.IsDeleted, true)
                .SetProperty(p => p.DeletedAt, DateTime.UtcNow));
    }
}
```

### Performance Optimization

```csharp
// AsNoTracking for read-only queries
public async Task<IEnumerable<Product>> GetAllReadOnlyAsync()
{
    return await _context.Products
        .AsNoTracking()
        .ToListAsync();
}

// Compiled queries
private static readonly Func<AppDbContext, int, Task<Product>> 
    _compiledQuery = EF.CompileAsyncQuery(
        (AppDbContext context, int id) =>
            context.Products.FirstOrDefault(p => p.Id == id));

public async Task<Product> GetByIdCompiledAsync(int id)
{
    return await _compiledQuery(_context, id);
}

// Split queries for collections
public async Task<Order> GetOrderWithItemsAsync(int id)
{
    return await _context.Orders
        .Include(o => o.Items)
        .AsSplitQuery()  // Generates separate SQL queries
        .FirstOrDefaultAsync(o => o.Id == id);
}

// Projection for better performance
public async Task<IEnumerable<ProductSummaryDto>> GetProductSummariesAsync()
{
    return await _context.Products
        .Select(p => new ProductSummaryDto
        {
            Id = p.Id,
            Name = p.Name,
            Price = p.Price
        })
        .AsNoTracking()
        .ToListAsync();
}
```

---

## 🔄 Middleware Pipeline

### Custom Middleware

```csharp
// Request logging middleware
public class RequestLoggingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<RequestLoggingMiddleware> _logger;

    public RequestLoggingMiddleware(
        RequestDelegate next,
        ILogger<RequestLoggingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        var requestId = Guid.NewGuid().ToString();
        context.Items["RequestId"] = requestId;

        _logger.LogInformation(
            "Request {RequestId}: {Method} {Path}",
            requestId,
            context.Request.Method,
            context.Request.Path);

        var sw = Stopwatch.StartNew();

        await _next(context);

        sw.Stop();

        _logger.LogInformation(
            "Request {RequestId} completed in {ElapsedMilliseconds}ms with status {StatusCode}",
            requestId,
            sw.ElapsedMilliseconds,
            context.Response.StatusCode);
    }
}

// Extension method
public static class MiddlewareExtensions
{
    public static IApplicationBuilder UseRequestLogging(
        this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<RequestLoggingMiddleware>();
    }
}

// Program.cs
app.UseRequestLogging();
```

### API Key Authentication Middleware

```csharp
public class ApiKeyMiddleware
{
    private readonly RequestDelegate _next;
    private readonly IConfiguration _configuration;
    private const string API_KEY_HEADER = "X-API-Key";

    public ApiKeyMiddleware(
        RequestDelegate next,
        IConfiguration configuration)
    {
        _next = next;
        _configuration = configuration;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        if (!context.Request.Headers.TryGetValue(API_KEY_HEADER, out var extractedApiKey))
        {
            context.Response.StatusCode = 401;
            await context.Response.WriteAsync("API Key missing");
            return;
        }

        var apiKey = _configuration.GetValue<string>("ApiKey");

        if (!apiKey.Equals(extractedApiKey))
        {
            context.Response.StatusCode = 401;
            await context.Response.WriteAsync("Invalid API Key");
            return;
        }

        await _next(context);
    }
}
```

### Rate Limiting Middleware

```csharp
public class RateLimitingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly IMemoryCache _cache;
    private readonly int _requestLimit = 100;
    private readonly TimeSpan _timeWindow = TimeSpan.FromMinutes(1);

    public RateLimitingMiddleware(
        RequestDelegate next,
        IMemoryCache cache)
    {
        _next = next;
        _cache = cache;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        var clientId = context.Connection.RemoteIpAddress?.ToString() 
            ?? "unknown";
        var cacheKey = $"rate_limit_{clientId}";

        var requestCount = _cache.GetOrCreate(cacheKey, entry =>
        {
            entry.AbsoluteExpirationRelativeToNow = _timeWindow;
            return 0;
        });

        if (requestCount >= _requestLimit)
        {
            context.Response.StatusCode = 429;
            await context.Response.WriteAsync("Rate limit exceeded");
            return;
        }

        _cache.Set(cacheKey, requestCount + 1, _timeWindow);

        await _next(context);
    }
}
```

### Request/Response Logging

```csharp
public class RequestResponseLoggingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<RequestResponseLoggingMiddleware> _logger;

    public RequestResponseLoggingMiddleware(
        RequestDelegate next,
        ILogger<RequestResponseLoggingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        // Log request
        context.Request.EnableBuffering();
        var requestBody = await ReadRequestBodyAsync(context.Request);
        
        _logger.LogInformation(
            "Request: {Method} {Path} {Body}",
            context.Request.Method,
            context.Request.Path,
            requestBody);

        // Capture response
        var originalBodyStream = context.Response.Body;

        using var responseBody = new MemoryStream();
        context.Response.Body = responseBody;

        await _next(context);

        // Log response
        context.Response.Body.Seek(0, SeekOrigin.Begin);
        var responseBodyText = await new StreamReader(context.Response.Body).ReadToEndAsync();
        context.Response.Body.Seek(0, SeekOrigin.Begin);

        _logger.LogInformation(
            "Response: {StatusCode} {Body}",
            context.Response.StatusCode,
            responseBodyText);

        await responseBody.CopyToAsync(originalBodyStream);
    }

    private async Task<string> ReadRequestBodyAsync(HttpRequest request)
    {
        request.Body.Seek(0, SeekOrigin.Begin);
        var body = await new StreamReader(request.Body).ReadToEndAsync();
        request.Body.Seek(0, SeekOrigin.Begin);
        return body;
    }
}
```

---

## 🚀 Performance & Caching

### Response Caching

```csharp
// Program.cs
builder.Services.AddResponseCaching();

app.UseResponseCaching();

// Controller
[ResponseCache(Duration = 60)] // Cache for 60 seconds
[HttpGet]
public async Task<IActionResult> GetAll()
{
    var products = await _productService.GetAllAsync();
    return Ok(products);
}

// Vary by query parameter
[ResponseCache(Duration = 60, VaryByQueryKeys = new[] { "category" })]
[HttpGet]
public async Task<IActionResult> GetByCategory([FromQuery] string category)
{
    var products = await _productService.GetByCategoryAsync(category);
    return Ok(products);
}

// No cache
[ResponseCache(NoStore = true, Location = ResponseCacheLocation.None)]
[HttpGet("realtime")]
public async Task<IActionResult> GetRealtime()
{
    return Ok(DateTime.UtcNow);
}
```

### Memory Cache

```csharp
public class CachedProductService : IProductService
{
    private readonly IProductService _inner;
    private readonly IMemoryCache _cache;
    private readonly ILogger<CachedProductService> _logger;
    private const string CACHE_KEY_PREFIX = "product_";
    private readonly TimeSpan _cacheExpiration = TimeSpan.FromMinutes(10);

    public CachedProductService(
        IProductService inner,
        IMemoryCache cache,
        ILogger<CachedProductService> logger)
    {
        _inner = inner;
        _cache = cache;
        _logger = logger;
    }

    public async Task<ProductDto> GetByIdAsync(int id)
    {
        var cacheKey = $"{CACHE_KEY_PREFIX}{id}";

        if (_cache.TryGetValue(cacheKey, out ProductDto cachedProduct))
        {
            _logger.LogInformation("Cache hit for product {ProductId}", id);
            return cachedProduct;
        }

        _logger.LogInformation("Cache miss for product {ProductId}", id);
        var product = await _inner.GetByIdAsync(id);

        var cacheOptions = new MemoryCacheEntryOptions()
            .SetAbsoluteExpiration(_cacheExpiration)
            .SetSlidingExpiration(TimeSpan.FromMinutes(2))
            .RegisterPostEvictionCallback((key, value, reason, state) =>
            {
                _logger.LogInformation(
                    "Cache entry {Key} evicted. Reason: {Reason}",
                    key,
                    reason);
            });

        _cache.Set(cacheKey, product, cacheOptions);

        return product;
    }

    public async Task UpdateAsync(int id, UpdateProductDto dto)
    {
        await _inner.UpdateAsync(id, dto);
        
        // Invalidate cache
        var cacheKey = $"{CACHE_KEY_PREFIX}{id}";
        _cache.Remove(cacheKey);
    }
}
```

### Distributed Cache (Redis)

```csharp
// Program.cs
builder.Services.AddStackExchangeRedisCache(options =>
{
    options.Configuration = builder.Configuration
        .GetConnectionString("Redis");
    options.InstanceName = "MyApp_";
});

// Service
public class RedisProductService : IProductService
{
    private readonly IProductService _inner;
    private readonly IDistributedCache _cache;
    private readonly ILogger<RedisProductService> _logger;
    private readonly TimeSpan _cacheExpiration = TimeSpan.FromHours(1);

    public RedisProductService(
        IProductService inner,
        IDistributedCache cache,
        ILogger<RedisProductService> logger)
    {
        _inner = inner;
        _cache = cache;
        _logger = logger;
    }

    public async Task<ProductDto> GetByIdAsync(int id)
    {
        var cacheKey = $"product_{id}";
        var cachedData = await _cache.GetStringAsync(cacheKey);

        if (!string.IsNullOrEmpty(cachedData))
        {
            _logger.LogInformation("Redis cache hit for product {ProductId}", id);
            return JsonSerializer.Deserialize<ProductDto>(cachedData);
        }

        _logger.LogInformation("Redis cache miss for product {ProductId}", id);
        var product = await _inner.GetByIdAsync(id);

        var options = new DistributedCacheEntryOptions
        {
            AbsoluteExpirationRelativeToNow = _cacheExpiration
        };

        await _cache.SetStringAsync(
            cacheKey,
            JsonSerializer.Serialize(product),
            options);

        return product;
    }
}
```

### Output Caching (ASP.NET Core 7+)

```csharp
// Program.cs
builder.Services.AddOutputCache(options =>
{
    options.AddBasePolicy(builder => 
        builder.Expire(TimeSpan.FromSeconds(60)));
    
    options.AddPolicy("Expire20", builder => 
        builder.Expire(TimeSpan.FromSeconds(20)));
    
    options.AddPolicy("NoCache", builder => 
        builder.NoCache());
});

app.UseOutputCache();

// Controller
[OutputCache(Duration = 60)]
[HttpGet]
public async Task<IActionResult> GetAll()
{
    return Ok(await _productService.GetAllAsync());
}

[OutputCache(PolicyName = "Expire20")]
[HttpGet("featured")]
public async Task<IActionResult> GetFeatured()
{
    return Ok(await _productService.GetFeaturedAsync());
}
```

---

## 📡 Real-time Communication (SignalR)

### Hub Setup

```csharp
// Hub
public class ChatHub : Hub
{
    private readonly ILogger<ChatHub> _logger;

    public ChatHub(ILogger<ChatHub> logger)
    {
        _logger = logger;
    }

    public async Task SendMessage(string user, string message)
    {
        await Clients.All.SendAsync("ReceiveMessage", user, message);
    }

    public async Task SendMessageToUser(string userId, string message)
    {
        await Clients.User(userId).SendAsync("ReceiveMessage", message);
    }

    public async Task JoinGroup(string groupName)
    {
        await Groups.AddToGroupAsync(Context.ConnectionId, groupName);
        await Clients.Group(groupName).SendAsync("UserJoined", Context.ConnectionId);
    }

    public async Task LeaveGroup(string groupName)
    {
        await Groups.RemoveFromGroupAsync(Context.ConnectionId, groupName);
        await Clients.Group(groupName).SendAsync("UserLeft", Context.ConnectionId);
    }

    public override async Task OnConnectedAsync()
    {
        _logger.LogInformation("Client connected: {ConnectionId}", Context.ConnectionId);
        await base.OnConnectedAsync();
    }

    public override async Task OnDisconnectedAsync(Exception exception)
    {
        _logger.LogInformation("Client disconnected: {ConnectionId}", Context.ConnectionId);
        await base.OnDisconnectedAsync(exception);
    }
}

// Program.cs
builder.Services.AddSignalR();

app.MapHub<ChatHub>("/chathub");
```

### Strongly Typed Hubs

```csharp
// Client interface
public interface IChatClient
{
    Task ReceiveMessage(string user, string message);
    Task UserJoined(string connectionId);
    Task UserLeft(string connectionId);
}

// Strongly typed hub
public class ChatHub : Hub<IChatClient>
{
    public async Task SendMessage(string user, string message)
    {
        await Clients.All.ReceiveMessage(user, message);
    }

    public async Task JoinGroup(string groupName)
    {
        await Groups.AddToGroupAsync(Context.ConnectionId, groupName);
        await Clients.Group(groupName).UserJoined(Context.ConnectionId);
    }
}
```

### Injecting Services into Hubs

```csharp
public class NotificationHub : Hub
{
    private readonly INotificationService _notificationService;
    private readonly IUserService _userService;

    public NotificationHub(
        INotificationService notificationService,
        IUserService userService)
    {
        _notificationService = notificationService;
        _userService = userService;
    }

    public async Task SendNotification(int userId, string message)
    {
        var user = await _userService.GetByIdAsync(userId);
        await _notificationService.SendAsync(user, message);
        await Clients.User(userId.ToString()).SendAsync("ReceiveNotification", message);
    }
}
```

---

## 🧪 Testing

### Unit Testing

```csharp
public class ProductServiceTests
{
    private readonly Mock<IProductRepository> _repositoryMock;
    private readonly Mock<IMapper> _mapperMock;
    private readonly Mock<ILogger<ProductService>> _loggerMock;
    private readonly ProductService _service;

    public ProductServiceTests()
    {
        _repositoryMock = new Mock<IProductRepository>();
        _mapperMock = new Mock<IMapper>();
        _loggerMock = new Mock<ILogger<ProductService>>();
        _service = new ProductService(
            _repositoryMock.Object,
            _mapperMock.Object,
            _loggerMock.Object);
    }

    [Fact]
    public async Task GetByIdAsync_ExistingProduct_ReturnsProduct()
    {
        // Arrange
        var productId = 1;
        var product = new Product { Id = productId, Name = "Test Product" };
        var productDto = new ProductDto { Id = productId, Name = "Test Product" };

        _repositoryMock
            .Setup(r => r.GetByIdAsync(productId))
            .ReturnsAsync(product);

        _mapperMock
            .Setup(m => m.Map<ProductDto>(product))
            .Returns(productDto);

        // Act
        var result = await _service.GetByIdAsync(productId);

        // Assert
        Assert.NotNull(result);
        Assert.Equal(productId, result.Id);
        Assert.Equal("Test Product", result.Name);

        _repositoryMock.Verify(r => r.GetByIdAsync(productId), Times.Once);
    }

    [Fact]
    public async Task GetByIdAsync_NonExistingProduct_ReturnsNull()
    {
        // Arrange
        var productId = 999;
        _repositoryMock
            .Setup(r => r.GetByIdAsync(productId))
            .ReturnsAsync((Product)null);

        // Act
        var result = await _service.GetByIdAsync(productId);

        // Assert
        Assert.Null(result);
    }

    [Theory]
    [InlineData(1)]
    [InlineData(5)]
    [InlineData(10)]
    public async Task GetByIdAsync_VariousIds_CallsRepository(int productId)
    {
        // Arrange
        _repositoryMock
            .Setup(r => r.GetByIdAsync(It.IsAny<int>()))
            .ReturnsAsync((Product)null);

        // Act
        await _service.GetByIdAsync(productId);

        // Assert
        _repositoryMock.Verify(r => r.GetByIdAsync(productId), Times.Once);
    }
}
```

### Integration Testing

```csharp
public class ProductsControllerIntegrationTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly WebApplicationFactory<Program> _factory;
    private readonly HttpClient _client;

    public ProductsControllerIntegrationTests(WebApplicationFactory<Program> factory)
    {
        _factory = factory.WithWebHostBuilder(builder =>
        {
            builder.ConfigureServices(services =>
            {
                // Remove existing DbContext
                var descriptor = services.SingleOrDefault(
                    d => d.ServiceType == typeof(DbContextOptions<AppDbContext>));
                
                if (descriptor != null)
                    services.Remove(descriptor);

                // Add in-memory database
                services.AddDbContext<AppDbContext>(options =>
                {
                    options.UseInMemoryDatabase("TestDb");
                });

                // Build and seed database
                var sp = services.BuildServiceProvider();
                using var scope = sp.CreateScope();
                var db = scope.ServiceProvider.GetRequiredService<AppDbContext>();
                db.Database.EnsureCreated();
                SeedDatabase(db);
            });
        });

        _client = _factory.CreateClient();
    }

    private void SeedDatabase(AppDbContext context)
    {
        context.Products.AddRange(
            new Product { Id = 1, Name = "Product 1", Price = 10.99m },
            new Product { Id = 2, Name = "Product 2", Price = 20.99m }
        );
        context.SaveChanges();
    }

    [Fact]
    public async Task GetAll_ReturnsSuccessAndProducts()
    {
        // Act
        var response = await _client.GetAsync("/api/products");

        // Assert
        response.EnsureSuccessStatusCode();
        var content = await response.Content.ReadAsStringAsync();
        var products = JsonSerializer.Deserialize<List<ProductDto>>(content);
        
        Assert.NotNull(products);
        Assert.Equal(2, products.Count);
    }

    [Fact]
    public async Task GetById_ExistingId_ReturnsProduct()
    {
        // Act
        var response = await _client.GetAsync("/api/products/1");

        // Assert
        response.EnsureSuccessStatusCode();
        var content = await response.Content.ReadAsStringAsync();
        var product = JsonSerializer.Deserialize<ProductDto>(content);
        
        Assert.NotNull(product);
        Assert.Equal(1, product.Id);
    }

    [Fact]
    public async Task Create_ValidProduct_ReturnsCreated()
    {
        // Arrange
        var newProduct = new CreateProductDto
        {
            Name = "New Product",
            Price = 15.99m
        };

        var content = new StringContent(
            JsonSerializer.Serialize(newProduct),
            Encoding.UTF8,
            "application/json");

        // Act
        var response = await _client.PostAsync("/api/products", content);

        // Assert
        Assert.Equal(HttpStatusCode.Created, response.StatusCode);
    }
}
```

### E2E Testing with SpecFlow

```csharp
[Binding]
public class ProductSteps
{
    private readonly ScenarioContext _context;
    private readonly HttpClient _client;
    private HttpResponseMessage _response;

    public ProductSteps(ScenarioContext context)
    {
        _context = context;
        _client = new HttpClient { BaseAddress = new Uri("http://localhost:5000") };
    }

    [Given(@"I have a product with name ""(.*)"" and price (.*)")]
    public void GivenIHaveAProduct(string name, decimal price)
    {
        _context["product"] = new CreateProductDto
        {
            Name = name,
            Price = price
        };
    }

    [When(@"I submit the product")]
    public async Task WhenISubmitTheProduct()
    {
        var product = _context["product"] as CreateProductDto;
        var content = new StringContent(
            JsonSerializer.Serialize(product),
            Encoding.UTF8,
            "application/json");

        _response = await _client.PostAsync("/api/products", content);
    }

    [Then(@"the product should be created")]
    public void ThenTheProductShouldBeCreated()
    {
        Assert.Equal(HttpStatusCode.Created, _response.StatusCode);
    }

    [Then(@"I should receive the product details")]
    public async Task ThenIShouldReceiveTheProductDetails()
    {
        var content = await _response.Content.ReadAsStringAsync();
        var product = JsonSerializer.Deserialize<ProductDto>(content);
        
        Assert.NotNull(product);
        Assert.NotEqual(0, product.Id);
    }
}
```

---

## 📦 Deployment

### Docker

```dockerfile
# Dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY ["MyApp.API/MyApp.API.csproj", "MyApp.API/"]
COPY ["MyApp.Application/MyApp.Application.csproj", "MyApp.Application/"]
COPY ["MyApp.Domain/MyApp.Domain.csproj", "MyApp.Domain/

"]
COPY ["MyApp.Infrastructure/MyApp.Infrastructure.csproj", "MyApp.Infrastructure/"]

RUN dotnet restore "MyApp.API/MyApp.API.csproj"
COPY . .
WORKDIR "/src/MyApp.API"
RUN dotnet build "MyApp.API.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "MyApp.API.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "MyApp.API.dll"]
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  api:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "5000:80"
    environment:
      - ASPNETCORE_ENVIRONMENT=Production
      - ConnectionStrings__DefaultConnection=Server=db;Database=MyAppDb;User=sa;Password=YourStrong@Passw0rd
    depends_on:
      - db
      - redis

  db:
    image: mcr.microsoft.com/mssql/server:2022-latest
    environment:
      - ACCEPT_EULA=Y
      - SA_PASSWORD=YourStrong@Passw0rd
    ports:
      - "1433:1433"
    volumes:
      - sqldata:/var/opt/mssql

  redis:
    image: redis:alpine
    ports:
      - "6379:6379"

volumes:
  sqldata:
```

### Azure App Service

```bash
# Login to Azure
az login

# Create resource group
az group create --name myResourceGroup --location eastus

# Create App Service plan
az appservice plan create \
  --name myAppServicePlan \
  --resource-group myResourceGroup \
  --sku B1 \
  --is-linux

# Create web app
az webapp create \
  --resource-group myResourceGroup \
  --plan myAppServicePlan \
  --name myUniqueAppName \
  --runtime "DOTNET|8.0"

# Deploy
az webapp deployment source config-zip \
  --resource-group myResourceGroup \
  --name myUniqueAppName \
  --src ./publish.zip
```

### CI/CD with GitHub Actions

```yaml
# .github/workflows/deploy.yml
name: Deploy to Azure

on:
  push:
    branches: [ main ]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3

    - name: Setup .NET
      uses: actions/setup-dotnet@v3
      with:
        dotnet-version: '8.0.x'

    - name: Restore dependencies
      run: dotnet restore

    - name: Build
      run: dotnet build --configuration Release --no-restore

    - name: Test
      run: dotnet test --no-restore --verbosity normal

    - name: Publish
      run: dotnet publish -c Release -o ./publish

    - name: Deploy to Azure Web App
      uses: azure/webapps-deploy@v2
      with:
        app-name: ${{ secrets.AZURE_WEBAPP_NAME }}
        publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
        package: ./publish
```

---

## 📚 Essential NuGet Packages

### Core Packages
```xml
<!-- Web API -->
<PackageReference Include="Microsoft.AspNetCore.OpenApi" Version="8.0.0" />
<PackageReference Include="Swashbuckle.AspNetCore" Version="6.5.0" />

<!-- Entity Framework Core -->
<PackageReference Include="Microsoft.EntityFrameworkCore" Version="8.0.0" />
<PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="8.0.0" />
<PackageReference Include="Microsoft.EntityFrameworkCore.Tools" Version="8.0.0" />
<PackageReference Include="Microsoft.EntityFrameworkCore.Design" Version="8.0.0" />

<!-- Authentication -->
<PackageReference Include="Microsoft.AspNetCore.Authentication.JwtBearer" Version="8.0.0" />
<PackageReference Include="Microsoft.AspNetCore.Identity.EntityFrameworkCore" Version="8.0.0" />

<!-- Validation -->
<PackageReference Include="FluentValidation.AspNetCore" Version="11.3.0" />

<!-- Mapping -->
<PackageReference Include="AutoMapper.Extensions.Microsoft.DependencyInjection" Version="12.0.1" />

<!-- Caching -->
<PackageReference Include="Microsoft.Extensions.Caching.StackExchangeRedis" Version="8.0.0" />

<!-- SignalR -->
<PackageReference Include="Microsoft.AspNetCore.SignalR" Version="1.1.0" />

<!-- MediatR (CQRS) -->
<PackageReference Include="MediatR" Version="12.2.0" />

<!-- Logging -->
<PackageReference Include="Serilog.AspNetCore" Version="8.0.0" />
<PackageReference Include="Serilog.Sinks.Console" Version="5.0.1" />
<PackageReference Include="Serilog.Sinks.File" Version="5.0.0" />

<!-- Testing -->
<PackageReference Include="xUnit" Version="2.6.6" />
<PackageReference Include="Moq" Version="4.20.70" />
<PackageReference Include="FluentAssertions" Version="6.12.0" />
<PackageReference Include="Microsoft.AspNetCore.Mvc.Testing" Version="8.0.0" />
```

---

## 🎯 Best Practices Checklist

### API Design
- [ ] Use RESTful conventions
- [ ] Implement proper HTTP status codes
- [ ] Version your APIs
- [ ] Use consistent naming conventions
- [ ] Implement pagination for collections
- [ ] Support filtering and sorting
- [ ] Use proper content negotiation
- [ ] Implement HATEOAS (optional)

### Security
- [ ] Use HTTPS everywhere
- [ ] Implement authentication & authorization
- [ ] Validate all input
- [ ] Sanitize output
- [ ] Use parameterized queries (prevent SQL injection)
- [ ] Implement rate limiting
- [ ] Use CORS properly
- [ ] Keep dependencies updated
- [ ] Don't expose sensitive data in errors
- [ ] Implement proper password hashing

### Performance
- [ ] Use async/await consistently
- [ ] Implement caching strategically
- [ ] Use database indexing
- [ ] Optimize database queries
- [ ] Use pagination
- [ ] Implement compression
- [ ] Use CDN for static files
- [ ] Monitor performance metrics
- [ ] Use connection pooling
- [ ] Implement lazy loading where appropriate

### Code Quality
- [ ] Follow SOLID principles
- [ ] Use dependency injection
- [ ] Write unit tests
- [ ] Write integration tests
- [ ] Use meaningful names
- [ ] Keep methods small
- [ ] Avoid code duplication (DRY)
- [ ] Comment complex logic
- [ ] Use consistent formatting
- [ ] Perform code reviews

### Logging & Monitoring
- [ ] Log appropriate information
- [ ] Use structured logging
- [ ] Include correlation IDs
- [ ] Don't log sensitive data
- [ ] Monitor application health
- [ ] Set up alerts
- [ ] Track performance metrics
- [ ] Monitor error rates

---

## 📊 Progress Tracking

### Weekly Check-in
**Date:** _______

**This week I:**
- [ ] Studied ___ hours
- [ ] Completed sections: ___
- [ ] Built features: ___
- [ ] Fixed bugs: ___
- [ ] Deployed to: ___

**Technologies learned:**
- 

**Challenges faced:**
- 

**Solutions implemented:**
- 

**Next week's goals:**
- 

### Skills Assessment (1-10 scale)

| Skill | Month 1 | Month 3 | Month 6 | Month 12 |
|-------|---------|---------|---------|----------|
| ASP.NET Core Basics | | | | |
| Web API Development | | | | |
| Entity Framework Core | | | | |
| Authentication/Authorization | | | | |
| Dependency Injection | | | | |
| Middleware | | | | |
| Testing | | | | |
| Performance Optimization | | | | |
| Deployment | | | | |
| Architecture & Design | | | | |

### Project Milestones

**Project Name:** _______

- [ ] Project setup
- [ ] Database design
- [ ] API endpoints implemented
- [ ] Authentication added
- [ ] Testing completed
- [ ] Documentation written
- [ ] Deployed to staging
- [ ] Deployed to production

---

## 🗓️ Study Schedule

### Daily Routine (2 hours)

**Option A: Learning-focused**
- **0:00-0:30** Read documentation/articles
- **0:30-1:00** Watch tutorial
- **1:00-1:30** Code along
- **1:30-2:00** Practice exercises

**Option B: Building-focused**
- **0:00-0:15** Review previous work
- **0:15-1:30** Build features
- **1:30-2:00** Testing & debugging

### Weekly Breakdown

**Monday:** Fundamentals
- Study ASP.NET Core concepts
- Read documentation
- Write example code

**Tuesday:** Database & EF Core
- Entity Framework practice
- Database design
- Migrations

**Wednesday:** API Development
- Build endpoints
- Implement routing
- Add validation

**Thursday:** Security
- Authentication implementation
- Authorization policies
- Security best practices

**Friday:** Advanced topics
- Caching strategies
- Performance optimization
- Architecture patterns

**Saturday:** Project work
- Build real application
- Integrate learned concepts
- Implement features

**Sunday:** Review & testing
- Write tests
- Code review
- Refactor
- Plan next week

---

## 🔗 Additional Resources

### Documentation
- **Official Docs:** https://docs.microsoft.com/aspnet/core
- **Entity Framework Docs:** https://docs.microsoft.com/ef/core
- **C# Docs:** https://docs.microsoft.com/dotnet/csharp

### Learning Platforms
- **Microsoft Learn** - Free courses
- **Pluralsight** - Comprehensive ASP.NET path
- **Udemy** - Various ASP.NET courses
- **YouTube** - IAmTimCorey, Nick Chapsas

### Community
- **r/dotnet** (Reddit)
- **r/csharp** (Reddit)
- **Stack Overflow** ([asp.net-core] tag)
- **.NET Foundation** Discord
- **ASP.NET Community Standup** (live streams)

### Blogs & Channels
- **Andrew Lock** (andrewlock.net)
- **Scott Hanselman**
- **Nick Chapsas** (YouTube)
- **Tim Corey** (YouTube)
- **.NET Blog** (Microsoft)

### Books
- **"ASP.NET Core in Action"** - Andrew Lock
- **"Pro ASP.NET Core"** - Adam Freeman
- **"Dependency Injection in .NET"** - Mark Seemann
- **"Clean Architecture"** - Robert C. Martin

---

## 💭 Quick Tips for Success

**Development:**
- Start with minimal APIs, progress to controllers
- Learn C# fundamentals thoroughly first
- Master async/await patterns
- Understand the middleware pipeline
- Practice dependency injection early

**Best Practices:**
- Always use async/await for I/O operations
- Follow naming conventions
- Keep controllers thin, logic in services
- Use DTOs for API contracts
- Validate all input

**Learning:**
- Build real projects, not just tutorials
- Read official documentation
- Study open-source ASP.NET projects
- Join the .NET community
- Ask questions on Stack Overflow

**Career:**
- Build a portfolio on GitHub
- Contribute to open source
- Write technical blog posts
- Get Azure/AWS certifications
- Network with other developers

---

## 🎯 Next Steps

### Immediate Actions (This Week)
1. [ ] Set up development environment (VS/Rider)
2. [ ] Create first ASP.NET Core Web API
3. [ ] Read getting started documentation
4. [ ] Implement simple CRUD operations
5. [ ] Add Entity Framework Core
6. [ ] Deploy to local Docker

### This Month
1. [ ] Master Web API fundamentals
2. [ ] Learn Entity Framework Core basics
3. [ ] Implement authentication
4. [ ] Build complete CRUD API
5. [ ] Write unit tests
6. [ ] Deploy to cloud (Azure/AWS)

### Next 3 Months
1. [ ] Master clean architecture
2. [ ] Implement CQRS pattern
3. [ ] Learn SignalR
4. [ ] Build microservices
5. [ ] Master performance optimization
6. [ ] Complete 2-3 substantial projects

### 6-12 Months
1. [ ] Architect enterprise applications
2. [ ] Master advanced patterns
3. [ ] Contribute to open source
4. [ ] Build production-ready systems
5. [ ] Mentor junior developers
6. [ ] Consider Azure/AWS certifications

---

**Remember: ASP.NET Core is a powerful, modern framework for building web applications and APIs. Focus on understanding fundamentals, practice consistently, and build real projects. The .NET ecosystem is vast and welcoming—don't hesitate to ask for help! Happy coding! 🚀**

---

*Customize this guide based on your specific learning path and career goals. Keep it updated as you progress and discover new resources!*