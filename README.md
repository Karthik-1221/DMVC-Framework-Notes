# DMVC Framework Notes - Beginner to Advanced

A structured learning resource for the **DMVCFramework** that moves from beginner-friendly fundamentals to advanced topics used in production APIs and web applications. DMVCFramework is an open-source Delphi framework for building RESTful services, JSON-RPC APIs, and web applications with MVC architecture, middleware support, authentication features, ORM capabilities, and multiple deployment options.[1][2]

## Table of Contents

- [Introduction to DMVC](#introduction-to-dmvc)
- [Setting Up the Environment](#setting-up-the-environment)
- [Core Concepts](#core-concepts)
- [Routing and Request Handling](#routing-and-request-handling)
- [Data Binding and Validation](#data-binding-and-validation)
- [Working with Databases](#working-with-databases)
- [Authentication and Authorization](#authentication-and-authorization)
- [Error Handling and Logging](#error-handling-and-logging)
- [Middleware and Services](#middleware-and-services)
- [Advanced Topics](#advanced-topics)
- [Testing in DMVC](#testing-in-dmvc)
- [Deployment and Best Practices](#deployment-and-best-practices)
- [Next Steps](#next-steps)

## Introduction to DMVC

DMVCFramework is a Delphi framework designed for RESTful services, JSON-RPC APIs, and web applications, with a complete MVC structure and strong support for backend development patterns.[2] It is widely used because it provides controller-based routing, middleware, authentication, ORM support through MVCActiveRecord, content negotiation, and production-ready deployment models.[2][1]

### What it is

- A framework for building web APIs and web applications in Object Pascal/Delphi.[2]
- Based on the MVC pattern, where responsibilities are split across models, views, and controllers.[2]
- Capable of serving REST endpoints, JSON-RPC methods, static files, and real-time features such as SSE and WebSocket support.[1][2]

### Why it is used

- Keeps application structure organized and easier to maintain.[1][2]
- Supports common backend needs like authentication, rate limiting, validation, and database access.[1][2]
- Works well for both simple APIs and larger production-grade services.[2]

```pascal
[MVCPath('/api/hello')]
TMyController = class(TMVCController)
public
  [MVCPath('/world')]
  [MVCHTTPMethod([httpGET])]
  function HelloWorld: string;
end;

function TMyController.HelloWorld: string;
begin
  Result := 'Hello, DMVC!';
end;
```

## Setting Up the Environment

DMVCFramework supports multiple Delphi versions, including Delphi 10 Seattle through Delphi 13 Florence, and can run on Windows and Linux depending on the deployment target.[1][2] A basic setup usually involves installing Delphi, obtaining DMVCFramework from GitHub, and compiling the relevant package set for the Delphi version in use.[1][2]

### Basic steps

1. Install a supported Delphi version.[1][2]
2. Clone the framework repository.
3. Open the package folder that matches the Delphi version.
4. Build and install the design-time or runtime packages as needed.
5. Create a new project and register controllers in `TMVCEngine`.

```bash
git clone https://github.com/danieleteti/delphimvcframework.git
```

```pascal
FMVC := TMVCEngine.Create(Self);
FMVC.AddController(TMyController);
```

## Core Concepts

The MVC structure separates the application into models for data, views for presentation, and controllers for request handling, which improves clarity and maintainability.[2] DMVCFramework also supports server-side views and JSON serialization, so the same project can expose APIs and render responses in different formats.[2]

### Models

Models represent application data and business rules. In DMVCFramework, models are often plain Delphi classes or MVCActiveRecord entities for database-backed records.[2]

```pascal
type
  TPerson = class
  private
    FName: string;
    FAge: Integer;
  public
    property Name: string read FName write FName;
    property Age: Integer read FAge write FAge;
  end;
```

### Views

Views are used when rendering HTML or template-based output. DMVCFramework supports server-side views with engines such as Mustache and TemplatePro.[2]

```pascal
procedure TMyController.ShowHome;
begin
  ViewData['title'] := 'Home';
  Render('home');
end;
```

### Controllers

Controllers receive HTTP requests, run logic, and return data or rendered views. They are the main entry points for routes in a DMVC app.[2]

```pascal
[MVCPath('/api/users')]
TUserController = class(TMVCController)
public
  [MVCPath]
  [MVCHTTPMethod([httpGET])]
  function GetUsers: TObjectList<TPerson>;
end;
```

## Routing and Request Handling

Routing in DMVCFramework is attribute-based, which means paths and HTTP methods are declared directly on controller classes and action methods.[2] This makes endpoint definitions readable and keeps request handling close to controller logic.[2]

### Key ideas

- Use `MVCPath` to define the route path.[2]
- Use `MVCHTTPMethod` to specify allowed HTTP verbs.[2]
- Use path parameters to extract values from the URL.[1][2]

```pascal
[MVCPath('/api/products')]
TProductController = class(TMVCController)
public
  [MVCPath('/($id)')]
  [MVCHTTPMethod([httpGET])]
  function GetProduct(const id: Integer): string;
end;

function TProductController.GetProduct(const id: Integer): string;
begin
  Result := 'Requested product id: ' + id.ToString;
end;
```

## Data Binding and Validation

DMVCFramework supports JSON serialization and deserialization, which helps bind request payloads to Delphi objects and return structured responses.[2] Validation is commonly applied before saving or processing incoming data, either manually in controller logic or through structured model checks.

### Common flow

- Read JSON request data into a Delphi object.[2]
- Validate required fields and business rules.
- Return clear HTTP errors for invalid input.

```pascal
procedure TUserController.CreateUser;
var
  LUser: TPerson;
begin
  LUser := Context.Request.BodyAs<TPerson>;
  if LUser.Name.Trim.IsEmpty then
    raise EMVCException.Create(400, 'Name is required');
  Render(201, 'User created');
end;
```

## Working with Databases

DMVCFramework includes MVCActiveRecord, an ORM that supports CRUD operations and works with several databases including PostgreSQL, MySQL, MariaDB, Firebird, InterBase, SQLite, and Microsoft SQL Server.[2] The framework also supports named queries, connection pooling, transaction management, and repository-style patterns for cleaner data access.[1][2]

### Example entity

```pascal
type
  [MVCNameCase(ncLowerCase)]
  [MVCTable('customers')]
  TCustomer = class(TMVCActiveRecord)
  private
    FID: Integer;
    FName: string;
  published
    [MVCColumn('id', [mcPrimaryKey, mcAutoGenerated])]
    property ID: Integer read FID write FID;

    [MVCColumn('name')]
    property Name: string read FName write FName;
  end;
```

### Example usage

```pascal
var
  Customer: TCustomer;
begin
  Customer := TCustomer.Create;
  Customer.Name := 'Alice';
  Customer.Insert;
end;
```

## Authentication and Authorization

DMVCFramework provides JWT authentication, HTTP Basic Authentication, custom authentication mechanisms, and middleware options such as JWT blacklist support.[2] These features help secure endpoints and control access based on tokens, credentials, or custom rules.[2]

### Typical authentication steps

- Accept user credentials.
- Verify identity.
- Issue a JWT token.
- Protect routes by validating the token on future requests.[2]

```pascal
procedure TAuthController.Login;
begin
  if IsValidUser('admin', 'secret') then
    Render(200, '{"token":"sample-jwt-token"}')
  else
    Render(401, 'Unauthorized');
end;
```

## Error Handling and Logging

DMVCFramework integrates with LoggerPro for comprehensive logging and includes production-oriented support for monitoring and debugging.[2] Good error handling usually means returning appropriate HTTP status codes, logging exceptions, and avoiding exposure of sensitive internal details.

### Good practices

- Return `400` for bad input.
- Return `401` or `403` for access issues.
- Return `500` for unexpected server failures.
- Log request context and exceptions for troubleshooting.[2]

```pascal
try
  DoWork;
  Render(200, 'Success');
except
  on E: Exception do
  begin
    LogE(E.Message);
    Render(500, 'Internal Server Error');
  end;
end;
```

## Middleware and Services

DMVCFramework includes a middleware system for request and response processing, and recent releases add built-in rate limiting middleware for both in-memory and Redis-backed setups.[1][2] Middleware is useful for cross-cutting concerns such as authentication, logging, CORS, compression, static file serving, and traffic control.[1][2]

### Example middleware registration

```pascal
Engine.AddMiddleware(
  TMVCRateLimitMiddleware.Create(
    100,
    60,
    'Too many requests. Please slow down.'
  )
);
```

### Why services matter

- Keep controller logic thin.
- Move reusable business logic into dedicated classes.
- Improve testability and maintainability.

```pascal
type
  TUserService = class
  public
    function GetDisplayName(const AName: string): string;
  end;

function TUserService.GetDisplayName(const AName: string): string;
begin
  Result := 'User: ' + AName;
end;
```

## Advanced Topics

DMVCFramework supports dependency injection, repository patterns, custom serialization behavior, WebSocket support, SSE, and performance-related features such as profiling and optimized middleware usage.[1][2] These topics become important when applications grow in size, traffic, and architectural complexity.[1][2]

### Dependency Injection

Dependency injection helps decouple controllers from concrete implementations, making the codebase easier to test and extend.[1][2]

```pascal
type
  IUserRepository = interface
    function GetAllNames: TArray<string>;
  end;

  [MVCPath('/api/accounts')]
  TAccountController = class(TMVCController)
  private
    FRepo: IUserRepository;
  public
    [MVCInject]
    constructor Create(ARepo: IUserRepository); reintroduce;
  end;
```

### Custom Components

Custom components can include middleware, serializers, helpers, or reusable controller bases that match a project's architecture.

```pascal
type
  TCustomAuditMiddleware = class(TMVCMiddleware)
  public
    procedure OnBeforeRouting(Context: TWebContext; var Handled: Boolean); override;
  end;
```

### Performance Optimization

- Use rate limiting and compression where appropriate.[1][2]
- Keep database queries efficient and avoid unnecessary serialization overhead.[2]
- Push business logic into services and repositories for cleaner profiling and tuning.[1][2]

```pascal
Context.Response.SetCustomHeader('Cache-Control', 'public, max-age=60');
```

## Testing in DMVC

The framework repository highlights a substantial unit testing base, with more than 250 tests noted in the project README.[2] Testing in a DMVC project typically includes unit tests for services and repositories, plus integration tests for controllers and HTTP responses.[2]

### What to test

- Controller routes and status codes.
- Service logic.
- Validation behavior.
- Database repository methods.
- Authentication workflows.

```pascal
procedure TUserServiceTest.ShouldReturnDisplayName;
var
  Service: TUserService;
begin
  Service := TUserService.Create;
  Assert.AreEqual('User: Alice', Service.GetDisplayName('Alice'));
end;
```

## Deployment and Best Practices

DMVCFramework supports deployment as standalone applications, Windows services, Linux daemons, Apache modules, IIS ISAPI extensions, and desktop-hosted applications.[2] A solid deployment setup should pair secure configuration, structured logging, tested middleware, and environment-based settings such as dotEnv support.[1][2]

### Best practices

- Keep controllers focused on HTTP concerns.
- Put business logic into services or repositories.[1]
- Use JWT and TLS-enabled deployment for secure APIs.[2]
- Centralize configuration with environment files where possible.[1][2]
- Add logging, validation, and testing before production release.[2]

```pascal
SetEnvironmentVariable('DMVC_ENV', 'production');
```

## Next Steps

This repository is a study guide, so improvements, corrections, examples, and discussion topics are welcome. Contributions through pull requests, issue discussions, and practical examples can make the notes more useful for beginners and experienced Delphi developers alike.

- Add your own code samples and explanations.
- Open discussions for patterns, debugging tips, or deployment issues.
- Compare beginner and advanced approaches in separate examples.
- Read the official documentation: [DMVCFramework Official Documentation](https://www.danieleteti.it/delphimvcframework_3_4_3_aluminium/).[1]
- Explore the source code and examples: [DMVCFramework GitHub Repository](https://github.com/danieleteti/delphimvcframework).[2]
