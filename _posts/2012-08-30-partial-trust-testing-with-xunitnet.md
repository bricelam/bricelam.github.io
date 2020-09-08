---
layout: post
title: Partial Trust Testing with xUnit.net 1.9
permalink: /2012/08/partial-trust-testing-with-xunitnet.html
tags: xunit.net
---

One of the problems that framework teams occasionally face on the .NET platform is maintaining the ability to run under
partial trust. The most common use of partial trust is by web applications that have been [configured to run in medium
trust][1]. Under medium trust, certain calls may fail due to more restricted code access security permissions. This
includes things like using reflection, accessing environment variables, and file I/O operations on certain paths. This
post describes how we test partial trust scenarios on the [Entity Framework][2] team.

Shortly after we switched to using xUnit.net on the team, I took the liberty of updating some of our partial trust tests
to integrate a little better with the new testing framework. This resulted in an easy-to-use set of components that can
be used to test code running under partial trust. The code for these components is located under the
[test\EntityFramework\FunctionalTests\TestHelpers][3] directory of the repository. Please note that the code there is
owned by [Microsoft Open Technologies, Inc.][4] and licensed under the [Apache License, Version 2.0][5].

There are a few different ways of using the components which I'll describe in sections.

Opt-in tests
============
With this approach, individual tests can opt-in to running under partial trust using the `PartialTrustFact` attribute.

```cs
public class MyTests : MarshalByRefObject
{
    [Fact]
    public void Full_trust_test()
    {
        // Executes in full trust
    }

    [PartialTrustFact]
    public void Partial_trust_test()
    {
        // Executes in medium trust
    }
}
```

You may have noticed that the test class inherits from `MarshalByRefObject`. This is required. Under the covers, a new
`AppDomain` is created and configured for medium trust. A new instance of the test class is then created in this domain
and a reference is passed back in order to call the test method. If your class uses any test fixtures, they must also
either inherit from `MarshalByRefObject` or be decorated with the `Serializable` attribute.

Opt-out tests
=============
This is the exact opposite of the previous approach. Instead of partial trust tests opting in, full trust tests must opt
out by using the `FullTrust` attribute. To use this approach, decorate your test class with the `PartialTrustFixture`
attribute.

```cs
[PartialTrustFixture]
public class MyTests : MarshalByRefObject
{
    [Fact, FullTrust]
    public void Full_trust_test()
    {
        // Executes in full trust
    }

    [Fact]
    public void Partial_trust_test()
    {
        // Executes in medium trust
    }
}
```

This approach is a little more flexible in that your partial trust tests are no longer required to be simple `Fact`
tests. Here is an example of a partial trust `Theory`.

```cs
[PartialTrustFixture]
public class MyTests : MarshalByRefObject
{
    [Theory]
    [InlineData(1)]
    [InlineData(2)]
    public void Partial_trust_theory(int number)
    {
        // Executes in medium trust
    }
}
```

The `PartialTrustFixture` attribute can also be applied to base classes.

```cs
[PartialTrustFixture]
public class TestBase : MarshalByRefObject
{
}

public class MyTests : TestBase
{
    [Fact]
    public void Partial_trust_test()
    {
        // Executes in medium trust
    }
}
```

Mixed-trust tests
=================
There may be cases when you need to perform some setup that cannot be done in partial trust. If so, you will have to
drop down to using the `PartialTrustSandbox` component.

```cs
public class MyTests : MarshalByRefObject
{
    [Fact]
    public void Mixed_trust_test()
    {
        // Executes in full trust

        var this_pt = PartialTrustSandbox.Default
            .CreateInstance<MyTests>();

        this_pt.Partial_trust_part();
    }

    void Partial_trust_part()
    {
        // Executes in medium trust
    }
}
```

Summary
=======
When I designed these components, I wanted to make it dead simple for developers to write partial trust tests.
Considering that there are only four classes (three of which are just attributes), I feel that this goal was
accomplished. Hopefully my work will benefit other projects that need to test partial trust scenarios using xUnit.net.


  [1]: http://msdn.microsoft.com/en-us/library/ms998341.aspx
  [2]: http://msdn.com/data/ef
  [3]: http://entityframework.codeplex.com/SourceControl/changeset/view/ffc11def5ac2#test%2fEntityFramework%2fFunctionalTests%2fTestHelpers%2fPartialTrustSandbox.cs
  [4]: http://www.codeplex.com/site/users/view/MSOpenTech
  [5]: http://www.apache.org/licenses/LICENSE-2.0
