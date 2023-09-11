# MinimalAPI example project in csharp
on [github](https://github.com/vincepr/CS_MinimalApiApp)

## Infos
Goal is learning some Csharp basics, about APIs, SQL-Integration, Swagger etc. Coding along loosely (at least in the beginning) with the Youtube Project from [IAmTimCorey](https://www.youtube.com/watch?v=dwMFg6uxQ0I)


## Notes on Setting up the Project
First we create the `ASP.NET Core Web Api` then the `SQL Server Database Project` and a `class library` to define the shape of our data.

### SQL Server Database Project
- had to change to SQL Server 2019, to get it to work in Win11 (might have had to do something with disk sector size for win11 having changed the default settings, but whatever for now)
- add the Tables we want
- add `StoredProcedures` for all our incoming crud requests. (more optimized than just plain incoming sql queries)
example for the `spUser_Update.sql` (User is in squareBrackets because it is a reserved keyword)
```sql
CREATE PROCEDURE [dbo].[spUser_GetAll]
AS
BEGIN
	SELECT Id, FirstName, LastName
	FROM dbo.[User];
END
```

```sql
CREATE PROCEDURE [dbo].[spUser_Get]
	@Id int
AS
BEGIN
	SELECT Id, FirstName, LastName
	FROM dbo.[User]
	WHERE Id = @Id;
END
```

```sql
CREATE PROCEDURE [dbo].[spUser_Insert]
	@FirstNAme nvarchar(50),
	@LastName nvarchar(50)
AS
BEGIN
	INSERT INTO dbo.[User] (FirstName, LastName)
	VALUES (@FirstName, @LastName);
END
```

```sql
CREATE PROCEDURE [dbo].[spUser_Update]
	@Id int,
	@FirstName nvarchar(50),
	@LastName nvarchar(50)
AS
BEGIN
	UPDATE dbo.[USER]
	SET Firstname = @FirstName, LastName = @LastName
	WHERE Id = @Id;
END
```

```sql
CREATE PROCEDURE [dbo].[spUser_Delete]
	@Id int
AS
BEGIN
	DELETE
	FROM dbo.[User]
	WHERE Id = @Id;
END
```
- next we a `Script` to the database-project. A post-deployment-script to run after the db is up.
```sql
/*  if were empty we fill some sample data for testing */
if not exists (select 1 from dbo.[User])
begin
    insert into dbo.[User] (FirstName, LastName)
    values('Tim', 'Hames'),('Bob', 'Ross'),('Amber','Spender'),('Cameron','Griffin');
end
```
- finally we rightclick the db-project and `Publish`
    -  Edit - Browse - Local - Select the local MSSQLLocalDB
    - give the local DB a name then `Save Profile As` into the db-project folder. Now we can re-publish after making changes etc.

The DB should be ready now. We can View - SQL Server Object Explorer. Open the localdb - Databases - nameOfDb - Tables - dbo.User rightclick and showData to check if it worked.

### Class Library
We create the class library and name it DataAccess. Will define the shape of the data in our App/Api
```cs
namespace DataAccess.Models;
internal class User{
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
}
```
#### Importing some packages/libraries
rightClick Dependencies in the DataAccess class library - Manage NuGet Packages. We add
- Dapper (Micro-ORM supporting different SQL Servers)
- System.Data.SqlClient
- Microsoft.Extensions.Configuration.Abstractions

### Data access library
SqöDataAccess.cs
```cs
using Dapper;
using System.Data;
using Microsoft.Extensions.Configuration;
using System.Data.SqlClient;

namespace DataAccess.DbAccess;
public class SqlDataAccess
{
    // _ to indicate private vars coming over from Dependency Injection
    private readonly IConfiguration _config;

    public SqlDataAccess(IConfiguration config)
    {
        _config = config;
    }

    public async Task<IEnumerable<T>> LoadData<T, U>(
        string storedProcedure,             // storedProcedure name in our db
        U parameters,                       // (optional) params like the {id} for /Get?id=123
        string connectionId = "Default")    // info about what db-name/port/etc we are connecting to
                                            //  this is stored in the appsettings.json of our Api
    {
        // with the using here we garante (graceful) conn.Close() when leaving the scope
        // so we garantee 100%? were not leaving connections to the db open.
        using IDbConnection conn = new SqlConnection(_config.GetConnectionString(connectionId));
        return await conn.QueryAsync<T>(storedProcedure, parameters, commandType: CommandType.StoredProcedure);
    }

    public async Task SaveData<T>(
        string storedProcedure,
        T parameters,
        string connectionId = "Default")
    {
        using IDbConnection conn = new SqlConnection(_config.GetConnectionString(connectionId));
        await conn.ExecuteAsync(storedProcedure, parameters, commandType: CommandType.StoredProcedure);
    }
}
```
- we extract out our interfaces for easy conveniant use as dependency injection. In Visual Studio we can just select the public class `SqlDataAccess` and then press ctrl + dot then `extract interface` to autogenerate:

```cs
namespace DataAccess.DbAccess
public interface ISqlDataAccess
{
    Task<IEnumerable<T>> LoadData<T, U>(string storedProcedure, U parameters, string connectionId = "Default");
    Task SaveData<T>(string storedProcedure, T parameters, string connectionId = "Default");
}
// here we could easy switch to another db just by implementing another interface like this (for mongodb/redis for ex.)
```

- now we implement all our usecases:
```cs
namespace DataAccess.Data;
public class UserData
{
    private readonly ISqlDataAccess _db;

    public UserData(ISqlDataAccess db)
    {
        _db = db;
    }

    public Task<IEnumerable<UserModel>> GetUsers() =>
        _db.LoadData<UserModel, dynamic>("dbo.spUser_GetAll", new { });

    public async Task<UserModel?> GetUser(int id)
    {
        var res = await _db.LoadData<UserModel, dynamic>("dbo.spUser_Get", new { Id = id });
        return res.FirstOrDefault();        // default for our UserModel is null
    }

    public Task InsertUser(UserModel user) => 
        _db.SaveData("dbo.spUser_Insert", new {user.FirstName, user.LastName});

    public Task UpdateUser(UserModel user) =>
        _db.SaveData("dbo.spUser_Update", user);

    public Task DeleteUser(int id) =>
        _db.SaveData("dbo.spUser_Delete", new {Id=id});
}
```
- then we again extract this as an interface for use in dependency injection. (autogenerated with `extract interface`)
```cs
public interface IUserData
{
    Task DeleteUser(int id);
    Task<UserModel?> GetUser(int id);
    Task<IEnumerable<UserModel>> GetUsers();
    Task InsertUser(UserModel user);
    Task UpdateUser(UserModel user);
}
```

### the web API
View - SQL Server Object Explorer - Sql Server - localdbxyz - Cs_MinimalApiDB - rightClick Properties - ConnectionString. Should give something like: `Data Source=(localdb)\MSSQLLocalDB;Initial Catalog=Cs_MinimalApiDB;Integrated Security=True;Connect Timeout=30;Encrypt=False;Trust Server Certificate=False;Application Intent=ReadWrite;Multi Subnet Failover=False`

- now we add the above ConnectionString to our `appsettings.json`:
    - note here formating of `Trust Server Certificate=False` had to be changed to `TrustServerCertificate=False` and the same for `Multi Subnet Failover=False` (removed empty spaces) for it to work
```cs
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "ConnectionStrings": {
    "Default": "Data Source=(localdb)\\MSSQLLocalDB;Initial Catalog=Cs_MinimalApiDB;Integrated Security=True;Connect Timeout=30;Encrypt=False;TrustServerCertificate=False;ApplicationIntent=ReadWrite;MultiSubnetFailover=False"
  }
}
```
- rightClick `Dependencies` then `Add Project Reference` and add the previous generated class Lib DataAccess

#### Program.cs
```cs
// we define some top level imports for our whole api project here (or refactor them to own file 'GlobalUsings.cs')
global using DataAccess.Data;
global using DataAccess.Models;
// normal usings
using CS_MinimalApi;        // importing our Handlers from next step
using DataAccess.DbAccess;  // importing Sql Interfaces

//  the WebApp builder:
var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
// Learn more about configuring Swagger/OpenAPI at https://aka.ms/aspnetcore/swashbuckle
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// dependency inject our Interfaces into the app.
// This way we can use for example our UserData down in our RouteHandlers
builder.Services.AddSingleton<ISqlDataAccess, SqlDataAccess>();
builder.Services.AddSingleton<IUserData, UserData>();


var app = builder.Build();

// Enable Swagger for DevEnv only
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();

// Route our Handlers, separated to its own static Class
app.SetupApiRoutes();

app.Run();
```

####
```cs
using System.Runtime.CompilerServices;

namespace CS_MinimalApi;
/*
    All Handlers that handle any Routes like /api or /api/login etc come here.
    - midleware like loggers could be used in here aswell
    - Dapper? (as i understand at the moment) does most of the heavy lifting here, like
        - Automagically serialize and deserialize to and from json.
        - TODO: might wanna read up on that at some later point
 */

public static class Api
{
    public static void SetupApiRoutes(this WebApplication app)
    {
        // mapping the Routes/enpoints to the Api methods
        app.MapGet("/Users", Handle_GetUsers);
        app.MapGet("/Users/{id}", Handle_GetUser);      // id from the Url (because no body for Getrequest)
        app.MapPost("/Users", Handle_InsertUser);       // user in body - body gets passed down and json parsed automagically using Dapper!
        app.MapPut("/Users", Handle_UpdateUser);        // user in body
        app.MapDelete("/Users", Handle_DeletetUser);    // id in json from in body
    }

    private static async Task<IResult> Handle_GetUsers(IUserData data)
    {
        try
        {
            return Results.Ok(await data.GetUsers());
        }catch (Exception ex)
        {
            return Results.Problem(ex.Message); // just throw the error back to client
        }
    }

    private static async Task<IResult> Handle_GetUser(int id, IUserData data)
    // we get the id from the Url
    // and the data from the singleton we injected into app in Programm.Main
    {
        try
        {
            var res = await data.GetUser(id);
            if (res == null) return Results.NotFound();
            return Results.Ok(res);
        }
        catch (Exception ex)
        {
            return Results.Problem(ex.Message);
        }
    }

    private static async Task<IResult> Handle_InsertUser(UserModel user, IUserData data)
    // we get the UserModel from the Body of the http-request (the data again from the singleton)
    {
        try
        {
            await data.InsertUser(user);
            return Results.Ok();
        }
        catch (Exception ex)
        {
            return Results.Problem(ex.Message);
        }
    }

    private static async Task<IResult> Handle_UpdateUser(UserModel user, IUserData data)
    {
        try
        {
            await data.UpdateUser(user);
            return Results.Ok();
        }
        catch (Exception ex)
        {
            return Results.Problem(ex.Message);
        }
    }

    private static async Task<IResult> Handle_DeletetUser(int userId, IUserData data)
    {
        try
        {
            await data.DeleteUser(userId);
            return Results.Ok();
        }
        catch (Exception ex)
        {
            return Results.Problem(ex.Message);
        }
    }
}
```