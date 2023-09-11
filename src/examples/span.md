# Span and Benchmarking in Csharp

## Span instead of String in Csharp
By using Span (in combination with fitting methods like Slice()) we can remove expensive Allocation on the heap alltogether.

```cs
namespace span;

class Program
{
    private static readonly string _dateAsText = "20.07.2022";
    static void Main(string[] args)
    {
        Console.WriteLine(DateFromString());
        Console.WriteLine(DateFromSpan());
    }

    public static (int day, int month, int year) DateFromString(){
        var dayTxt = _dateAsText.Substring(0,2);    // here we allocate "20",
        var monthTxt = _dateAsText.Substring(3,2);  // here "07",
        var yearTxt = _dateAsText.Substring(6);     // and here "2022" on the heap.
        var day = int.Parse(dayTxt);
        var month = int.Parse(monthTxt);
        var year = int.Parse(yearTxt);
        return (day, month, year);                  // they get cleaned when leaving scope.
    }

    public static (int day, int month, int year) DateFromSpan(){
        ReadOnlySpan<char> dateAsSpan = _dateAsText;
        var dayTxt = dateAsSpan.Slice(0,2);
        var monthTxt = dateAsSpan.Slice(3,2);
        var yearTxt = dateAsSpan.Slice(6);
        var day = int.Parse(dayTxt);
        var month = int.Parse(monthTxt);
        var year = int.Parse(yearTxt);
        return (day, month, year);
    }
}
```
- Span will only need to store a pointer and some data about the length of the slice. this can all stay on the Stack. So it will always outperform heap allocations by a ton.

# Benchmarking Class in Csharp
1. Add the BenchmarkDotNet Nu Get package. `dotnet add package BenchmarkDotNet`
2. create a class having one more more methods decorated with the `Benchmark` attribute.
3. Run your benchmark project in `Release mode` using the Run method of the `BenchmarkRunner`

Example of Benchmarking the above code:
- run it with: `dotnet run -c Release`

```cs
using BenchmarkDotNet.Attributes;
using BenchmarkDotNet.Running;
namespace span;

class Program
{
    static void Main(string[] args)
    {
        BenchmarkRunner.Run<BenchmarkDemo>();
    }
}

[MemoryDiagnoser]
public class BenchmarkDemo{
    private static readonly string _dateAsText = "20.07.2022";
    
    [Benchmark]
    public (int day, int month, int year) DateFromString(){
        var dayTxt = _dateAsText.Substring(0,2);
        var monthTxt = _dateAsText.Substring(3,2);
        var yearTxt = _dateAsText.Substring(6);
        var day = int.Parse(dayTxt);
        var month = int.Parse(monthTxt);
        var year = int.Parse(yearTxt);
        return (day, month, year);
    }

    [Benchmark]
    public (int day, int month, int year) DateFromSpan(){
        ReadOnlySpan<char> dateAsSpan = _dateAsText;
        var dayTxt = dateAsSpan.Slice(0,2);
        var monthTxt = dateAsSpan.Slice(3,2);
        var yearTxt = dateAsSpan.Slice(6);
        var day = int.Parse(dayTxt);
        var month = int.Parse(monthTxt);
        var year = int.Parse(yearTxt);
        return (day, month, year);
    }
}
```

Results:
```
|         Method |     Mean |    Error |   StdDev |   Gen0 | Allocated |
|--------------- |---------:|---------:|---------:|-------:|----------:|
| DateFromString | 48.34 ns | 0.070 ns | 0.062 ns | 0.0459 |      96 B |
|   DateFromSpan | 34.75 ns | 0.254 ns | 0.238 ns |      - |         - |
```