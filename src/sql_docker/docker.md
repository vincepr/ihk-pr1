# Docker with mssql server
https://github.com/vincepr/CS_PostgreExample

## setting up the basic dockerfile
- docker images from https://hub.docker.com/_/microsoft-mssql-server
- adventurework db samples (Lightweight 2022.bak) from: https://learn.microsoft.com/en-us/sql/samples/adventureworks-install-configure?view=sql-server-ver16&tabs=ssms

- `docker compose up -d` with:
```yaml
version: '3.8'
services:
  sqlserver:
    image: "mcr.microsoft.com/mssql/server:2022-latest"
    environment:
      ACCEPT_EULA: "Y"
      MSSQL_SA_PASSWORD: "pa55word!"
      #MSSQL_PID: "Express"
    ports:
      - "1433:1433"
```

- or just `docker run -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=pa55word!" -p 5555:1433 -d mcr.microsoft.com/mssql/server:2022-latest -d`

## Microsoft Server Management Studio
using Server Management Studio (SSMS) to check into the db quickly
- Server type Database Engine
- we can either use the ip `127.0.0.1` or `localhost,5555` as Servername
- Authentification SQL Server Authentification with user: `sa` and pw: `pa55word`

## adding the backup data to our dockercontainer
- we run `RESTORE FILELISTONLY FROM DISK = 'C:\Users\vince\source\repos\cs_sql_tests\AdventureWorksLT2022.bak'` against a sql server. To get the Logical Names for the following MOVE commands.

- `restore-backup.sql`
```sql
RESTORE DATABASE [AdventureWorks] FROM DISK = '/tmp/AdventureWorksLT2022.bak'
WITH FILE = 1,
MOVE 'AdventureWorksLT2022_Data' TO '/var/opt/mssql/data/AdventureWorks.mdf'
MOVE 'AdventureWorksLT2022_Log' TO '/var/opt/mssql/data/AdventureWorks.ldf'
NOUNLOAD, REPLACE, STATS = 5
GO
```

- `dockerfile`      Note DOCKERFILE did not work in windows while dockerfile lowercase did
```DOCKERFILE
## build step to fill with our data from file
FROM mcr.microsoft.com/mssql/server:2022-latest AS build
ENV ACCEPT_EULA=Y
ENV MSSQL_SA_PASSWORD=pa55word!

WORKDIR /tmp
COPY AdventureWorksLT2022.bak .
COPY restore-backup.sql .

RUN /opt/mssql/bin/sqlservr --accept-eula & sleep 10 \
    && /opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P "pa55word!" -i /tmp/restore-backup.sql \
    && pkill sqlservr

## copy our created data to the persistant db
#   this removes the backup files
#   this also removes any traces of the 'pa55word!' in the bash history/ENV etc.
FROM mcr.microsoft.com/mssql/server:2022-latest AS release
ENV ACCEPT_EULA=Y

COPY --from=build /var/opt/mssql/data /var/opt/mssql/data
```
- `docker build -t restored-db .`
- `docker run -p 1433:1433 -d restored-db`