---
layout: post
title: Think before you assert
tags: xunit.net
---

In addition to thinking about what conditions need to be met for the test to pass, think about how descriptive the
error should be when it fails.

The biggest anti-pattern is Assert.True (or Assert.False). I've seen some pretty elaborate expressions put between
those parenthesis, and when one thing goes wrong, all you get is "Assert.True() Failure"

//
// Boolean Asserts
//

Assert.False(true);
// Assert.False() Failure

Assert.True(false);
// Assert.True() Failure

//
// Collection Asserts
//

Assert.All(new[] { false, true }, x => Assert.True(x));
// Assert.All() Failure: 1 out of 2 items in the collection did not pass.
// [0]: Assert.True() Failure

Assert.Collection(
    new[] { false, true },
    _ => { });
// Assert.Collection() Failure
// Expected item count: 1
// Actual item count:   2

Assert.Collection(
    new[] { false, true },
    x => Assert.True(x),
    _ => { });
// Assert.Collection() Failure
// Error during comparison of item at index 0
// Inner exception: Assert.True() Failure

Assert.Contains(2, new[] { 0, 1 });
// Assert.Contains() Failure
// Not found: 2
// In value:  Int32[] [0, 1]

Assert.Contains(new[] { 0, 1 }, x => x == 2);
// Assert.Contains() Failure
// Not found: (filter expression)
// In value:  Int32[] [0, 1]

Assert.DoesNotContain(0, new[] { 0, 1 });
// Assert.DoesNotContain() Failure
// Found:    0
// In value: Int32[] [0, 1]

Assert.DoesNotContain(new[] { 0, 1 }, x => x == 0);
// Assert.DoesNotContain() Failure
// Found:    (filter expression)
// In value: Int32[] [0, 1]

Assert.Empty(new[] { 0 });
// Assert.Empty() Failure

Assert.Equal(new[] { 0, 1 }, new[] { 1, 0 });
// Assert.Equal() Failure
// Expected: Int32[] [0, 1]
// Actual:   Int32[] [1, 0]

Assert.None(new[] { 0, 1 }, 0);
Assert.None(new[] { 0, 1 }, x => x == 0);
// The collection contained 1 matching element(s) instead of 0.

Assert.NotEmpty(new int[0]);
// Assert.NotEmpty() Failure

Assert.NotEqual(new[] { 0, 1 }, new[] { 0, 1 });
// Assert.NotEqual() Failure
// Expected: Not [0, 1]
// Actual:   [0, 1]

Assert.Single(new[] { 0, 1 });
Assert.Single(new[] { 0, 1 }, x => x <= 1);
// The collection contained 2 matching element(s) instead of 1.

//
// Equality Asserts
//

Assert.Equal(0, 1);
// Assert.Equal() Failure
// Expected: 0
// Actual:   1

Assert.Equal(0.4, 0.6, precision: 0);
// Assert.Equal() Failure
// Expected: 0 (rounded from 0.4)
// Actual:   1 (rounded from 0.6)

Assert.StrictEqual(new int[0], new int[0]);
// Assert.Equal() Failure
// Expected: Int32[] []
// Actual:   Int32[] []

Assert.NotEqual(0, 0);
// Assert.NotEqual() Failure
// Expected: Not 0
// Actual:   0

Assert.NotEqual(0.1, 0.2, precision: 0);
// Assert.NotEqual() Failure
// Expected: Not 0 (rounded from 0.1)
// Actual:   0 (rounded from 0.2)

// TODO: Assert.NotStrictEqual

https://github.com/xunit/xunit/tree/master/src/xunit.assert/Asserts