---
title: So what is functional programming anyway?
description: "Hint: it's about programming with functions"
date: 2020-09-18
draft: false
tags: [code]
---

## Programming with functions
Functional programming, as the name implies, is **programming with functions**. Not classes, not objects, not any other abstraction, just functions.

Programming with functions is liberating. Functions are much simpler than objects. They can't be abstract, they don't have constructors, destructors, base classes, virtual methods, a `this` keyword (well... except if you're JavaScript, then functions do have a this keyword, and it's weird), a `super` keyword, etc. Here's one.

```fsharp
let add x y = 
    x + y
```

You might not know the language above, but you can probably read this anyway. This is a function called `add` that takes two parameters `x` and `y`, and returns the sum of these two parameters.

Here is FP's promise: you can make entire programs using no higher-level abstraction than this; and these programs will probably be less bug-prone and easier to maintain than if you were to imprison these in fancy *objects* with *encapsulation*. Let your functions be free and they will thank you. I don't think I can adequately explain why in a short blog post; but I hope to spike your curiosity with some examples.

## Composing functions together

Functional programming is about programming with functions, and this means we can pass functions around. No, not interfaces, just functions. Try to imagine what this could look like without silly-looking objects with silly names in silly code files. How about dependency injection?

```fsharp
let doComplexThingy dbFunction apiFunction =
    let apiResult = apiFunction()
    let dbResult = dbFunction(apiResult)
    dbResult
```

I didn't need to define interfaces and wonder what names to give them or what files to put them in, and register them with a DI provider and sacrifice a goat or two on the altar of object-oriented correctness. If I'm working with a nice language like the above, I don't even need to specify types, the compiler infers them for me. This is a function that takes two functions as parameters, and it doesn't care how they're implemented. The caller may be a unit test supplying stubs for these, or production code providing concrete implementations. This caller may decide to make a shorthand version that supplies the dependencies already:

```fsharp
let doComplexThingyConcrete () =
    doComplexThingy callDbForReal callApiForReal

// Now we can just do
doComplexThingyConcrete()
```
Notice how we don't need a constructor and private members to hold the dependencies. A function can capture whatever pieces of data it needs, either implicitely from its enclosing scope, or explicitely via arguments. Think about all the redundant code you might be saved from writing.

Dependency injection is but one of many tasks that can be achieved easily when building functions out of other functions, but let's leave it at that for now.

## Values over variables

Functional programming means stuff usually is immutable. This makes code *way* easier to understand. Let's say you see this in your code:

```fsharp
thisIsAVariable
```

And you wonder: hm, what value does it have? Where do it come from? Well, if stuff is not immutable, you have to look at everywhere it is assigned, and figure out, at this point in the code, what would have been the last assignment (and if it's assigned the value of another mutable variable, there you go again). This means there is a potentially unbounded amount of code you need to understand, for-loops, if-statements and all, before being able to tell what the value is! Chances are, you give up, put a `println` somewhere or a breakpoint and run the code since that'll be faster.

But if it's immutable, then your answer lies here:
```
Right-click -> Go to Definition 
```
That's it! This will bring you the single line of code you need to understand to figure out where this value comes from. 

In fact, since in FP a *variable* only ever has one value, we don't call them *variables*, we call them values!

```fsharp
let PI = 3.14159...
// There is no conceptual difference between the identifier and the value it holds.
// PI is 3.14159... and 3.14159... is PI.
```

## Data over entities

Functional programming is also about programming with *data* rather than *entities*. You realize this matters when trying to compare things:

```fsharp
let expected = {
    Address = {
        City = "Montreal",
        Line1 = "1234 Main St.",
        Line2 = "appt. 2",
        Province = "QC",
    }
    LastName = "Doe",
    FirstName = "John",
    // etc
}
let actual = callApi()

// Ugh, how do I make sure they are equal? Better not forget to check any field...
```

In language that emphasizes entities over data, this:
```js
expected === actual // JavaScript
expected == actual // basically all the other ones
```
is not the equality you're looking for. This actually checks if the values assigned to these two identifiers are the same *entity*, that is, the same location in memory! You may be able to override this operator for each and every type you want to be able to compare - but then how will you test these overrides?

Well, if we're programming with values, then our language (like the one we've been using so far) should provide us with structural equality, so we can just do
```fsharp
expected = actual
```
And this will return true if they are the same value, in the mathematical sense, like two matrices or two sets being equal.

This is only scratching the surface, but I hope I illustrated some key aspects of functional programming:
 - Functions over classes
 - Creating functions by composing functions together
 - Values over variables
 - Data over entities

These principles can be applied in any programming language, although some make it easier than others. This article has used F#.