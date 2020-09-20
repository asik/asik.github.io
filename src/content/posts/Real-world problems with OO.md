---
title: Why encapsulation fails
description: And how pure functions can help
date: 2020-09-15
draft: false
tags: [code]
---

One promise of OOP is encapsulation. In its simplest definition, encapsulation is restricting access to an object's internal state. Encapsulation promises more than this: it promises to hide complexity to the user of a class by defining a **contract** (i.e. the **public** members of the class) and hiding (by making **private**) all other information, such that the client can operate on this contract alone, and not have to worry about how this contract is implemented. I don't think this should be controversial.

However...

## Implicit data dependencies defeat encapsulation


```csharp
class MyClass
    public void Initialize()
    public int Method1()

obj = new MyClass()
obj.Method1() // throws because Initialize() was not yet called
```

`MyClass` offers a leaky abstraction. Presumably, `Method1` relies on data that has to be provided by `Initialize`; although the consumer does not see this directly, he is affected by it all the same. The example above makes the error obvious, but keep in mind one does not always have the class definition in front of his eyes; furthermore, conditional logic, especially spread across different methods, may create a hard-to-spot code path leading to this error. It may well go undetected for a long time before leading to an expensive problem in production.

Temporal dependencies between methods (i.e. a method may do different things depending on what other method was called before) put the burden on the client to ensure the object is in the "desired state" before using it. Now the consumer has the worst of both worlds: having to manage the object's internal state, without direct visibility into this state, much less control.

Refactoring to pure functions makes this error impossible:

```csharp
static Data1 Initialize()

static int Function1(Data1 arg)
```

`Function1` cannot be called before `Initialize`, since it takes a `Data1` as a parameter, which only `Initialize` provides.
In a statically typed language, the compiler would simply not let the programmer get this wrong. Even without static types, the fact that the contract states the dependency explicitely makes it hard to miss.

## Multi-threading defeats encapsulation

It's fairly well known at this point that OOP makes multi-threaded programming difficult; sharing mutable objects between threads leads to data races. Locks attempt to address the problem by serializing access to this data; but this defeats the purpose of multi-threading which is to run things in *parallel*. 

Correct use of locks is difficult; errors lead to either more data races or deadlocks, which are some the trickiest problems to diagnose and fix.

The fundamental issue is that our OO languages were designed with single-threaded execution in mind, and a class can only keep its internal state coherent when execution (across all its methods) is kept serial.

```cs
// What happens if I call this on multiple threads?
myList.Add(3) 
```

Refactoring to pure functions, as much as feasible, eliminates data races and lets us run them on however many threads are available.

```fsharp
newList = List.add myList 3 // This is obviously thread-safe
```

Game programmers are familiar with **shaders**, small programs designed to run per-primitive (pixel, vertex, etc). Graphics cards run these on hundreds or thousands of small compute units. The reason they are massively parallelizable, enabling any game to leverage the power of a mini-supercomputer, is that a **shader** is a pure function.

There is something to be said about the current popularity of single-threaded languages (Python, JavaScript), which has somewhat obscured this issue for many programmers. This is a nice side-effect, for sure, but simply ignoring the multi-core nature of modern processors will only get us so far. In the meantime, our cores are sitting idle and our users are waiting.