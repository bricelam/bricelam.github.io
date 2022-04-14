---
layout: post
title: 'C# Object Notation'
tags: miscellaneous
---

JavaScript Object Notation (JSON) is a subset of the JavaScript language used for the definition and exchange of data.
While I'm not proposing that we create a similar standard using C#, I do want to illustrate some of the rich object
initialization syntax of the language. This terse syntax can come in handy when defining data for things like unit
tests, templating models, etc.

Object Initializers
===================
Consider the following class.

```cs
class Point
{
    public int X { get; set; }
    public int Y { get; set; }
}
```

Object initializers allow you to create a new object and set its properties in the same statement. Here is an example.

```cs
var x = new Point { X = 0, Y = 1 };
```

This syntax can be nested and and shortened by allocating instances inside constructors. Consider the following class.

```cs
class Rectangle
{
    public Point P1 { get; } = new Point();
    public Point P2 { get; } = new Point();
}
```

It can be initialized as follows.

```cs
var x = new Rectangle
{
    P1 = { X = 0, Y = 1 },
    P2 = { X = 2, Y = 3 }
};
```

Collection Initializers
=======================
Collection initializers let you create and add items to a collection in the same statement. The class must implement
`IEnumerable` and have a method named `Add`. C# 6 even allows the `Add` method to be an extension method. Consider the
following "class". (I know it won't actually compile.)

```cs
class IntCollection : IEnumerable
{
    public void Add(int value) { }
}
```

It can be initialized using the following syntax.

```cs
var x = new IntCollection { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 };
```

The `Add` method can have multiple parameters...

```cs
class LineString : IEnumerable
{
    public void Add(int x, int y) { }
}
```

...and be initialized like this.

```cs
var x = new LineString { { 0, 1 }, { 2 , 3 }, { 4 , 5 } };
```

The `Add` method can even be overloaded!

```cs
class StringAndIntCollection : IEnumerable
{
    public void Add(string value) { }
    public void Add(int value) { }
}
```
```cs
var x = new StringAndIntCollection { 0, "1", 2, "3" };
```

Collection initializers can also be nested and shortened inside object initializers.

```cs
class Contact
{
    public string Name { get; set; }
    public List<string> PhoneNumbers { get; } = new List<string>();
}
```
```cs
var x = new Contact
{
    Name = "Chris Smith",
    PhoneNumbers = { "206-555-0101", "425-882-8080" }
};
```

The only downfall of collection initializers is that they can't be used with object initializers on the same instance.

```cs
class NamedIntCollection : IEnumerable
{
    public string Name { get; set; }
    public void Add(int value) { }
}
```
```cs
// Like this...
var x = new NamedIntCollection { Name = "Primes" };
x.Add(2);
x.Add(3);
x.Add(5);

// ...or this...
var y = new NamedIntCollection { 2, 3, 5 };
x. Name = "Primes";

// ...but NEVER this.
var z = new NamedIntCollection { Name = "Primes", 2, 3, 5 };
```

Index Initializers
==================
Index initializers are new to C# 6. In order to use them, an indexer must be present.

```cs
class IntToStringMap
{
    public string this[int index] { get; set; }
}
```

Their syntax is pretty intuitive.

```cs
var x = new IntToStringMap
{
    [7] = "seven",
    [9] = "nine",
    [13] = "thirteen"
};
```

The indexer can have multiple parameters.

```cs
class Bitmap
{
    public bool this[int x, int y] { get; set; }
}
```
```cs
var x = new Bitmap
{
    [0, 0] = true,
    [0, 1] = false,
    [1, 0] = false,
    [1, 1] = true
};
```

It can be overloaded too.

```cs
class IntToStringAndBackMap
{
    public string this[int index] { get; set; }
    public int this[string index] { get; set; }
}
```
```cs
var x = new IntToStringAndBackMap
{
    [0] = "zero",
    ["one"] = 1
};
```

The syntax can also be shortened by instantiating objects inside the getter.

```cs
class ScatterPlot
{
    public Point this[int index]
    {
        get { return new Point(); }
        set { }
    }
}
```
```cs
var x = new ScatterPlot
{
    [0] = { X = 0, Y = 1},
    [1] = { X = 2, Y = 3}
};
```

Indexer initializers, unlike collection initializers, can be mixed with object initializers.

```cs
class NamedMap
{
    public string Name { get; set; }
    public string this[string index] { get; set; }
}
```
```cs
var x = new NamedMap
{
    Name = "Atbash Cipher",
    ["A"] = "Z",
    ["B"] = "Y",
    ["C"] = "X"
};
```

Pop Quiz
========
Alright, it's time to put everything you've learned to the test. Can you come up with a class that enables the following
syntax? :wink:

```cs
var x = new Thing
{
    A = 1,
    ["B"] = 2,
    ["C", 3] = { "D", { 4, "E" } },
    [5] = { F = 6 }
};
```

Conclusion
==========
If we were to come up with a standard, I'm not sure what we'd call it since the obvious, CSON, is already taken by
CoffeeScript. The following, however, is an example of a theoretical C# Object Notation document that could be used to
create a SQL script.

```cs
new Schema
{
    new Table
    {
        Name = "Customer",
        ["Id"] = { Type = "int", Required = true },
        ["Name"] = { Type = "varchar(30)" },
        Constraints =
        {
            new PrimaryKey { Name = "PK_Customer", Columns = { "Id" } }
        }
    },
    new Table
    {
        Name = "Order",
        ["Id"] = { Type = "int", Required = true },
        ["CustomerId"] = { Type = "int", Required = true },
        ["Total"] = { Type = "real" },
        Constraints =
        {
            new PrimaryKey { Name = "PK_Order", Columns = { "Id" } },
            new ForeignKey
            {
                Name = "FK_Order_Customer",
                Columns = { "CustomerId" },
                ReferencedTable = "Customer",
                ReferencedColumns = { "Id" }
            }
        }
    }
}
```
