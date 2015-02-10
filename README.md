
## Turning off shrinking

When a property fails, FsCheck normally tries to shrink the failing value in order to report the simplest
possible value that still fails. However, sometimes it can be desirable not to automatically apply shrinking.
There is a type in FsCheck called <code>DontShrink&lt;a&gt;</code>. I have only recently figured out how to use it! 

## The Common Property Implementation

To demonstrate shrinking/not shrinking, we will use a test property that contains a deliberate error: 

```C#
private static bool PropertyImplementation(IList<int> xs)
{
    return xs.Reverse().SequenceEqual(xs);
}
```

## With Shrinking

The following test fails and FsCheck shrinks the failing value down to the smallest list that still fails: 

```C#
[Test]
public void SpecFor_UsesTheDefaultShrinkerForTheGivenType_WillShrink()
{
    Spec
        .For(Any.OfType<IList<int>>(), xs => PropertyImplementation(xs))
        .Check(Configuration);
}
```

I specified a <code>Configuration</code> that verbosely displays the shrinks. Here is the output: 

```
shrink:
seq [1; -11; -24; -21; ...]
shrink:
seq [-11; -24; -21; 10; ...]
shrink:
seq [-24; -21; 10; -22; ...]
shrink:
seq [-21; 10; -22; 23; ...]
shrink:
seq [10; -22; 23; 12; ...]
shrink:
seq [-22; 23; 12; -16; ...]
shrink:
seq [23; 12; -16; 21; ...]
shrink:
seq [12; -16; 21; 17]
shrink:
seq [-16; 21; 17]
shrink:
seq [21; 17]
shrink:
seq [21; 0]
shrink:
seq [11; 0]
shrink:
seq [6; 0]
shrink:
seq [3; 0]
shrink:
seq [2; 0]
shrink:
seq [1; 0]
System.Exception : Falsifiable, after 1 test (16 shrinks) (StdGen (1567478199,295970420)):

seq [1; 0]


   at <StartupCode$FsCheck>.$Runner.get_throwingRunner@349-1.Invoke(String message) in C:\Users\Kurt\Projects\FsCheck\fsharp\src\FsCheck\Runner.fs: line 349
   at FsCheck.Runner.check(Config config, a p) in C:\Users\Kurt\Projects\FsCheck\fsharp\src\FsCheck\Runner.fs: line 264
   at DontShrinkTests.DontShrinkTests.SpecFor_UsesTheDefaultShrinkerForTheGivenType_WillShrink() in DontShrinkTests.cs: line 59System.Exception : Falsifiable, after 1 test (16 shrinks) (StdGen (1567478199,295970420)):
```

## Without Shrinking

In this second test, I have explicitly used a type of <code>DontShrink&lt;IList&lt;int&gt;&gt;</code> instead of <code>IList&lt;int&gt;</code>. 

```C#
[Test]
public void SpecFor_WhenExplicitlyToldToUseTheDontShrinkWrapper_WillNotShrink()
{
    Spec
        .For(Any.OfType<DontShrink<IList<int>>>(), xs => PropertyImplementation(xs.Item))
        .Check(Configuration);
}
```

This time, FsCheck does not shrink the failing value as can be seen in the following output:

```
System.Exception : Falsifiable, after 1 test (0 shrinks) (StdGen (1840284312,295970419)):

DontShrink (seq [20; -18; -20; -9; ...])


   at <StartupCode$FsCheck>.$Runner.get_throwingRunner@349-1.Invoke(String message) in C:\Users\Kurt\Projects\FsCheck\fsharp\src\FsCheck\Runner.fs: line 349
   at FsCheck.Runner.check(Config config, a p) in C:\Users\Kurt\Projects\FsCheck\fsharp\src\FsCheck\Runner.fs: line 264
   at DontShrinkTests.DontShrinkTests.SpecFor_WhenExplicitlyToldToUseTheDontShrinkWrapper_WillNotShrink() in DontShrinkTests.cs: line 67
```
