# Deconstruction in Csharp
Short syntax to get values out of Touples as expected.
- `Deconstruct` named function to enable custom deconstructing syntax.
- Records implement Deconstructing by default.
- 

```cs
var james = new Person{
    Name = "James",
    Surname = "Bond",
    DateOfBirth = new DateOnly(1959, 2, 22),
};

var (name, _, birthday) = james;


public class Person{
    public string Name { get; init; } = default!;
    public DateOnly DateOfBirth { get; init; }

    // Custom deconstructor for our Person class
    public void Deconstruct(out string name, out string surname, out DateOnly birthday)
    {
        name = Name;
        surname = Surname;
        birthday = DateOfBirth;
    }
}
```

It is even possible to write extension Method Deconstruct for Dotnet Core Implementations like DateOnly:

```cs
var date = new DateOnly(1959, 2, 22);
var (y, m, d) = date;

var (year, month, date) = new DateOnly(1999, 9, 9);


public static class DeconstructionExtensions{
    // Deconstructs externally on this DateOnly instance.
    public static void Deconstruct(this DateOnly dateonly, 
        out int year, out int month, out int day)
    {
        year = dateonly.Year;
        month = dateonly.Month;
        day = dateonly.Day;
    }
}
```