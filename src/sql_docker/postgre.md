
# PostgreSQL with donet 

Connecting a dotnet webapi with postgresql. All started in one dockerfile.

## setup
```
dotnet add package Microsoft.EntityFrameworkCore.Tools
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL
```

## Injecting the connection string
- instead of a static one, goal is to make the connection string dynamically available from the docker container
- placerholder in our code. that the docker compose should inject the real-current connection string into.

First we add the placeholder to the appsettings.json:
```json
"AllowedHosts": "*",
  "ConnectionStrings": {
    "Defaultconnection" : ""
  }
```

Then we inject the db-context into the builder
```cs
var builder = WebApplication.CreateBuilder(args);

// inject the connection string:
var conn = builder.Configuration.GetConnectionString("DefaulCconnection");
builder.Services.AddDbContext<ApiDbContext>(options =>
    options.UseNpgsql(conn)
);
```

## Add entitiy migrations
```
dotnet ef migrations add "initial-migrations"
```

## Dockerfile

```Dockerfile
FROM mcr.microsoft.com/dotnet/sdk:7.0 AS build
WORKDIR /app
EXPOSE 80
EXPOSE 433

## copy project files over and restore dependencies etc.
COPY *.csproj ./
RUN dotnet restore

## build the binary
COPY . ./
RUN dotnet publish -c Release -o out



FROM mcr.microsoft.com/dotnet/sdk:7.0 AS final
WORKDIR /app
## copy over the binary from build container
COPY --from=build /app/out .
ENTRYPOINT ["dotnet", "CS_Postgre_Example.dll"]
```
- docker build -t postgre_api
- docker run -p 8081:80 -e ASPNETCORE_URLS=http://+:80 postgre_api

Create the `docker-compose.yaml`
```yaml
version: '3.8'

# network to connect containers (sql-db and dotnet-api)
networks:
    dev:
        driver: bridge

services:

    #the dotnet api   -   running on port 8080
    cs-api:
        # name we get when building the Dockerfile
        image: docker.io/library/postgre_api
        depends_on:
            - "postgre_db"
        container_name: cs-api-services
        ports:
            - "8080:80"
        build:
            context: .
            dockerfile: Dockerfile
        environment:
            # the connection string (we use this env to pass in connectionstrings to our api)
            ##      NOTICE 2 underscores __ here!
            - ConnectionStrings__DefaultConnection=User ID=postgres;Password=postgres;Server=postgre_db;Port=5432;Database=SampleDbDriver; IntegratedSecurity=true;Pooling=true;
            - ASPNETCORE_URLS=http://+:80
        networks:
            - dev

    # the db we use to connect to   -   running on 5433
    postgre_db:
        image: postgres:latest
        container_name: postgre_db
        environment:
            - POSTGRES_USER=postgres
            - POSTGRES_PASSWORD=postgres
            - POSTGRES_DB=SampleDbDriver
        ports:
            - "5433:5432"
        # volume for persistand data between sessions
        volumes:
            - app_data:/var/lib/postgresql/data
        networks:
            - dev

# list all our volumes and define them/name them:
volumes:
    app_data:
```

Update the `appsettings.json`. Notice we changed the `Server=` to `localhost` and the Port to the actual port used in the dockercompose: `Port=5433`
```json
"ConnectionStrings": {
"DefaultConnection" : "User ID=postgres;Password=postgres;Server=localhost;Port=5433;Database=SampleDbDriver; IntegratedSecurity=true;Pooling=true;"
}
```

### the dirty way to fill our db 
(better would be to create the db if not existing on local machine. Or maybe entityframework provides a better way)

- since the db stores fiels to the volume wa can just:
If we were to quickly want to test if the connection string is ok we could:

Now we just need to recreate our files. (while the server is up we:)
```
dotnet build
dotnet ef database update
```
- now the db should have neccessary tables created. And we can be sure the connection string is correct.
- time to delete the defaultConnection from the appsettings again. (since the container will provide the env)

## setting it up for real
compose down to remove the old containers then rebuild from scratch:
```
docker-compose down
docker-compose up --build
```
