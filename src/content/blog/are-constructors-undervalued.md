---
title: 'Are Constructors Undervalued?'
description: 'My experience says yes'
pubDate: '2026-09-28'
tags: ["software", "software development", "OOP", "programming"]
---

Refactoring some code recently. I was in the middle of moving a large branching `if` statement which created varying instances of an object using a parameterless constructor into a parametered constructor of that object and I realised that this is a kind of refactoring I've done a lot of and it got me wondering, are constructors undervalued?

The purpose of a constructor is to create an instance of an object, and they can be utilised to define rules around how that object can be created. This can be a powerful tool to ensure that classes can't be created in an invalid state. Instead, I'm seeing a lot POCOs (I'm currently working in the .NET world) without a constructor defined, so an example of what this looks like is:

``` C#
public class MyClass
{
    public string Name { get; set; }
    public double? Value { get; set; }
}

public class Something
{
    public void DoSomething(OtherClass otherClass)
    {
        // Other functionality
        
        MyClass myClass = null;

        if (condition == "A")
            myClass = new() { Name = "Condition A", Value = otherClass.ValueA }
        else if (condition == "B")
            myClass = new() { Name = "Condition B", Value = otherClass.ValueB }
        else if (otherCondition == false)
            myClass = new() { Name = "Some Other Condition", Value = otherClass.Other }
        else
            myClass = new() { Name = "Not Required", Value = null }
        
        // Other functionality
    }
}
```

Depending on how widely used `MyClass` is, I find litterings of code similar to the above throughout the codebase. Not only does this violate the DRY principle, but it litters your functions with logic around how `MyClass` should be created instead of focussing on the thing it should be doing.

Moving the above logic to a constructor would look like this:

``` C#
public class MyClass
{
    public string Name { get; }
    public double? Value { get; }

    public MyClass(string condition, bool otherCondition, OtherClass otherClass)
    {
        Name = name;

        if (condition == "A")
        {
            Name = "Condition A";
            Value = otherClass.ValueA;
        }
        else if (condition == "B")
        {
            Name = "Condition B";
            Value = otherClass.ValueB;
        }
        else if (otherCondition == false)
        {
            Name = "Some Other Condition";
            Value = otherClass.Other;
        }
        else
        {
            Name = "Not Required";
            Value = null;
        }
    }
}

public class Something
{
    public void DoSomething(OtherClass otherClass)
    {
        // Other functionality
        MyClass myClass = new(condition, otherCondition, otherClass);
        // Other functionality
    }
}
```

So the first thing I want to point out is that I'm currently working on a legacy .NET Framework project so, besides `new()`, I've not used any of the newer syntactic sugar available in recent versions to simplify this further. Secondly, I'm aware that moving the logic into a constructor doesn't _feel_ as elegant as inlining it in a function. It's got three parameters and they might be specific to just the `DoSomething()` function. What if there's another function which uses `MyClass`, but has different conditions for creating an instance? Easy, new constructor! If you keep the logic for creating an instance of an object in the class itself, it lets you write less code elsewhere, meaning that it should be clearer to see what your functions do.

Is this just something I've seen on projects I've worked on, or is everyone else using constructors properly?

Anyway, I'm selling constructors...