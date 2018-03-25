---
layout: post
title: "[Notes]: What’s inside of TurboFan? by Benedikt Meurer"
description: "Notes on What’s inside of TurboFan? by Benedikt Meurer"
tags: notes turbofan ignition v8 javascript
---

## Prelude

Some notes from the **What’s inside of TurboFan?** by Benedikt Meurer talk at AgentConf 2018. Watch
on YouTube [here](https://www.youtube.com/watch?v=Eowsw8XXVCQ).

## Actual Notes :D

### Intro to TurboShaft

- TurboFan is the optimizing compiler in the execution pipeline for the v8 JS engine.

- Replaces Crankshaft--the old optimizing compiler--which couldn't handle optimizations for new JS
  (ES2015+) features.

  - One interesting case of this is that Crankshaft couldn't handle optimizing try-catch and ES2015
    has a lot try-catch usage. For example, `for..of` loops have an implicit `try..catch`.

- TurboShaft begins compilation from bytecode whereas Crankshaft compiles from JS source. This is
  an important difference because Ignition (the v8 interpreter) generates bytecode as part of its
  iterpretation process. This means that TurboShaft has a headstart and doesn't have to re-parse
  the JS source as part of it's compilation process.

- TurboShaft takes the bytecode and generates a graph representation of it, which is then fed into
  the inlining and specialization phase. This phase will use runtime information from the
  interpreter to prune graph paths that are not used, thereby optimizing the code. This is the
  *frontend* of the compiler

- The result is passed through an optimization phase where textbook optimizations (e.g. redundancy
  elimination, escape analysis) are made to the graph representation of the code.

- The last phase (or *backend*) of the compiler performs machine-level optimizations and code
  generation. This phase is particular for the machine the code is running on (little redundant but
  worth repeating :D).

### Inlining Specifics

- Inline means to essentially take the body of a function and place it at the call site of said
  function.

- Might not look like much on its own but, combined with other optimization passes (e.g. constant
  folding), it can have amazing results.

- The compiler can also inline builtin JS functions (e.g. Array.prototype.every) and run further
  optimization passes on the library functions.

- Builtin inlining is important because it helps developers write idiomatic JS.

### Predictable Performance

- Crankshaft had really crazy performance cliffs. This means that special cases that differed from
  an optimized code path would incur large and unpredictable penalties.

- In Crankshaft, certain features (e.g. `for..of`, `const` usage, etc.), caused a function to be
  completely disabled for optimizations. TurboShaft removes this obstacle and allows functions
  containing these so called *"optimization killers"* to be optimized. It does not mean these
  features can be fully optimized, but at least the rest of the function is checked for
  optimization.

- A caveat to keep in mind about optimizing JS code is that, in real world scenarios, there may be
  *significantly* more time spent doing layout and style calculations. Fully focusing on getting the
  most optimal JS compilation may not be the best way to spend time.

- A key takeaway though is that when optimizations are made through TurboShaft there are
  significant performance increases.

## Useful references

- [Link to talk on YouTube](https://www.youtube.com/watch?v=Eowsw8XXVCQ)
- [Slides from the talk](https://docs.google.com/presentation/d/1kcb_NDzKlahXhjjtZMQsEIVeqMNrBVibO2HzOGuQ_Ag/edit?usp=sharing)