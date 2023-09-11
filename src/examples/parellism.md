# Parallel For foreach and Task.WhenAll in Csharp

```cs

using BenchmarkDotNet.Attributes;

namespace tutorials.ApiParallelTest;

public class ApiParallelTest{

    private static readonly HttpClient HttpClient = new();
    private const int TaskCount = 1000;

    [Benchmark]
    public async Task<List<int>> ForEachApi(){
        List<int> list = new();
        var fakeTasks = Enumerable.Range(0, TaskCount)
            .Select(_ => new Func<Task<int>>(() => SomeAsyncRequest(HttpClient))).ToList();
        
        foreach(var task in fakeTasks){
            // since the Tasks we created above are wrapped in a Function they didnt fire
            // we now evoke them and await that:
            list.Add(await task());
        }
        return list;
    }

    // this will load all available cores to nearly 100% load
    // - so wasting some cycles blocking a thread while awaiting!
    [Benchmark]
    public async Task<List<int>> UnlimitedParallelForeachApi(){
        List<int> list = new();
        var fakeTasks = Enumerable.Range(0, TaskCount)
            .Select(_ => 
                new Func<int>(() => SomeAsyncRequest(HttpClient).GetAwaiter().GetResult())).ToList();
        

        Parallel.For(0, TaskCount, i => list.Add(fakeTasks[i]()));
        return list;
    }

    // here we can provide a variable degree of parallelism (ex. according to load)
    [Benchmark]
    public async Task<List<int>> MaxParallelForeachApi(int maxDegreeParallelism){
        List<int> list = new();
        var fakeTasks = Enumerable.Range(0, TaskCount)
            .Select(_ => 
                new Func<int>(() => SomeAsyncRequest(HttpClient).GetAwaiter().GetResult())).ToList();
        

        Parallel.For(0, TaskCount, new ParallelOptions{
            MaxDegreeOfParallelism = maxDegreeParallelism,
        }, i => list.Add(fakeTasks[i]()));
        return list;
    }

    // Task.WhenAll does NOT block and await but instead asynchrony instead.
    // - this doesnt overutilize the cores that much -
    // - while still beeing much faster than ForeachApi
    // - this will perform more consistently and efficient
    // -> and overall scale the best
    [Benchmark]
    public async Task<List<int>> WhenAllApi(){
        var fakeTasks = Enumerable.Range(0, TaskCount)
            .Select(_ => SomeAsyncRequest(HttpClient));
        var result = await Task.WhenAll(fakeTasks);
        return result.ToList();
    }

    // CUSTOM combined parallel AND await to get the fasts troughput possible
    [Benchmark]
    public async Task<List<int>> CustomAsyncParallelv1() => await ParallelAndAsyncApi(1);
    [Benchmark]
    public async Task<List<int>> CustomAsyncParallelv2() => await ParallelAndAsyncApi(10);
    [Benchmark]
    public async Task<List<int>> CustomAsyncParallelv3() => await ParallelAndAsyncApi(100);

    public async Task<List<int>> ParallelAndAsyncApi(int batches){
        List<int> list = new();
        var fakeTasks = Enumerable.Range(0, TaskCount)
            .Select(_ => new Func<Task<int>>(() => SomeAsyncRequest(HttpClient))).ToList();
        
        await CustomParallelAndAsyncForeach(fakeTasks, batches, async func =>{
            list.Add(await func());
        });
        return list;
    }

    /// Custom Task that executes both in parallel AND uses Await at the same time
    public static Task CustomParallelAndAsyncForeach<T>(
            IEnumerable<T> source,
            int degreeOfParallelization,
            Func<T, Task> body)
    {
        async Task AwaitPartition(IEnumerator<T> partition)
        {
            using(partition){
                while(partition.MoveNext()){
                    await body(partition.Current);
                }
            }
        }

        return Task.WhenAll(
            System.Collections.Concurrent.Partitioner
                .Create(source)
                .GetPartitions(degreeOfParallelization)
                .AsParallel()
                .Select(AwaitPartition)
        );
    }

    // this is the request we make a bunch of times, doing some Json parsing as work.
    private static async Task<int> SomeAsyncRequest(HttpClient httpClient){
        var response = await httpClient.GetStringAsync($"");
        var user = JsonSerializer.Deserialize<SomeUserData>(response);
        return user!.Data;
    }
}
```