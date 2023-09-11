# Records in C#
- allocated in the Heap (like classes/objects etc.)
- bunch of quality of life over class/struct
```cs
using System.Text;
namespace tutorials.RecordType;
public class Example{
    public static void Run(){
        var rob = new Person("Rob Rega", new DateOnly(1940,1,2));
        var rob2 = new Person("Rob Rega", new DateOnly(1940,1,2));

        var clara = new PersonAsClass{
            FullName = "Clara Carleson",
            DateOfBirth = new DateOnly(1940,1,2),
        };var clara2 = new PersonAsClass{
            FullName = "Clara Carleson",
            DateOfBirth = new DateOnly(1940,1,2),
        };

        // good ToString() by default:
        Console.WriteLine(rob);     // Person { FullName = Rob Rega, DateOfBirth = 02/01/1940 }
        Console.WriteLine(clara);   // tutorials.RecordType.PersonAsClass

        // good '==' / Equality checking by Properties NOT Reference.
        Console.WriteLine(rob == rob2);     // True
        Console.WriteLine(clara == clara2); // False

        // easy to deconstruct the Properties:
        var (name, dateOfBirth) = rob;

        // Properties are immutable {like init set;}
        // - we can use 'with' clone data into NEW records.
        var older_rob = rob with { DateOfBirth = rob.DateOfBirth.AddYears(1)};
        Console.WriteLine(older_rob);
        // -> Person { FullName = Rob Rega, DateOfBirth = 02/01/1941 }

        // and here we override the default Print and 'with' behavior:
        var arob = new APerson("Rob Rega", new DateOnly(1940,1,2));
        var older_arob = arob with {DateOfBirth = rob.DateOfBirth.AddYears(2)};
        Console.WriteLine(older_arob);
        // -> APerson { FullName:old-man-Rob RegaDateOfBirth:02/01/1942 }
    }
}

  
// This is the Version with all the boilerplate
public record PersonAsRecord{
    public string FullName { get; init; } = default!;
    public DateOnly DateOfBirth { get; init; }= default!;
}
// functionally the same as the above (but with less boilerplate)
public record Person(string FullName, DateOnly DateOfBirth);

// we have to make it a struct explicitly (is a class by default)
public record struct PersonAsStruct(string FullName, DateOnly DateOfBirth);

// normal class for comparison
public class PersonAsClass{
    public string FullName { get; init; } = default!;
    public DateOnly DateOfBirth { get; init; }= default!;
}


// the copying mechanism we can override:
public record APerson(string FullName, DateOnly DateOfBirth){
    // We can override the default 'with'-behavior:
    protected APerson(APerson oldPerson){
        FullName = "old-man-" + oldPerson.FullName;
        DateOfBirth = oldPerson.DateOfBirth.AddYears(1);
    }

    // we could also override the Method used for the default Printing:
    protected virtual bool PrintMembers(StringBuilder builder){
        builder.Append($"FullName:{FullName}");
        builder.Append($"DateOfBirth:{DateOfBirth}");
        return true;
    }

    // and here the compiler generated ToString() that we could also override:
    public override string ToString(){
        StringBuilder stringBuilder = new StringBuilder();
        stringBuilder.Append("APerson");
        stringBuilder.Append(" { ");
        if (PrintMembers(stringBuilder))
            stringBuilder.Append(" ");
        stringBuilder.Append("}");
        return stringBuilder.ToString();
    }
}
```
Here some more of the basic auto generated implementation:
- Basically its just a class with some basic quality-of-life ontop
- To check what the compiler lowers code down to: https://sharplab.io/
```cs
// we could also override the Equality checking
protected virtual Type EqualityContract {
    get{
        return typeof(APerson);  // this is the one used by default
    } 
}

public static override bool operator !=(APerson left, APerson right){
    return !(left == right);
}

public static override bool operator ==(APerson left, APerson right){
    if ((object)left != right){
        if ((object)left != null)
            return left.Equals(right);
        return false;
    }
    return true;
}
```