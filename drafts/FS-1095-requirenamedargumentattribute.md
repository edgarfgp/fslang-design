# F# RFC FS-1095 - RequireNamedArgument attribute

The design suggestion [Add an attribute enforcing the use of named argument at callsite
](https://github.com/fsharp/fslang-suggestions/issues/414) has been marked "approved in principle".


This RFC covers the detailed proposal for this suggestion.

- [x] Approved in principle
- [x] [Suggestion](https://github.com/fsharp/fslang-suggestions/issues/414)
- [x] [Implementation](https://github.com/dotnet/fsharp/pull/11368)
- [ ] Design Review Meeting(s) with @dsyme and others invitees
- [x] [Discussion](https://github.com/fsharp/fslang-design/discussions/538)

# Summary

Allow to enforce usage of named arguments to method call sites by annotating `[<RequireNamedArgument>]` on the method definition.

# Motivation

This allows an API designer to enforce call sites abide to design choices in similar fashion to the existing attributes `RequireQualifiedAccess` and `RequiresExplicitTypeArguments`.

Applying the `RequireNamedArgument` attribute to the method definition will enforce call sites to use the named argument syntax.

This is useful when
* designing object oriented APIs, with similar feel to Smalltalk or Objective-C where method and argument names form a complete message signature
* describing the domain, and wanting the developers who is working on it to mindfully notice what they are passing into the method
* using subsequent arguments of same type can be confusing or prone to introducing bugs at call sites or during refactorings

This impacts some type providers where the order of parameter of a type provided member may switch due to adjustment of the input provided to a type provider.

# Detailed design

Basic example:

```fsharp
type A() =
  
  static member x.B(dividend:int, divisor:int) = dividend / divisor
  
  [<RequireNamedArgument>]
  static member x.C(dividend:int, divisor:int) = dividend / divisor

A.B(0, 15) // OK
A.C(0, 15) // Not OK
A.C(dividend=0, divisor=15) // OK
```

Error message:

`The method '%s' has the 'RequireNamedArgumentAttribute' attribute specified, use the named arguments syntax (e.g. 'MethodName(x = value)').`

# Drawbacks

* People consuming API using the feature may not adhere to the design choice made in the API design
* Record types could be used for similar effect (but type providers don't have the facility to generate those at this time)
* Increases FSharp.Core footprint

# Alternatives

Using record types, using single case/single field discriminated union, using units of measure. More details on those in [the discussion thread](https://github.com/fsharp/fslang-design/discussions/538).

# Compatibility

## Is this a breaking change?

It is not a breaking change.

## What happens when previous versions of the F# compiler encounter this design addition as source code?

Code would still compile but the rule won't be enforced. 

## What happens when previous versions of the F# compiler encounter this design addition in compiled binaries?

Code would still compile but the rule won't be enforced. 

## If this is a change or extension to FSharp.Core, what happens when previous versions of the F# compiler encounter this construct?

Code would still compile but the rule won't be enforced. 

# Unresolved questions

Whether the attribute would find a way to the BCL and support by other compilers.

Does the compiler errors if the attribute is used on a function? or a warning? or ignored?

What happens when the attribute is put on a virtual method but not on an overriden one?

What happens when the attribute is put on a method defined in an interface but not on the implementation?

What happens with optional parameters?

Should the feature support applying the attribute to only a subset of arguments, on argument by argument basis as well? How does it play out with restriction about named arguments coming last in the call site?

# Appendix

## Language Zoo

```smalltalk
'hello world' indexOf: $o startingAt: 6
```

```objectivec
[helloWorldString indexOf: o startingAt: 6];
```

```csharp
"hello world".Substring(startIndex:1);
```

```fsharp
"hello world".Substring(startIndex=1)
```