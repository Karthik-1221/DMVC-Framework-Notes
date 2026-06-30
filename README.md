# DMVC Framework Notes - Beginner to Advanced

A structured learning resource for the **DMVCFramework**, covering fundamentals through advanced concepts for building REST APIs and web applications in Delphi. This repository also includes supporting Delphi language, architecture, and productivity topics from the provided self-learning document so learners can connect framework knowledge with practical development skills.[1][2][3]

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
- [Additional Delphi Topics](#additional-delphi-topics)
- [Developer Productivity and IDE Tips](#developer-productivity-and-ide-tips)
- [Testing in DMVC](#testing-in-dmvc)
- [Deployment and Best Practices](#deployment-and-best-practices)
- [Next Steps](#next-steps)

## Introduction to DMVC

DMVCFramework is an open-source Delphi framework for building RESTful services, JSON-RPC APIs, and web applications using the MVC pattern.[1][2] It is popular because it combines routing, controllers, middleware, authentication, ORM support, server-side views, and flexible deployment options in one framework.[1][2]

### What it is

- A Delphi framework for web APIs and web applications.[2]
- Based on the MVC pattern, which separates application responsibilities into models, views, and controllers.[2]
- Capable of handling REST endpoints, static content, JSON payloads, and real-time features such as Server-Sent Events and WebSocket support.[1][2]

### Why it is used

- Improves code organization and maintainability.[1][2]
- Helps structure backend projects with reusable controllers and services.[2]
- Supports production features such as JWT authentication, middleware, and ORM-based database access.[1][2]

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

DMVCFramework supports several Delphi versions, including Delphi 10 Seattle through Delphi 13 Florence.[1][2] A standard setup includes installing Delphi, cloning the DMVCFramework repository, and building the package version that matches the IDE.[1][2]

### Basic steps

1. Install a supported Delphi version.[1][2]
2. Clone the framework repository.[2]
3. Open the correct package folder for your Delphi version.
4. Build and install the needed packages.
5. Create a project and register controllers through `TMVCEngine`.

```bash
git clone https://github.com/danieleteti/delphimvcframework.git
```

```pascal
FMVC := TMVCEngine.Create(Self);
FMVC.AddController(TMyController);
```

## Core Concepts

DMVCFramework follows the MVC pattern, where models handle data, views handle presentation, and controllers handle requests.[2] This separation makes the application easier to test, extend, and maintain.[2]

### Models

Models represent data and business rules. In DMVCFramework, models may be plain Delphi classes or database entities based on MVCActiveRecord.[2]

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

Views are useful for rendering HTML or template-based output. DMVCFramework supports server-side views with template systems such as Mustache and TemplatePro.[2]

```pascal
procedure TMyController.ShowHome;
begin
  ViewData['title'] := 'Home';
  Render('home');
end;
```

### Controllers

Controllers process HTTP requests and return data or rendered responses. They are central to defining API behavior in DMVCFramework.[2]

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

Routing in DMVCFramework is mainly attribute-based, so route definitions and allowed HTTP methods are attached directly to controller actions.[2] This keeps endpoint declarations compact and easy to read.[2]

### Key ideas

- Use `MVCPath` for route paths.[2]
- Use `MVCHTTPMethod` for HTTP verbs.[2]
- Use route parameters to capture values from URLs.[1][2]
- Use attributes consistently because they are a core Delphi topic and a core part of DMVC routing.[3][2]

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

DMVCFramework supports serialization and deserialization of JSON payloads, making it easier to bind request bodies to Delphi objects.[2] Validation should be applied before saving data or executing business logic so bad input can be rejected early.

### Common flow

- Read request data into a Delphi object.[2]
- Validate required values and formats.
- Return meaningful HTTP error responses.
- Prefer clean, local variable usage where inline variables improve readability.[3]

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

DMVCFramework includes MVCActiveRecord, which provides ORM support for CRUD operations and works with multiple relational databases such as PostgreSQL, MySQL, MariaDB, Firebird, InterBase, SQLite, and SQL Server.[2] This matches the Object-Relational Mapping topic in your document and is one of the most important practical parts of DMVC backend development.[3][2]

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

DMVCFramework provides support for JWT authentication, HTTP Basic Authentication, and custom authentication solutions.[2] These features help secure endpoints and control access to protected routes.[2]

### Typical flow

- Accept credentials.
- Validate the user.
- Issue a JWT token.
- Use the token to access protected resources.[2]

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

DMVCFramework integrates with LoggerPro and includes production-friendly logging support.[2] A good approach is to return proper HTTP status codes, log exceptions, and avoid exposing internal errors to the client.

### Good practices

- Use `400` for invalid input.
- Use `401` or `403` for authorization failures.
- Use `500` for unexpected server errors.
- Log enough context to make debugging easier.[2]

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

DMVCFramework includes middleware support for request and response processing, and recent versions add rate limiting middleware with in-memory and Redis-backed options.[1][2] Middleware is useful for authentication, CORS, logging, response shaping, compression, and traffic control.[1][2]

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

### Services

Service classes keep controllers small by moving business logic into reusable units. This also supports easier testing and cleaner refactoring, which is one of the self-learning topics from the document.[3]

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

DMVCFramework supports advanced capabilities such as dependency injection, repository patterns, custom serializers, SSE, WebSocket support, and performance-oriented design options.[1][2] These topics are useful when moving from simple APIs to larger and more maintainable backend systems.[1][2]

### Dependency Injection

Dependency injection reduces coupling by allowing controllers and services to depend on abstractions instead of concrete classes.[1][2] This topic also appears directly in your attached self-learning list.[3]

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

Custom components can be created for middleware, serialization helpers, reusable controller bases, or audit pipelines.

```pascal
type
  TCustomAuditMiddleware = class(TMVCMiddleware)
  public
    procedure OnBeforeRouting(Context: TWebContext; var Handled: Boolean); override;
  end;
```

### Performance Optimization

- Use rate limiting and compression where suitable.[1][2]
- Keep queries efficient and avoid unnecessary serialization.[2]
- Prefer stateless API design when possible for scalability.[3]

```pascal
Context.Response.SetCustomHeader('Cache-Control', 'public, max-age=60');
```

## Additional Delphi Topics

The provided document lists several Delphi topics that are useful alongside DMVCFramework, including Generics, Escape Characters, In-Line variables, Attributes, Dependency Injection, Refactoring, PPL, Anonymous Methods, Scope-Based Resource Management, Sqids Encoding, RAII, Stateless Architecture, Sacroscent, and Object-Relational Mapping.[3] These topics are included here as separate headings so the repository can also act as a broader Delphi study reference.[3]

### Generics

Generics let you build reusable classes and methods that work with multiple data types. In DMVC projects, they are useful for repositories, collections, wrappers, and generic response handling.[3]

```pascal
var
  Users: TObjectList<TPerson>;
begin
  Users := TObjectList<TPerson>.Create(True);
end;
```

### Escape Characters

Escape characters are commonly used when writing JSON strings, log messages, or special text values manually. Even when serializers are preferred, it is still useful to understand escaping basics.[3]

```pascal
var
  JsonText: string;
begin
  JsonText := '{"message":"Hello\nDMVC"}';
end;
```

### In-Line Variables

Your document specifically explains inline variable declarations and their limited scope inside code blocks.[3] They can make controller and service code shorter by declaring variables exactly where they are needed.[3]

```pascal
procedure TSampleController.Test;
begin
  var LCount := 10;
  Render(LCount.ToString);
end;
```

### Attributes

Attributes are metadata annotations in Delphi, and they are heavily used in DMVCFramework for routes, HTTP methods, serialization settings, and injection patterns.[3][2]

```pascal
[MVCPath('/api/orders')]
[MVCDoc('Order endpoints')]
TOrderController = class(TMVCController)
end;
```

### Adding Unit to the Uses Section

Managing the `uses` section correctly helps keep dependencies clear and compilation clean. It is a small topic, but it matters when organizing multi-unit Delphi projects.[3]

```pascal
uses
  System.SysUtils,
  System.Classes,
  MVCFramework;
```

### Dependency Injection

Dependency injection makes code more modular and testable by separating abstractions from implementations. It is useful in service registration, repository access, and controller design.[3][1][2]

```pascal
type
  IEmailService = interface
    procedure Send(const ATo, ABody: string);
  end;
```

### Refactoring

Refactoring improves code quality without changing behavior. In DMVC projects, this often means moving duplicated logic out of controllers and into services, helpers, or repositories.[3]

```pascal
function NormalizeName(const AValue: string): string;
begin
  Result := AValue.Trim.ToLower;
end;
```

### Parallel Programming Library (PPL)

PPL is useful for background processing and concurrent tasks in Delphi. It should be applied carefully in server applications so shared-state issues are avoided.[3]

```pascal
TTask.Run(
  procedure
  begin
    GenerateReport;
  end
);
```

### Anonymous Methods

Anonymous methods are useful for short callbacks, background tasks, and event-based logic. They are often used together with asynchronous or task-based code in Delphi.[3]

```pascal
var
  Proc: TProc;
begin
  Proc := procedure begin Log('Done'); end;
  Proc();
end;
```

### Scope-Based Resource Management

Scope-based resource management aims to make cleanup safer by limiting resource lifetime to a clear code scope. It helps reduce leaks and resource misuse.[3]

```pascal
procedure UseList;
begin
  var LList := TStringList.Create;
  try
    LList.Add('Scoped resource');
  finally
    LList.Free;
  end;
end;
```

### Sqids Encoding

Sqids encoding can be used to convert numeric IDs into friendlier public identifiers. This is useful when exposing IDs in URLs or APIs without showing raw sequential values.[3]

```pascal
function EncodePublicID(const AID: Integer): string;
begin
  Result := 'sqid_' + AID.ToString;
end;
```

### Resource Acquisition Is Initialization (RAII)

RAII is a resource management pattern where object lifetime controls acquisition and cleanup. In Delphi, this idea is often discussed together with interface-based lifetime or scoped wrappers.[3]

```pascal
procedure ProcessData;
begin
  var LStream := TStringStream.Create('sample');
  try
    Render(LStream.DataString);
  finally
    LStream.Free;
  end;
end;
```

### Stateless Architecture

Stateless architecture means each request contains all the information needed for processing, without relying on server-side session memory. This aligns well with REST API design and scalable backend systems.[3]

```pascal
procedure TTokenController.Profile;
begin
  Render(200, 'Read identity from JWT instead of session state');
end;
```

### Sacroscent

This topic appears in the provided document as a listed heading, so it is preserved here as a placeholder study item.[3] It can be expanded later with a proper explanation once the intended meaning or context is clarified.[3]

```pascal
// Placeholder topic from source document
```

### Object-Relational Mapping

Object-relational mapping connects database tables with Delphi classes. In DMVCFramework, MVCActiveRecord is the main ORM-based feature used for this pattern.[3][2]

```pascal
Customers := TMVCActiveRecord.All<TCustomer>;
```

## Developer Productivity and IDE Tips

Your document also includes practical IDE and workflow items such as `Ctrl + Shift + G`, auto formatting, and shortcuts like `Ctrl + D`, `Ctrl + A`, and `Ctrl + Shift + J`.[3] These small habits improve speed and consistency during daily Delphi development.[3]

### Ctrl + Shift + G

This topic is listed in the source document and can be documented here as an IDE navigation or productivity shortcut based on your preferred workflow reference.[3]

```text
Ctrl + Shift + G
```

### Auto Formatting

Auto formatting keeps code consistent and easier to read, especially in larger projects or team environments.[3]

```pascal
begin
  Render('Formatted code is easier to maintain');
end;
```

### Shortcuts

The source document lists `Ctrl + D`, `Ctrl + A`, and `Ctrl + Shift + J` as useful shortcuts.[3] Keeping these in the README helps beginners build better productivity habits while learning DMVC and Delphi together.[3]

```text
Ctrl + D
Ctrl + A
Ctrl + Shift + J
```

## Testing in DMVC

The project repository notes a large automated test base, with more than 250 tests highlighted in the README.[2] In practice, DMVC testing usually includes unit tests for services and repositories, plus integration tests for routes, controllers, and API responses.[2]

### What to test

- Controller routes and status codes.
- Service logic.
- Validation behavior.
- Repository methods.
- Authentication flows.

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

DMVCFramework supports deployment as standalone applications, Windows services, Linux daemons, Apache modules, IIS ISAPI extensions, and desktop-hosted applications.[2] A good production setup should combine secure configuration, validation, structured logging, tested middleware, and clear environment-based settings.[1][2]

### Best practices

- Keep controllers focused on HTTP concerns.
- Move business logic into services or repositories.[1]
- Use JWT and TLS-enabled deployment for security.[2]
- Organize code so refactoring is easy as the application grows.[3]
- Use formatting and shortcuts consistently to improve daily development flow.[3]

```pascal
SetEnvironmentVariable('DMVC_ENV', 'production');
```

## Next Steps

This repository is meant to be a practical study guide, so contributions, improvements, and topic expansions are encouraged. Adding examples, clarifying placeholders, and discussing best practices can make it more useful for both beginners and experienced Delphi developers.[3]

- Add more examples for each DMVC section and each Delphi topic from the document.[3]
- Expand placeholders such as Sacroscent and IDE shortcut meanings with your own notes.[3]
- Open discussions for architecture decisions, debugging patterns, and deployment setups.
- Read the official documentation: [DMVCFramework Official Documentation](https://www.danieleteti.it/delphimvcframework_3_4_3_aluminium/).[1]
- Explore the source code: [DMVCFramework GitHub Repository](https://github.com/danieleteti/delphimvcframework).[2]
