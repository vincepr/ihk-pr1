# Dapper
Micro ORM. Used to 

# what it does
- Provides fluent interface for mapping DataReaders to objects. (instead of doing the Marshalling by hand)
- works across SQL Server, SQLite, MySQL, PostgresSQL...
- no unneccessary Memory Overhead.
## what dapper is not
- no (automatic) SQL generation from code. Like Entity Framework.
- no Database generation. Again like Entity Framework.

## some examples
```cs
public class Person{
    public string Title { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public int Age { get; set; }
}

private const string CONNECTION_STRING = "Server=.;Database=..."
[HttpGet("")]
public async Task<IActionResult> Index(){
    var sql = @"SELECT
                [Title],
                [FirstName],
                [LastName],
                [Age]
            FROM [Person].[Person]
            WHERE FirstName= 'Kevin' ";
    using (var conn = new SqlConnection(CONNECTION_STRING)){
        var persons = await conn.QueryAsync<Person>(sql);
        return Ok(persons);
    }
}
```
- using syntax and dapper take care of the .open() and .close() to the database connection.
- We pass in a the target Object-Type as generic (here Person)

### QueryAsync with parameters
if we always use the parameters we can just pass them in wrapped in a object. either a real one that will just match for the defined `@firstN`, or wrappeed in an anonymous object like below:
```cs
public async Task<IActionResult> Index(){
    var sql = @"SELECT
                [Title],
                [FirstName],
                [LastName],
                [Age]
            FROM [Person].[Person]
            WHERE FirstName = @firstN ";
    using (var conn = new SqlConnection(CONNECTION_STRING)){
        var persons = await conn.QueryAsync<Person>(sql, new{firstN = "Kevin" });
        return Ok(persons);
    }
}
```

### QueryAsync with dynamic parameters
If we were to dynamically build a sql querry, like custom search interface (sort by age, name, only with tile="Dr" ...) we can use the DynamicParameters()
```cs
[HttpGet("")]
public async Task<IActionResult> Index([FromQuery]string? getName){
    var sql = @"SELECT
                [Title],
                [FirstName],
                [LastName],
                [Age]
            FROM [Person].[Person]";
    dynParams = new DynamicParameters();
    if (getName != null){
        sql += " WHERE FirstName = @firstName";
        dynParams.Add("firstName", getName);
    }

    using (var conn = new SqlConnection(CONNECTION_STRING)){
        var persons = await conn.QueryAsync<Person>(sql, dynParams);    
        return Ok(persons);
    }
}
```
- if we pass in the 'empty' dynParams to the QueryAsyn nothing will happen and the normal sql gets used. Otherwise it will search for the '/api?getName=xzy' xzy-FirstNamed persons.

### QueryFirst or QueryFirstOrDefault
useful if we just need to chick if a value exists (at all) in the db.

### ExecuteAsync for INSERT/UPDATE/DELETE
Parameterized Queries to restrict against sql injections.
- INSERT
```cs
public async Task DemoINSERT(string CONNECTION_STRING){
    await using var conn = new SqlConnection(CONNECTION_STRING);

    var sql = @"INSERT INTO [dbo].[Person].[Person]
                ([Title],
                [FirstName],
                [LastName],
                [Age])
            VALUES
                (@title, @firstName, @lastName, @age)";
    var newUser = new User(){
        Title = "Doctor", FirstName = "Who", LastName = "The Doctor", Age = 99
    };
    var result = await conn.ExecuteAsync(sql, newUser);
}
```
- UPDATE
```cs
public async Task DemoUPDATE(string CONNECTION_STRING){
    await using var conn = new SqlConnection(CONNECTION_STRING);

    var sql = @"UPDATE [dbo].[Person].[Person]
                SET LastName = @lastName
                WHERE FirstName = @firstName";
    var result = await conn.ExecuteAsync(sql, 
        new{ firstName "Who" , lastName="Who" });
}
```
- DELETE
```cs
public async Task DemoDELETE(string CONNECTION_STRING){
    await using var conn = new SqlConnection(CONNECTION_STRING);

    var sql = @"DELETE FROM [dbo].[Person].[Person]
                WHERE FirstName = @firstName";
    var result = await conn.ExecuteAsync(sql, new{ firstName "Who" });
}
```

### StoredProcedure
Once it is more than just a handfull of Sql Querries (they also might be more optimized against) we can use StoredProcedures instead. See the [minimalApi example](https://github.com/vincepr/CS_MinimalApiApp).
```cs
public async Task DemoStoredProcedure(string CONNECTION_STRING){
    await using var conn = new SqlConnection(CONNECTION_STRING);
    var nameOfProcedure = "dbo.LastNameList";
    var result = await conn.QueryAsync<string>(sql, nameOfProcedure: CommandType.StoredProcedure);
}
```

### Transactions
great way to make sure everything finished correctly before any changes are made permanent to the db.

Or if the sql execution sends back something and only if that is correct/what we expect we commit this execution.
```cs
[HttpGet("")]
public async Task<IActionResult> DemoWithoutTransactions(){
    var sql = @"INSERT INTO [dbo].[Users]
                    ([SomeName])
                VALUES
                    (@someName)";
    using (var conn = new SqlConnection(CONNECTION_STRING)){
        await conn.OpenAsync();
        using (var transaction = conn.BeginTransaction()){
            for (var x = 0; x < 1000; x++){
                // we throw some error without the transaction the first 250 inserts would be permanent in the db => bad!
                if (x == 250) throw new Exception("Some Error happened");
                await connection.ExecuteAsync(
                    sql,
                    new {someName = $"testing{DateTime.UtcNow.Ticks}"},
                    transaction: transaction
                );
            }
            transaction.Commit();   // only if all 1000 inserts succeeded will we commit, otherwise it never gets put in the db.
        }
        return Ok();
    }
}
```