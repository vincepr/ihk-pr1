# ref, in, out keyword in Csharp
Basically like `ref` keyword. Instead of copying like normally for stack-values a reference gets passed.

## ref-keyword
- ref doesnt enforce any constraints on what to do with `x` in the below example
```cs
int value = 99;
Example(ref value);
Console.WriteLine(value);   // -> 11

void Example(ref int x){
    x = 11;  // mutates dereferenced original 'value'
}
```

## out-keyword
```cs
var couldParse = int.TryParse("123", out int parsedValue);
Console.WriteLine(parsedValue); // -> 123
var value = 100;
var couldParse = int.TryParse("asdf", out value);   // original value gets mutated
Console.WriteLine(parsedValue); // -> 0 (the default init value)
```

- out enforces us to modify/set `x = 11;` or it wont compile
- every branch has to set the out value on Exit!
```cs
int value = 99;
Example(ref value);
Console.WriteLine(value);   // -> 11

void Example(out int x){
    x = 11;  // this mutation is enfored with the out keyword
}
```

## in-keyword
Usecase: Passing down valuetypes (especially big structs) as function parameters will copy each time.

this causes a huge memory allocation burden.

One could use the ref keyword BUT now the function could mutate the original etc.
- we can avoid this by using `in`.
- this effectively makes it a ref but **readonly**!
```cs
int value = 99;
Example(in value);
void Example(in int x){
    // value = 123;         <- NOT ALLOWED
    Console.WriteLine(x);   // is allowed
}
```

### When to use
- if the struct is bigger than pointer size
- if using `in` with a struct make sure the struct is **readonly** or defensive copies will degrade performance benefits.