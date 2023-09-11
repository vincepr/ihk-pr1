# Pattern Matching in Csharp

Just some Examples to show the syntax

```cs
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

namespace tutorials.PatternMatching;

public static class Example{
    public static void Run(){
        var circle = new Circle(5);
        var rect = new Rectangle(420, 123);
        var square = new Rectangle(55, 55);

        var shapes = new List<Shape> {circle, rect, square};

        var rngShape = shapes[new Random().Next(shapes.Count)];

        // the legacy way to cast
        if (rngShape is Circle){
            var cir = (Circle)rngShape;      // hard cast (can trhow)
            var cir2 = rngShape as Circle;   // safe cast
            Console.WriteLine($"Circle with Diameter: {cir.Diameter}");
        }
        // or precast in the if statement (let if ish)
        if (rngShape is Rectangle re) 
            Console.WriteLine(re.Height + re.Width);
        
        switch (rngShape){
            case Circle c:
                Console.WriteLine("This is a cicle with radius = " + c.Radius);
                break;
            // we can even use the when keyword that already uses the casted r.
            case Rectangle r when r.Height == r.Width:
                Console.WriteLine("This is a square with A = " + r.Width);
                break;
        }

        /*      Csharp 8 and ondward    */

        // pattern matching in if statement
        if (rngShape is Circle {Radius : 10, Diameter : 20}) 
            Console.WriteLine("");

        // switch expression:
        var shapeDetails = rngShape switch {
            Circle c => "This is a circle"+ c.Area,
            Rectangle sq when sq.Height == sq.Width => "This is a square",
            {Area: 100} => "This area was 100",
            _ => "Not implemented Shape."
        };

        /*      Csharp 9 and ondward    */
        if (rngShape is not Circle {Area: < 100 and >=20}){ }

        shapeDetails = rngShape switch {
            Circle {Area: >100 and < 200} => "Perfect circle",
            _ => "Not implemented Shape."
        };

        shapeDetails = rngShape.Area switch{
            >= 123 and < 1000 => "matching against the double itself",
            _ => null,
        };

        // Syntax for nested Objects:
        if (rngShape is ShapeWithShape {Nested: {Area: > 100}}) 
            Console.WriteLine("");

        /*      Csharp 9 and ondward    */ 
        // simplyfing access in nested situations:
        if (rngShape is ShapeWithShape {Nested.Area: > 100}) 
            Console.WriteLine("");
    }

    
    // Example functions using Pattern Matching:
    public static bool IsAlphaNumeric(this char c) =>
        c is >= 'a' and <= 'z' or >= 'A' and <= 'Z';
    
    // Works on Touples just fine:
    public static decimal GetGroupPriceDiscout(int groupSize, DateTime date)
        => (groupSize, date.DayOfWeek) switch
    {
        (<= 0, _) => throw new ArgumentException("Error: groupSize must be positive."),
        (_, DayOfWeek.Saturday or DayOfWeek.Sunday) => 0.0m,
        (>=5, not DayOfWeek.Monday) => 1.0m,
        (>=10 and <20,_) => 2.0m,
        (>20,_) => 3.0m,
        _ => 0.0m,
    };
}

public abstract class Shape{
    public abstract double Area { get; }
}
public class Rectangle : Shape, ISquare{
    public double Height {get; set;}
    public double Width {get; set;}
    public Rectangle(double h, double w){
        Height = h;
        Width = w;
    }
    public override double Area => Height * Width;
}
public class Circle : Shape{
    private const double PI = Math.PI;
    public double Diameter {get; set;}
    public double Radius => Diameter/2;
    public Circle(double d){
        Diameter = d;
    }
    public override double Area => Diameter * PI;
}
public interface ISquare{
    double Height {get; set;}
    double Width {get; set;}
}
// just to show the syntax for nested objects:
public class ShapeWithShape : Shape{
    public override double Area => 123;
    public Shape Nested {get; set;}
    public ShapeWithShape(){
        this.Nested = new Rectangle(1,2);
    }

}
```