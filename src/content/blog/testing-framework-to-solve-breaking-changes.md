---
title: 'How I Stopped Dependency Changes From Massively Breaking The Unit Test Build'
description: "I don't really recommend doing this"
pubDate: '2026-10-23'
tags: ["software", "testing", "unit testing", "C#", "NUnit"]
---

## The Scenario

I had a codebase that had a few large classes in the service layer, each with a lot of dependencies. When I started on this project there were a lot of changes and small refactors going on in this codebase, which meant that the class dependencies were changing a lot, causing the unit test project build to fail due to constructor of the class being instantiated changing. This meant that adding a new dependency to a class potentially meant going through hundreds of tests and updating the changed constructor, so this solution is aimed at fixing this specific problem.

## My Solution

To start with, I should mention that this was implemented in C# with NUnit so the code examples will reflect that. There may be other stack specific nuances which I'll try to point out. The examples use NSubstitute for the mocks, but Moq also works fine. I can think of one reason why other mocking libraries may not work exactly like I've defined which I'll explain later on.

Lets start with the file structure of the unit test project:

```ansi
| - UnitTestProject
| -- Class1
| --- MyFunctionTests.cs
| --- MyFunction2Tests.cs
| --- ...
| -- Class2
| --- MyFunction3Tests.cs
| --- ...
```

This was something I'd already put in place before putting this solution in place. With the size of the classes, it helped keep things a bit more organised by not keeping everything in a monolithic test class for each business logic class being tested. A small benefit of this is that the conventional unit testing name structure of `ClassName_FunctionName_Scenario()` could be ignored in favour of `Scenario()` since the class name and function name are embedded in the file structure and namespace. You'll see how the tests being arranged like this plays into the solution later on.

So step 1 is to move any common set up steps into the "StartUp" section of the test. This is the NUnit terminology for a piece of code which runs before each individual unit test. In NUnit world, "Teardown" is the terminology for code which runs after every unit test.

So we go from:

```csharp
[Test]
public async Task MyFunctionIsSuccessIf5()
{
    // Arrange
    var dep1 = Substitute.For<Dependency1>();
    var dep2 = Substitute.For<Dependency2>();
    var myClass = new MyClass(dep1, dep2);

    // Act
    var result = await myclass.MyFunction(5);

    // Assert
    Assert.That(result.Success, Is.True);
}

[Test]
public async Task MyFunctionIsFailIf10()
{
    // Arrange
    var dep1 = Substitute.For<Dependency1>();
    var dep2 = Substitute.For<Dependency2>();
    var myClass = new MyClass(dep1, dep2);

    // Act
    var result = await myclass.MyFunction(10);

    // Assert
    Assert.That(result.Success, Is.False);
}

```

To:

```csharp
private MyClass _myClass;

[StartUp]
public void StartUp()
{
    var dep1 = Substitute.For<Dependency1>();
    var dep2 = Substitute.For<Dependency2>();
    _myClass = new MyClass(dep1, dep2);
}

[Test]
public async Task MyFunctionIsSuccessIf5()
{
    // Arrange
    // N/A

    // Act
    var result = await _myclass.MyFunction(5);

    // Assert
    Assert.That(result.Success, Is.True);
}

[Test]
public async Task MyFunctionIsFailIf10()
{
    // Arrange
    // N/A

    // Act
    var result = await _myclass.MyFunction(10);

    // Assert
    Assert.That(result.Success, Is.False);
}
```

So that's a good start. Any changes to `MyClass` dependencies now need to be updated once _per class_. In my scenario that definitely helped, but it still meant that I needed to update a bunch of files just to get the tests to build, so I took it a step further.

Introducing the `ClassTestBase`! The resulting code looks like this:

```csharp
public class MyClassTestBase
{
    public MyClass myClass;

    [SetUp]
    public void SetUp()
    {
        var dep1 = new Dependency1();
        var dep2 = new Dependency1();
        myClass = new MyClass(dep1, dep2);
    }
}

public class MyFunctionTests : MyClassTestBase
{
    [Test]
    public async Task MyFunctionIsSuccessIf5()
    {
        // Arrange
        // N/A

        // Act
        var result = await myclass.MyFunction(5);

        // Assert
        Assert.That(result.Success, Is.True);
    }

    [Test]
    public async Task MyFunctionIsFailIf10()
    {
        // Arrange
        // N/A

        // Act
        var result = await myclass.MyFunction(10);

        // Assert
        Assert.That(result.Success, Is.False);
    }
}
```

As long as I inherit the `MyClassTestBase` on all classes which are testing functions in the `MyClass` class then I just have to change the dependencies in a single place. Problem solved!

Kind of. Real life is messy like that.

### Taking It A Step Further

So I've solved the problem for dependencies, but what about mocks? Since we've got a lot of dependencies, a lot of the time a mock is doing the same thing (if anything at all) in each test. The simple solution would be to add it to the `StartUp` function, but we've just made that a lot tricker by lifting it up to be more generic for tests across a range of different functions. The mocks will all have different requirements for each function being tested.

Just to be clear what my end goal is here; I have say 5 unit tests for a particular function. In 4 of these tests I want a call to the mocked function, `IsDataValid()`, to return `true`, and in one scenario I want it to return `false`. The function returning `true` is part of the happy path route for the test so I want to configure it once, as default behavior, so that all calls to it return `true`. Since I have a scenario where I need the function to return `false`, I need the ability to overwrite the default behaviour for this specific test case.

This change is going to get messy so I also want to be able to split things up a bit more by configuring the data being returned from the mocks in a separate function to where the mocks are configured. This part is personal preference but with what I was dealing with at the time, it definitely helped keep things more organised.u

So here's what we need; some code to run before the unit test to spin up the class containing the function we're testing, a function to run before the unit test to set up any data we need for the mocks for our defined happy path, and a function to run before the unit test to set up our mocked dependencies. We want this to run in this order:

1. Create dependencies & class under test
2. Create mocked response data
3. Configure mocks to return response data

So now I've created a new problem for myself. In NUnit, you can't _really_ have more than one `[SetUp]` function. If you do, you can't guarantee the order in which the functions will execute. Since step 3 is dependent on both step 1 and 2 going first, I'm going to need to solve this.

Enter our abstract `TestBase`:

```csharp
public abstract class TestBase
{
    public abstract void SetUp_CreateClasses();
    public abstract void SetUp_Data();
    public abstract void SetUp_ConfigureMocks();

    [SetUp]
    public void Setup()
    {
        SetUp_CreateClasses();
        SetUp_Data();
        SetUp_ConfigureMocks();
    }
}

public abstract class MyClassTestBase : TestBase
{
    public Dependency1 dep1;
    public Dependency2 dep2;
    public MyClass myClass;

    public override void SetUp_CreateClasses()
    {
        dep1 = new Dependency1();
        dep2 = new Dependency1();
        myClass = new MyClass(dep1, dep2);
    }
}

public class MyFunctionTests : MyClassTestBase
{
    private Data _data;

    public override void SetUp_Data()
    {
        _data = new Data
        {
            Name = "Josh Hughes",
            Website = "www.joshh.co.uk"
            ...
        };
    }

    public override void SetUp_ConfigureMocks()
    {
        dep1.GetData(Arg.IsAny<int>()).Returns(_data);
        dep2.IsValidData().Returns(true);
    }

    [Test]
    public async Task MyFunctionIsSuccessIf5()
    {
        // Arrange
        // N/A

        // Act
        var result = await myclass.MyFunction(5);

        // Assert
        Assert.That(result.Success, Is.True);
    }

    [Test]
    public async Task MyFunctionIsFailIf10()
    {
        // Arrange
        dep2.IsValidData().Returns(false);

        // Act
        var result = await myclass.MyFunction(10);

        // Assert
        Assert.That(result.Success, Is.False);
    }
}
```

We've made it, this is the final result! A fair bit has changed so I'll break it down. I've added a bit more detail to the tests to help explain these changes as well.

Firstly, there's the abstract `TestBase` class. This serves as our test entry point. The single `Setup()` function contains the only `[SetUp]` attribute in this process and defines the order we want things to run in. We want to create the classes for the dependencies and the class we're testing, followed by setting up the data we need for the test, finishing with configuring the mocks needed for the test. This order is now set in stone. We're relying on hopes and dreams for multiple setups to run in the right order. These functions are defined as abstract because we wil add the implementations further down the pipeline.

Next change is the `MyClassTestBase` class which now inherits `TestBase`. I've replaced the existing `SetUp()` function with an override of `SetUp_CreateClasses()`. Nothing much else has changed here. I have lifted the dependencies to the class scope. This is required since we'll need to access them in child classes to configure the mocks.

Finally we have `MyFunctionTests`. Since this is already inheriting `MyClassTestBase` there's not much to do other than satisfy the compiler warnings you're getting by implementing an override of `SetUp_Data()` and `SetUp_ConfigureMocks()`. As mentioned, I've fleshed out the tests a bit more for this example so it's clearer that `SetUp_Data()` is a function for instantiating all of the data needed for `SetUp_ConfigureMocks()` to use to configure the mocks. I strongly suggest only configuring data and mocks for your happy path, I'll explain why in the next section. 

The final change to mention is that in `SetUp_ConfigureMocks()` I'm setting `IsValidData()` to return `true` for every call. In the test `MyFunctionIsFailIf10()` I'm setting it to return `false` for every call. Since This solution was originally written using Moq as a mocking library before being ported to NSubstitute, I can confirm that this behaviour works for both of these libraries because the configuration in the arrange section of the test overwrites the behaviour from the setup. There may be some mocking solutions which don't behave in this way which will need a different approach.

## Lessons Learned

So this was a great solution, but only for a bit. We went from updating every single test, to just updating a class per function being tested, to updating a class per class being tested, drastically reducing the number of changes needed in tests, so you could get them built and run much quicker. The problem is that this solves the very specific problem of large classes containing large functions where class dependencies changes frequently. Over time as the code base matured, the frequency of changes to dependencies lessened so the advantages of the additional complexity of this framework weren't as obvious.

So lets talk about the complexity itself. It might not be immediately obvious what this is solving. It takes a bit more of a deep dive to get used to compared to what a lot of people are used to with unit tests. In my experience this is very minor though. I've had everyone from senior developers to freshly spawned developers able to pick this up without any problems.

The biggest issue with this is _how_ your write the tests, specifically with regards to the setting up of the "default" data and mocks. The first issue is that the setup isn't confined to your test. You may often find yourself scrolling, or using the split window feature of your IDE, to see what's going on in the setup and the test you're writing. What I'd say is the biggest problem is deciding what the default setup should be. Sure, I gave the example of "happy path", but a function can have more than one happy path, do you only configure for one branch of the happy path or do you cover all of them? Does that change depending on the number tests for each branch? For how often users follow a branch in the real world? What if a new branching happy path is created? Or Removed? A lot of this is down to team dynamics. My team were very relaxed about this, which did mean that there were some small inconsistencies throughout the tests, but it was rare that I'd run into a test class that needed a slight tweak. There's always the fallback of overwriting the defaults in the individual test if you're not sure.

Finally, since this was a quick solution to a problem where the root cause was classes doing more than they should, the real solution to this is really to refactor the code into smaller chunks. Doing that would involve more work, but does help in other ways. Adding a testing framework like this just adds to the list of things to be done when that refactor eventually does come around.