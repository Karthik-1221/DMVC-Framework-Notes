# DMVC Framework Notes - Beginner to Advanced

A structured learning resource for the **DMVCFramework**, designed to take a learner from the basics of building a web API in Delphi to advanced application design, database integration, authentication, middleware, testing, and deployment. DMVCFramework is an open-source Delphi framework for RESTful services, JSON-RPC APIs, and web applications built around MVC, attribute-based routing, middleware, Active Record support, and production-friendly backend patterns.[1][2][3]

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

DMVCFramework is used to build HTTP-based backend applications in Delphi, especially REST APIs, JSON-RPC services, and server-side web applications.[1][2] It follows the MVC pattern, where controllers manage requests, models represent data and business rules, and views render output when server-side templates are needed.[2][3]

### What it is

- A backend and web framework for Delphi applications.[2]
- Built around MVC architecture and attribute-based routing.[3]
- Supports JSON serialization, middleware, authentication, Active Record, and multiple deployment models.[1][2]

### Why it is used

- Keeps backend code organized into predictable layers.[3]
- Reduces manual HTTP plumbing by handling routing, binding, serialization, and response generation.[1][2]
- Makes it easier to scale from small demo APIs to larger production services.[1][2]

```pascal
[MVCPath('/api/hello')]
THelloController = class(TMVCController)
public
  [MVCPath]
  [MVCHTTPMethod([httpGET])]
  procedure SayHello;
end;

procedure THelloController.SayHello;
begin
  Render(200, 'Hello from DMVCFramework');
end;
```

In this example, the controller responds to `GET /api/hello`. The `MVCPath` attribute defines the resource path, and `MVCHTTPMethod` restricts the action to a GET request.[3]

## Setting Up the Environment

A typical DMVCFramework setup begins with installing a supported Delphi version, cloning the framework repository, and opening the matching package for the IDE version in use.[1][2] Once installed, a project usually creates a `TMVCEngine` instance, registers controllers, and starts listening for HTTP requests.[2]

### Basic setup steps

1. Install Delphi.
2. Clone the DMVCFramework repository.[2]
3. Build the appropriate packages for your Delphi version.
4. Create a web project or use an existing DMVC sample.
5. Register controllers and configure middleware in the engine.

```bash
git clone https://github.com/danieleteti/delphimvcframework.git
```

### Minimal engine setup

```pascal
uses
  MVCFramework,
  MVCFramework.Commons,
  Web.ReqMulti,
  Web.WebBroker,
  MyApp.MainController;

type
  TMyWebModule = class(TWebModule)
  private
    FMVC: TMVCEngine;
  public
    procedure WebModuleCreate(Sender: TObject);
    procedure WebModuleDestroy(Sender: TObject);
  end;

procedure TMyWebModule.WebModuleCreate(Sender: TObject);
begin
  FMVC := TMVCEngine.Create(Self,
    procedure(Config: TMVCConfig)
    begin
      Config[TMVCConfigKey.DefaultContentType] := TMVCMediaType.APPLICATION_JSON;
      Config[TMVCConfigKey.DefaultContentCharset] := 'utf-8';
    end);

  FMVC.AddController(TMainController);
end;

procedure TMyWebModule.WebModuleDestroy(Sender: TObject);
begin
  FMVC.Free;
end;
```

This setup creates the engine, defines basic configuration, and registers a controller. In real projects, this is also where middleware, authentication handlers, and environment-specific settings are often added.[1][2]

## Core Concepts

The core of DMVCFramework is the MVC pattern. Controllers receive and process requests, models hold data and business logic, and views are used when the server must render HTML rather than only return JSON.[2][3]

### Models

Models represent application data. In simple cases, they are plain Delphi classes; in database-backed applications, they often become Active Record entities mapped to tables.[2][4]

```pascal
type
  TUser = class
  private
    FID: Integer;
    FUsername: string;
    FEmail: string;
  public
    property ID: Integer read FID write FID;
    property Username: string read FUsername write FUsername;
    property Email: string read FEmail write FEmail;
  end;
```

A plain model like this is useful for request bodies, service-layer objects, or simple JSON serialization. Not every model has to map directly to a database table.

### Views

Views are useful when returning HTML pages from the server. Although many DMVC applications are API-first, server-side rendering is still supported.[2]

```pascal
procedure TMainController.Home;
begin
  ViewData['Title'] := 'DMVC Notes';
  ViewData['Message'] := 'Welcome to the home page';
  Render('home');
end;
```

Here, the controller sets values in `ViewData` and renders a template named `home`. This is helpful for dashboards, admin pages, and hybrid applications.

### Controllers

Controllers are the entry points for HTTP requests. Each controller inherits from `TMVCController`, and actions are ordinary methods marked with routing attributes.[3]

```pascal
[MVCPath('/api/users')]
TUsersController = class(TMVCController)
public
  [MVCPath]
  [MVCHTTPMethod([httpGET])]
  procedure GetUsers;

  [MVCPath('/($id)')]
  [MVCHTTPMethod([httpGET])]
  procedure GetUserByID(id: Integer);
end;

procedure TUsersController.GetUsers;
begin
  Render<TArray<string>>(['Alice', 'Bob', 'Charlie']);
end;

procedure TUsersController.GetUserByID(id: Integer);
begin
  Render(Format('User id requested: %d', [id]));
end;
```

This design keeps routing close to the code that handles the request. It also makes controllers easier to read and maintain than manually parsing paths.

## Routing and Request Handling

Routing is one of the most important parts of DMVCFramework. The framework uses attributes such as `MVCPath` and `MVCHTTPMethod` to map URLs and HTTP verbs to controller actions.[3]

### Key ideas

- `MVCPath` defines the URL path for a controller or action.[3]
- `MVCHTTPMethod` restricts which HTTP methods can call that action.[3]
- URL parameters such as `($id)` are automatically extracted and passed to action parameters.[3]
- Strongly typed action parameters help keep code clean and explicit.[3]

### Route example with parameters

```pascal
[MVCPath('/api/orders')]
TOrdersController = class(TMVCController)
public
  [MVCPath('/($orderid)/items/($itemid)')]
  [MVCHTTPMethod([httpGET])]
  procedure GetOrderItem(orderid: Integer; itemid: Integer);
end;

procedure TOrdersController.GetOrderItem(orderid, itemid: Integer);
begin
  Render(Format('Order %d, Item %d', [orderid, itemid]));
end;
```

A request to `/api/orders/25/items/3` will automatically pass `25` and `3` into the action method. This is cleaner than reading and converting route values manually.[3]

### Reading request information

```pascal
procedure TOrdersController.Search;
begin
  var LKeyword := Context.Request.QueryStringParam('q');
  var LPage := Context.Request.QueryStringParam('page');
  Render(Format('Search=%s, Page=%s', [LKeyword, LPage]));
end;
```

The `Context` object gives access to query string values, headers, body data, cookies, and response settings. It is one of the main objects used inside a controller action.[3]

## Data Binding and Validation

DMVCFramework can deserialize JSON request bodies into Delphi objects and serialize Delphi objects back into JSON responses.[1][2] This makes API actions much easier to write because the controller can work with typed data instead of raw text.

### Binding JSON into an object

```pascal
type
  TCreateUserRequest = class
  private
    FUsername: string;
    FEmail: string;
    FAge: Integer;
  public
    property Username: string read FUsername write FUsername;
    property Email: string read FEmail write FEmail;
    property Age: Integer read FAge write FAge;
  end;

procedure TUsersController.CreateUser;
begin
  var LRequest := Context.Request.BodyAs<TCreateUserRequest>;
  try
    if LRequest.Username.Trim.IsEmpty then
      raise EMVCException.Create(400, 'Username is required');

    if LRequest.Email.Trim.IsEmpty then
      raise EMVCException.Create(400, 'Email is required');

    if LRequest.Age < 18 then
      raise EMVCException.Create(400, 'Age must be at least 18');

    Render(201, 'User created successfully');
  finally
    LRequest.Free;
  end;
end;
```

This example reads the JSON body into a Delphi class and applies simple validation rules before continuing. The idea is to reject bad input early and keep service logic safe.

### Returning structured JSON

```pascal
type
  TUserResponse = class
  private
    FID: Integer;
    FName: string;
    FStatus: string;
  public
    property ID: Integer read FID write FID;
    property Name: string read FName write FName;
    property Status: string read FStatus write FStatus;
  end;

procedure TUsersController.GetProfile;
begin
  var LResponse := TUserResponse.Create;
  try
    LResponse.ID := 101;
    LResponse.Name := 'Alice';
    LResponse.Status := 'Active';
    Render(LResponse);
  finally
    LResponse.Free;
  end;
end;
```

This is a cleaner approach than manually creating JSON strings because the framework handles serialization for you.

## Working with Databases

DMVCFramework includes MVCActiveRecord, which implements an Active Record pattern for database access.[2][4] In this model, a Delphi class maps to a database table, and the class can perform operations such as insert, update, delete, and lookup.[4]

### Active Record entity example

```pascal
type
  [MVCTable('customers')]
  TCustomer = class(TMVCActiveRecord)
  private
    FID: Integer;
    FFullName: string;
    FEmail: string;
    FCreatedAt: TDateTime;
  published
    [MVCColumn('id', [mcPrimaryKey, mcAutoGenerated])]
    property ID: Integer read FID write FID;

    [MVCColumn('full_name')]
    property FullName: string read FFullName write FFullName;

    [MVCColumn('email')]
    property Email: string read FEmail write FEmail;

    [MVCColumn('created_at')]
    property CreatedAt: TDateTime read FCreatedAt write FCreatedAt;
  end;
```

This class maps the `customers` table to a Delphi class. The attributes describe how class properties match table columns and identify the primary key.[4]

### Insert data

```pascal
procedure TCustomersController.CreateCustomer;
begin
  var LCustomer := Context.Request.BodyAs<TCustomer>;
  try
    LCustomer.CreatedAt := Now;
    LCustomer.Insert;
    Render(201, LCustomer);
  finally
    LCustomer.Free;
  end;
end;
```

### Query by primary key

```pascal
procedure TCustomersController.GetCustomerByID(id: Integer);
begin
  var LCustomer := TMVCActiveRecord.GetByPK<TCustomer>(id);
  try
    if not Assigned(LCustomer) then
      raise EMVCException.Create(404, 'Customer not found');

    Render(LCustomer);
  finally
    LCustomer.Free;
  end;
end;
```

### Update existing data

```pascal
procedure TCustomersController.UpdateCustomer(id: Integer);
begin
  var LExisting := TMVCActiveRecord.GetByPK<TCustomer>(id);
  if not Assigned(LExisting) then
    raise EMVCException.Create(404, 'Customer not found');

  try
    var LInput := Context.Request.BodyAs<TCustomer>;
    try
      LExisting.FullName := LInput.FullName;
      LExisting.Email := LInput.Email;
      LExisting.Update;
      Render(200, LExisting);
    finally
      LInput.Free;
    end;
  finally
    LExisting.Free;
  end;
end;
```

This style is useful because database logic stays close to the entity class, reducing the need to write repetitive SQL for common CRUD operations.[4]

## Authentication and Authorization

DMVCFramework supports authentication mechanisms such as HTTP Basic Authentication and JWT-based authentication.[2][5] JWT is especially useful for stateless APIs because the token contains claims that can be validated on each request without storing session data on the server.[5]

### Simple login example

```pascal
procedure TAuthController.Login;
begin
  var LUsername := Context.Request.BodyParam('username');
  var LPassword := Context.Request.BodyParam('password');

  if (LUsername = 'admin') and (LPassword = 'secret') then
    Render(200, '{"token":"sample-jwt-token"}')
  else
    Render(401, 'Invalid credentials');
end;
```

### Protecting an endpoint

```pascal
[MVCPath('/api/secure')]
TSecureController = class(TMVCController)
public
  [MVCPath('/profile')]
  [MVCHTTPMethod([httpGET])]
  procedure GetProfile;
end;

procedure TSecureController.GetProfile;
begin
  Render(200, 'Authorized access to secure profile');
end;
```

In a real application, this controller would be protected by authentication middleware or a custom authorization check. The important idea is that login issues a token, and later requests present that token for validation.[5][6]

### Why JWT is useful

- Works well in stateless architectures.[5]
- Fits REST APIs and SPA/mobile clients.[5]
- Can carry user identity and permission claims.[6][5]

## Error Handling and Logging

A backend API should fail in predictable ways. Good error handling means validating input, returning appropriate status codes, logging exceptions, and keeping internal details out of public responses.[2]

### Exception handling example

```pascal
procedure TReportController.Generate;
begin
  try
    if Context.Request.QueryStringParam('type').Trim.IsEmpty then
      raise EMVCException.Create(400, 'Report type is required');

    Render(200, 'Report generated');
  except
    on E: EMVCException do
      Render(E.HTTPStatusCode, E.Message);
    on E: Exception do
    begin
      LogE(E.ClassName + ': ' + E.Message);
      Render(500, 'Internal server error');
    end;
  end;
end;
```

This pattern distinguishes expected API errors from unexpected system failures. That separation improves both debugging and client-side behavior.

### Logging ideas

- Log route, method, and request identifiers.
- Log exception class and message.
- Avoid logging secrets such as passwords or raw tokens.
- Add enough context to reproduce the issue later.

## Middleware and Services

Middleware processes requests and responses before or after controller actions. Services hold business logic that should not live directly inside controllers.[1][2]

### Middleware example

```pascal
type
  TRequestTimeMiddleware = class(TMVCMiddleware)
  public
    procedure OnBeforeRouting(Context: TWebContext; var Handled: Boolean); override;
    procedure OnAfterControllerAction(Context: TWebContext; const AActionName: string; const Handled: Boolean); override;
  end;

procedure TRequestTimeMiddleware.OnBeforeRouting(Context: TWebContext; var Handled: Boolean);
begin
  Context.Data['RequestStartedAt'] := Now;
end;

procedure TRequestTimeMiddleware.OnAfterControllerAction(Context: TWebContext; const AActionName: string; const Handled: Boolean);
begin
  var LStart := Context.Data['RequestStartedAt'];
  Context.Response.CustomHeaders.Values['X-Request-Tracked'] := 'true';
end;
```

Middleware is ideal for cross-cutting tasks such as logging, authentication, CORS, rate limiting, response headers, and request metrics.[1]

### Service example

```pascal
type
  TInvoiceService = class
  public
    function CalculateTotal(const APrice: Currency; const AQuantity: Integer): Currency;
    function BuildInvoiceMessage(const ACustomer: string; const ATotal: Currency): string;
  end;

function TInvoiceService.CalculateTotal(const APrice: Currency; const AQuantity: Integer): Currency;
begin
  Result := APrice * AQuantity;
end;

function TInvoiceService.BuildInvoiceMessage(const ACustomer: string; const ATotal: Currency): string;
begin
  Result := Format('Invoice for %s is %.2f', [ACustomer, ATotal]);
end;
```

### Using a service inside a controller

```pascal
procedure TInvoiceController.GetTotal;
begin
  var LService := TInvoiceService.Create;
  try
    var LTotal := LService.CalculateTotal(199.50, 3);
    Render(LService.BuildInvoiceMessage('Alice', LTotal));
  finally
    LService.Free;
  end;
end;
```

This design keeps HTTP-specific code in the controller and calculation logic in a reusable service class.

## Advanced Topics

As applications grow, projects often need dependency injection, custom components, better performance, clearer abstractions, and more reusable infrastructure.[1][2] DMVCFramework supports this style of growth with flexible controller design, middleware, Active Record, and authentication patterns.[1][2]

### Dependency Injection

Dependency injection helps remove tight coupling between a controller and the class it depends on. Instead of creating dependencies inside the controller, they are provided from outside.[1]

```pascal
type
  IUserRepository = interface
    ['{A1E73A26-4AE0-4E5F-8E29-B9B7DB66FE00}']
    function FindUserNameByID(const AID: Integer): string;
  end;

  TUserRepository = class(TInterfacedObject, IUserRepository)
  public
    function FindUserNameByID(const AID: Integer): string;
  end;

function TUserRepository.FindUserNameByID(const AID: Integer): string;
begin
  if AID = 1 then
    Result := 'Alice'
  else
    Result := 'Unknown';
end;
```

```pascal
type
  [MVCPath('/api/accounts')]
  TAccountsController = class(TMVCController)
  private
    FRepo: IUserRepository;
  public
    constructor Create(const ARepo: IUserRepository); reintroduce;

    [MVCPath('/($id)')]
    [MVCHTTPMethod([httpGET])]
    procedure GetAccountName(id: Integer);
  end;

constructor TAccountsController.Create(const ARepo: IUserRepository);
begin
  inherited Create;
  FRepo := ARepo;
end;

procedure TAccountsController.GetAccountName(id: Integer);
begin
  Render(FRepo.FindUserNameByID(id));
end;
```

This makes the controller easier to test because a fake repository can be provided during unit testing.

### Custom Components

A custom component in a DMVC application might be a reusable middleware, a base controller, a response helper, or a request pipeline extension.

```pascal
type
  TApiResponseHelper = class
  public
    class procedure RenderSuccess(const AController: TMVCController; const AData: string);
    class procedure RenderError(const AController: TMVCController; const AMessage: string; const AStatus: Integer);
  end;

class procedure TApiResponseHelper.RenderSuccess(const AController: TMVCController; const AData: string);
begin
  AController.Render(200, '{"success":true,"data":"' + AData + '"}');
end;

class procedure TApiResponseHelper.RenderError(const AController: TMVCController; const AMessage: string; const AStatus: Integer);
begin
  AController.Render(AStatus, '{"success":false,"error":"' + AMessage + '"}');
end;
```

This kind of helper reduces repeated response formatting across controllers.

### Performance Optimization

Performance work in DMVC applications usually focuses on efficient routing, lighter payloads, optimized database queries, appropriate middleware usage, and stateless API design.[1][4] The goal is to reduce response time and resource usage without making the codebase hard to maintain.

```pascal
procedure TProductsController.ListProducts;
begin
  Context.Response.CustomHeaders.Values['Cache-Control'] := 'public, max-age=60';
  Render<TArray<string>>(['Keyboard', 'Mouse', 'Monitor']);
end;
```

Simple response caching headers like this can reduce repeated traffic for frequently requested read-only endpoints.

## Additional Delphi Topics

A strong DMVC developer also benefits from solid Delphi language fundamentals. Topics such as generics, attributes, anonymous methods, resource management, refactoring, and ORM concepts help make framework code cleaner and more professional.

### Generics

Generics allow classes, records, and methods to work with different data types without rewriting the same logic. They are especially useful for collections, repository abstractions, wrappers, and utility code.

```pascal
type
  TApiResponse<T> = class
  private
    FSuccess: Boolean;
    FData: T;
  public
    property Success: Boolean read FSuccess write FSuccess;
    property Data: T read FData write FData;
  end;

procedure TSampleController.GenericResponse;
begin
  var LResponse := TApiResponse<string>.Create;
  try
    LResponse.Success := True;
    LResponse.Data := 'Generic payload';
    Render(LResponse);
  finally
    LResponse.Free;
  end;
end;
```

This approach becomes powerful when a project needs a consistent API response shape for many different data types.

### Escape Characters

Escape characters are important when strings contain quotes, backslashes, newlines, or tab characters. Even though most JSON should be produced by serializers, understanding escaping prevents subtle bugs.

```pascal
procedure TSampleController.ShowEscapedJson;
begin
  var LJson := '{"title":"DMVC Notes","message":"Line1\nLine2","path":"C:\\Temp\\app.log"}';
  Render(200, LJson);
end;
```

### In-Line Variables

Inline variables allow variables to be declared closer to where they are used. This often improves readability by reducing the distance between declaration and usage.

```pascal
procedure TMathController.Calculate;
begin
  var A := 10;
  var B := 20;
  var Sum := A + B;
  Render(Format('Sum = %d', [Sum]));
end;
```

This style is especially helpful in smaller code blocks, parsing steps, and temporary controller logic.

### Attributes

Attributes are metadata applied to classes, methods, and properties. DMVCFramework relies heavily on attributes for routing and configuration.[3]

```pascal
[MVCPath('/api/reports')]
TReportsController = class(TMVCController)
public
  [MVCPath('/daily')]
  [MVCHTTPMethod([httpGET])]
  procedure Daily;
end;

procedure TReportsController.Daily;
begin
  Render('Daily report');
end;
```

Without attributes like these, routing would require more manual registration and less expressive code.

### Adding Unit to the Uses Section

The `uses` section defines which units are available to the current unit. Keeping it clean helps compilation, readability, and dependency management.

```pascal
unit MyApp.UsersController;

interface

uses
  System.SysUtils,
  System.Classes,
  MVCFramework,
  MVCFramework.Commons,
  MyApp.UserService,
  MyApp.Models;
```

A good practice is to group system units, framework units, and project units in a logical order.

### Dependency Injection

Dependency injection improves modularity and testability by pushing dependency creation out of the class that uses them. This is especially useful for repositories, services, loggers, and external clients.

```pascal
type
  ILogger = interface
    ['{BE09A18C-C3A7-4BB8-A7A7-B1B299BFF000}']
    procedure Info(const AMsg: string);
  end;

  TConsoleLogger = class(TInterfacedObject, ILogger)
  public
    procedure Info(const AMsg: string);
  end;

procedure TConsoleLogger.Info(const AMsg: string);
begin
  OutputDebugString(PChar(AMsg));
end;
```

Once interfaces like this exist, controllers and services can depend on `ILogger` rather than a specific logger implementation.

### Refactoring

Refactoring is the process of improving code structure without changing behavior. In DMVC projects, common refactoring steps include extracting services, moving validation into helpers, and replacing duplicated controller logic.

```pascal
function NormalizeEmail(const AEmail: string): string;
begin
  Result := Trim(LowerCase(AEmail));
end;

procedure TUsersController.Save;
begin
  var LEmail := NormalizeEmail(Context.Request.BodyParam('email'));
  Render('Saved ' + LEmail);
end;
```

Small improvements like this make code easier to reuse and easier to test.

### Parallel Programming Library (PPL)

PPL provides abstractions such as tasks for concurrent execution. In web applications, it is best used for non-request-critical work such as report generation, notifications, or post-processing.

```pascal
uses
  System.Threading;

procedure TReportsController.GenerateAsync;
begin
  TTask.Run(
    procedure
    begin
      Sleep(3000);
      TFile.WriteAllText('report.txt', 'Report completed');
    end);

  Render(202, 'Report generation started');
end;
```

This lets the API return immediately while longer work continues in the background.

### Anonymous Methods

Anonymous methods are unnamed procedures or functions that can be passed around like values. They are often used in callbacks, asynchronous code, and small local behaviors.

```pascal
procedure ExecuteAction(const AProc: TProc);
begin
  AProc();
end;

procedure TSampleController.RunCallback;
begin
  ExecuteAction(
    procedure
    begin
      Log('Callback executed');
    end);

  Render('Callback finished');
end;
```

They are useful, but too many nested anonymous methods can make code harder to read.

### Scope-Based Resource Management

This style keeps resources alive only within the block where they are needed. It helps avoid leaks and makes ownership easier to understand.

```pascal
procedure TFileController.ReadConfig;
begin
  var LLines := TStringList.Create;
  try
    LLines.LoadFromFile('config.ini');
    Render(LLines.Text);
  finally
    LLines.Free;
  end;
end;
```

The resource is created, used, and released in one place, which makes maintenance safer.

### Sqids Encoding

Sqids-style encoding is useful when public URLs should not expose raw numeric database IDs. Even if the exact library differs, the concept is the same: translate internal IDs into safer external tokens.

```pascal
function EncodePublicID(const AID: Integer): string;
begin
  Result := 'USR' + IntToHex(AID, 6);
end;

procedure TUsersController.GetPublicID;
begin
  Render(EncodePublicID(25));
end;
```

This is useful in APIs where sequential integer IDs should not be obvious to clients.

### Resource Acquisition Is Initialization (RAII)

RAII is a design idea where acquiring a resource is tied to object lifetime, so cleanup happens predictably. Delphi often uses `try..finally`, interfaces, or wrapper objects to achieve a similar effect.

```pascal
procedure TStreamController.ReadText;
begin
  var LStream := TStringStream.Create('Hello DMVC');
  try
    Render(LStream.DataString);
  finally
    LStream.Free;
  end;
end;
```

The important principle is that resource ownership is explicit and cleanup is guaranteed.

### Stateless Architecture

A stateless system does not keep per-user session state in server memory between requests. Each request must carry everything needed for processing, such as a JWT token or request parameters.[5]

```pascal
procedure TProfileController.Me;
begin
  var LAuthHeader := Context.Request.Headers['Authorization'];
  if LAuthHeader.Trim.IsEmpty then
    raise EMVCException.Create(401, 'Missing token');

  Render('Profile resolved from token');
end;
```

This model improves scalability because any server instance can handle any request.

### Sacroscent

This topic can be kept as a placeholder heading until a precise technical meaning is decided for the repository. If it was intended as a special principle, naming convention, or design rule, it can later be expanded into a dedicated note.

```text
Placeholder topic for future clarification
```

### Object-Relational Mapping

ORM maps database tables to objects so application code can work with entities instead of manual row-by-row handling. In DMVCFramework, MVCActiveRecord is the most visible ORM-oriented feature.[4]

```pascal
procedure TCustomersController.ListAll;
begin
  var LCustomers := TMVCActiveRecord.All<TCustomer>;
  try
    Render<TObjectList<TCustomer>>(LCustomers);
  except
    LCustomers.Free;
    raise;
  end;
end;
```

This pattern makes CRUD-heavy applications easier to write, though developers should still understand SQL and query performance.

## Developer Productivity and IDE Tips

Good productivity habits improve both learning speed and code quality. Clean formatting, correct unit organization, and efficient shortcut usage save time in everyday Delphi development.

### Ctrl + Shift + G

This heading can be used to document a team-specific or IDE-specific shortcut workflow. Repository notes can later expand this with the exact action used in the preferred Delphi IDE setup.

```text
Document the exact IDE action here
```

### Auto Formatting

Consistent formatting makes code easier to review and maintain. It becomes especially important as projects grow or multiple developers work on the same codebase.

```pascal
procedure TSampleController.CleanCode;
begin
  if True then
    Render('Formatted code is easier to read');
end;
```

### Shortcuts

Useful editor shortcuts help with navigation, refactoring, and code editing speed. A repository like this can maintain a small list of high-value shortcuts for daily use.

```text
Ctrl + D
Ctrl + A
Ctrl + Shift + J
```

## Testing in DMVC

The DMVCFramework project highlights a large automated test base, showing that testing is an important part of the ecosystem.[2] In application code, the most common layers to test are services, repositories, and controller behavior.[2]

### Unit test for a service

```pascal
uses
  DUnitX.TestFramework;

type
  [TestFixture]
  TInvoiceServiceTests = class
  public
    [Test]
    procedure CalculateTotal_ShouldMultiplyPriceAndQuantity;
  end;

procedure TInvoiceServiceTests.CalculateTotal_ShouldMultiplyPriceAndQuantity;
begin
  var LService := TInvoiceService.Create;
  try
    Assert.AreEqual<Currency>(598.50, LService.CalculateTotal(199.50, 3));
  finally
    LService.Free;
  end;
end;
```

### Controller-level testing idea

- Test route behavior.
- Test status codes.
- Test JSON output structure.
- Test authentication failures.
- Test validation rules for bad input.

Testing becomes easier when controllers are thin and most business logic lives in services or repositories.

## Deployment and Best Practices

DMVCFramework can be deployed in multiple ways, including standalone applications, Windows services, Linux daemons, Apache modules, and IIS ISAPI extensions.[2] A production-ready deployment should focus on security, environment-based configuration, structured logging, and stateless request handling.[1][2]

### Best practices

- Keep controllers small and focused on HTTP concerns.
- Move business logic into services and repositories.
- Prefer structured serialization over manual JSON string building.
- Use JWT or another secure authentication approach for protected routes.[5]
- Validate inputs early and return clear status codes.
- Use logging and middleware to support monitoring and debugging.
- Design for stateless behavior when building scalable APIs.[5]

```pascal
procedure ConfigureEnvironment;
begin
  SetEnvironmentVariable('DMVC_ENV', 'production');
  SetEnvironmentVariable('DMVC_LOG_LEVEL', 'warning');
end;
```

## Next Steps

This repository can grow into a complete Delphi backend handbook by adding more real-world examples, request and response samples, architectural patterns, and step-by-step mini projects. Contributions, discussions, and topic expansions are encouraged as the notes evolve.

- Add sample projects for CRUD APIs, JWT authentication, and MVCActiveRecord-based applications.
- Expand testing examples with mock repositories and integration tests.
- Add deployment examples for Windows service and Linux daemon hosting.[2]
- Read the official documentation: [DMVCFramework Official Documentation](https://www.danieleteti.it/delphimvcframework_3_4_3_aluminium/).[1]
- Explore the framework source and examples: [DMVCFramework GitHub Repository](https://github.com/danieleteti/delphimvcframework).[2]
