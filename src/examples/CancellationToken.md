# CancellationToken in Csharp
To stop wasting ressources on for example disconnected Http-Requests, Websockets etc. a often used strategy is to use CancellationTokens. (like golang's ctx Context)
- this can stop the running process (ex a Sql query taking time with more following after) and free those ressources.

The token itself has **no** `token.Cancel()` functionality. 

But instead the Consumer (ex the sql-library or entity-framework) has to check that token and then itself Stop the process/cleanup etc.

## Cancellation Token in the WebApi
Cancellation Tokens are well supported by most frameworks, the web-controllers etc.

- Controller without Cancellation
```cs
public class Examplecontroller : ControllerBase{
    private readonly IExampleRepository _exampleRepository;

    public ExampleController(IExampleRepository exampleRepository){
        _exampleRepository = exampleRepository;
    }

    [HttpGet("")]
    public async Task<IActionResult> SomeUnderPerformingQuery(){
        var result = await _exampleRepository.GetSomeNumersAsync();
        return Ok(result);
    }
}
```

- Controller passing down a Cancellation Token for the iExampleRepository to use
```cs
public class Examplecontroller : ControllerBase{
    private readonly IExampleRepository _exampleRepository;

    public ExampleController(IExampleRepository exampleRepository){
        _exampleRepository = exampleRepository;
    }

    [HttpGet("")]
    public async Task<IActionResult> SomeUnderPerformingQuery(CancellationToken token){
        var result = await _exampleRepository.GetSomeNumersAsync(token);
        return Ok(result);
    }
}
```

- Our IExampleRepository (passing down the Cancellation Token)
```cs
public interface IExampleRepository{
    Task<int> GetSomeNumersAsync(CancellationToken token);
}

public class ExampleRepository : IExampleRepository
{
    private readonly IbBConnectionFactory _dbConnectionFactory;
    private readonly ILogger<ExampleRepository> _logger;

    public ExampleRepository(IDbConnectionFactory dbFac, ILogger<ExampleRepository> logger){
        _dbConnectionFactory = dbFac;
        _logger = logger;
    }

    public async Task<int> GetSomeNumbersAsync(CancellationToken token){
        using var con = await _dbConnectionFactory.CreateconnectionAsync();
        _logger.LogInformation("This is designed to take forever");

        var cmdDef = new CommandDefinition(@"WITH RECURSIVE r(i) AS (VALUES(0) UNION SELECT i FROM r LIMIT 3000 ...)", cancellationToken: token);

        try
        {
            var firstResult = await connection.QueryFirstOrDefaultAsync<int>(cmdDef);
            _logger.LogInformation("First result done");

            // now with the token the second Query here should never run (if we cancel the request in the time the first takes)
            var secondResult = await connection.QueryFirstOrDefaultAsync<int>(cmdDef);
            _logger.LogInformation("Second result done");
        }
        catch(TaskCancelledException err)
        {
            ConsoleWriteLine("Operation was cancelled. Freeing resources.");
        }
        return firstResult + secondResult;
    }
}
```

Implementation differ on how those Cancellations get handled.
`TaskCanceledException` or `OperationCanceledException` Throws are common.

In the above example we are wrapping the firstResult and secondResult Query in a try-catch. 

Another way would be some Middleware to at one single point of contact handle those and for example log them.


 
## Cancellation Token in a Console App
Example to show how to handle the Token yourself.

- The token itself cant be cancelled.
- Instead it is a byproduct of a `new CancellationTokenSource()`
- this Source provides a Token, and this source can then cancell tokens.

### How it can cancel
- the source can manually Cancel`: `cancellationTokenSource.Cancel()`
- the source can manually Cancel`: `cancellationTokenSource.CancelAfter(someTimeStamp)`
    - this way we could cancel Requests that take longer than 2 seconds etc.
- you can also set it in the TokenSource Constructor, to trigger a timeout-like cancellation.

```cs
var cancellationTokenSource = new CancellationTokenSource();
var token = cancellationTokenSource.Token;      // gets the Token to pass down
cancellationTokenSource.CancelAfter(3000);      // cancels after 3 seconds

// keep doing things while Token is active:
while (!token.IsCancellationRequested){
    Console.WriteLine("working");
}

// another strategy is to throw on cancellation (works with token and tokenSource)
token.ThrowIfCancellationRequested();   // -> OperationCancelledException
cancellationTokenSource.Token.ThrowIfCancellationRequested()
```

#### Example running a loop
```cs
public class Program{
    static async Task Main(string[] args){
        var cancellationTokenSource = new CancellationTokenSource();
        await ExampleWithLoop(cancellationTokenSource);
    }

    public static async Task ExampleWithLoop(CancellationTokenSource cancellationTokenSource){
        // this task runs on its own and waits for C to be pressed -> cancels
        Task.Run(() =>
        {
            var key = Console.ReadKey();
            if (key.Key == ConsoleKey.C){
                cancellationTokenSource.Cancel();
                Console.WriteLine("Cancelling the task after C was pressed");
            }
        });

        // so this can run synchronous and will keep running all 3 seconds.
        while (!cancellationTokenSource.Token.IsCancellationRequested){
            Console.WriteLine("Doing some work for 3 seconds.");
            await Task.Delay(3000);
        }
        Console.WriteLine("We exited the loop");
        
        // it is important to dispose of the TokenSource after it is done:
        cancellationTokenSource.Dispose();
    }
}
```