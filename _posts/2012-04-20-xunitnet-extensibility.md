---
layout: post
title: xUnit.net 1.9 Extensibility
permalink: /2012/04/xunitnet-extensibility.html
tags: xunit.net
---

Lately, I've taken to using [xUnit.net][1] as my unit testing framework of choice for .NET projects. The API is very
simple and straightforward as it tries not to re-invent concepts that are already a part of the language. For example
constructors and `IDisposable` are used instead of some new setup and teardown feature. It also makes great use of
generics and lambdas to keep its API surface to a minimum.

So far, the only bad thing I've found is the lack of documentation. I wanted to find out more about xUnit.net's
extensibility points so I could get more out of the framework. The best help I found was the Samples solution inside
[their main repository][2]. I decided to write a short post on what I found.

TraitAttribute
==============
This the simplest extensibility point. It's used to decorate a test method with arbitrary name-value pairs. In the
samples, they create a `Category` attribute that is based on `TraitAttribute`. A more useful implementation, perhaps, is
to keep track of the bug a particular regression test is for. Your custom attribute might look something like this.

```cs
class BugAttribute : TraitAttribute
{
    public BugAttribute(string id)
        : base("Bug", id)
    {
    }
}
```

You would then apply it to a test like this.

```cs
[Fact, Bug("3569")]
public void RegressionTest()
{
    // Test code here
}
```

IUseFixture<>
=============
xUnit.net creates an instance of your test class fore each test method. This is great for test isolation, but not so
good if your test setup takes a long time to run. Fixtures are instantiated just before the first test is run, and the
same instance is shared by all tests in the class. Also, if a fixture implements `IDisposable`, it will be disposed of
after the last test is run.

Here is an example of how to use a fixture.

```cs
public class UserRepositoryTests : IUseFixture<DatabaseFixture>
{
    DatabaseFixture _database;

    public void SetFixture(DatabaseFixture data)
    {
        _database = data;
    }

    [Fact]
    public void AddUser_creates_a_new_user()
    {
        var repo = new UserRepository(
            _database.Connection);

        repo.AddUser(new User { UserName = "bricelam" });

        var count = _database.Query<int>(
                "SELECT COUNT(*) " +
                "FROM Users " +
                "WHERE UserName='bricelam'");
        Assert.Equal(1, count);
    }

    [Fact]
    public void OtherTest()
    {
        // More test code here
    }
}
```

BeforeAfterTestAttribute
========================
Tests can have cross-cutting concerns too. That's where `BeforeAfterTestAttribute` comes in. It gives implementers two
hooks - `Before` and `After` - that get called each time a test method is executed. Here is an example implementation
for tracing.

```cs
class TraceAttribute : BeforeAfterTestAttribute
{
    public override void Before(MethodInfo methodUnderTest)
    {
        Trace.WriteLine(
            string.Format(
                "Before : {0}.{1}",
                methodUnderTest.DeclaringType.FullName,
                methodUnderTest.Name));
    }

    public override void After(MethodInfo methodUnderTest)
    {
        Trace.WriteLine(
            string.Format(
                "After : {0}.{1}",
                methodUnderTest.DeclaringType.FullName,
                methodUnderTest.Name));
    }
}
```

The [xUnit.net: Extensions][3] ship with this and a couple other useful implementations:

* `AssumeIdentity` – Replaces the `CurrentPrincipal` with another role
* `AutoRollbackAttribute` – Creates a `TransactionScope` around your test that gets rolled back when the test finishes

Using one of these attributes looks like this.

```cs
[Fact, Trace]
public void TestWithTracing()
{
    // Test code here
}
```

FactAttribute
=============
By now, you're more than familiar with the `Fact` attribute, but did you know that you can also create your own
attributes that derive from it? Here is an example that repeats a test for the specified number of times.

```cs
class RepeatTestAttribute : FactAttribute
{
    readonly int _count;

    public RepeatTestAttribute(int count)
    {
        _count = count;
    }

    protected override IEnumerable<ITestCommand> EnumerateTestCommands(
        IMethodInfo method)
    {
        return base.EnumerateTestCommands(method)
            .SelectMany(tc => Enumerable.Repeat(tc, _count));
    }
}
```

The above example just passes the underlying `ITestCommand` directly through, but you can access a whole host of other
extensibility points by creating your own implementation. The extensions' `TheoryAttribute` in conjunction with the
`TheoryCommand` class is a great example of this.

Applying your custom attribute is not different than you'd expect.

```cs
[RepeatTest(5)]
public void MyRepeatedTest()
{
    // Test code to repeat here
}
```

RunWithAttribute
================
The final and most advanced extensibility point is the `RunWithAttribute`. The attribute itself is very simple; all it
does is point to a class that implements `ITestClassCommand`.

```cs
class PrioritizedFixtureAttribute : RunWithAttribute
{
    public PrioritizedFixtureAttribute()
        : base(typeof(PrioritizedFixtureClassCommand))
    {
    }
}
```

The real power is in the target class. Here is one example that runs a set of tests in order of their priority.

```cs
class PrioritizedFixtureClassCommand : ITestClassCommand
{
    // The default implementation.
    // Assume members not shown simply delegate to this
    readonly TestClassCommand _inner = new TestClassCommand();

    public int ChooseNextTest(ICollection<IMethodInfo> testsLeftToRun)
    {
        return 0;
    }

    public IEnumerable<IMethodInfo> EnumerateTestMethods()
    {
        return from m in _inner.EnumerateTestMethods()
                let priority = GetPriority(m)
                orderby priority
                select m;
    }

    private static int GetPriority(IMethodInfo method)
    {
        var priorityAttribute = method
            .GetCustomAttributes(typeof(TestPriorityAttribute))
            .FirstOrDefault();

        return priorityAttribute == null
            ? 0
            : priorityAttribute.GetPropertyValue<int>("Priority");
    }
}

[AttributeUsage(AttributeTargets.Method, AllowMultiple = false)]
class TestPriorityAttribute : Attribute
{
    readonly int _priority;

    public TestPriorityAttribute(int priority)
    {
        _priority = priority;
    }

    public int Priority
    {
        get { return _priority; }
    }
}
```

The `TestPriority` attribute is used to influence how the tests are ran. Here's what your test code might look like
after putting it all together.

```cs
[PrioritizedFixture]
public class MyTests
{
    [Fact, TestPriority(1)]
    public void FirstTest()
    {
        // Test code here is always run first
    }

    [Fact, TestPriority(2)]
    public void SecondTest()
    {
        // Test code here is run second
    }
}
```


  [1]: http://xunit.codeplex.com
  [2]: http://xunit.codeplex.com/SourceControl/list/changesets
  [3]: http://nuget.org/packages/xunit.extensions
