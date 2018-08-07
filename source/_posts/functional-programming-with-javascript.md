---
title: Functional programming with Javascript
date: 2018-08-07 21:20:07
categories:
- Programming Paradigm
tags:
- Functional Programming
- Node.js
---

Functional programming (often abbreviated FP) is the process of building software by composing pure functions, avoiding shared state, mutable data, and side-effects. Functional programming is declarative rather than imperative, and application state flows through pure functions.

<!-- more -->

### Important concepts

A **pure function** is a function which:
* Given the same inputs, always returns the same output, and
* Has no side-effects

**Function composition** is the process of combining two or more functions in order to produce a new function or perform some computation.

**Shared state** is any variable, object, or memory space that exists in a shared scope, or as the property of an object being passed between scopes. A shared scope can include global scope or closure scopes.

The problem with shared state is that in order to understand the effects of a function, you have to know the entire history of every shared variable that the function uses or affects.

An **immutable object** is an object that can’t be modified after it’s created. Conversely, a mutable object is any object which can be modified after it’s created.

A **side effect** is any application state change that is observable outside the called function other than its return value.

A **higher order function** is any function which takes a function as an argument, returns a function, or both.

## References
