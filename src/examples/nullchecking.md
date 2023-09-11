# Nullchecking in csharp

```cs
// the incorrect way:
if (user == null)  {}

// the better way:
if (user is null) {}
if (user is not null) {}

// check if value is null then also if Name is null
if (value?.Name is null) {}

// if value is null some default value gets assigned
var test = value ?? new SomeDefaultClass("Mr Smith");

var firstName = value ?? throw new ArgumentNullException("Name cant't be null");
```

- refactoring a nullcheck:
```cs
public override bool Equals(object? obj){
    if (obj is Point) {
        var other = (Point)obj;
        return this == other;
    } else {
        return false;
    }
}

// we can refactor this. (asssigning after the is Point)
public override bool Equals(object? obj){
    if (obj is Point other) {
        return this == other;
    } else {
        return false;
    } 
}

// even shorter using ternary operator

public override bool Equals(object? obj){
    (obj is Point other)
    ? this == other 
    : false
}
```