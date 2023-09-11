# Evaluating Data Structures

# Big-O Notation
From good to bad:

|time| notation|
|---|---|
|Constant Time| O(1) |
|Logarithmic Time| O(lon(n)) |
|Linear Time| O(n) |
|Linearithmic Time| O(nlog(n)) |
|Quadratic Time| O(n²) |
|Cubic Time| O(n³) |
|Exponential Time| O(b^n)|
|Factorial Time| O(n!) |

## Properties
Big O notation only cares about the limit, when n gets really big. So 
- constants get ignored `9999 + n³ -> O(n³)`
- factors get ignored `9999*n² -> O(n²)`
- only the fastest growing factor becomes the O-Value `log(n)⁴ + 2n⁴ + 88n² -> O(n⁴)`

## Examples
### Constant time
```cs
c = a + 5*b / 12;
// since the loop always runs the same ammount of times -> constant (not coupled with n)
for (int i=0; i<99999; i++) {
    //do things here
}
```

### Linear time
```cs
var i = 0;
while (i<n) {
    i = i+1;
}

i = 0;
while (i<n) {
    i + 1000;
}
```
both blocks are `O(n)`

### Quadratic time
```cs
for (int i=0; i<n; i++) {
    for (int j=0; j<n; j++){
        // ...
    }
}
// fn(n) = n*n = n² -> O(fn(n)) = O(n²)

for (int i=0; i<n; i++) {
    for (int j=i; j<n; j++) {    //replaced  =0 with =i
        // ...
    }
}
// fn(n) = n*n = n² -> O(fn(n)) = O(n²)
```