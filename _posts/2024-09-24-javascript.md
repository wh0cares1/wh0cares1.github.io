---
layout: single
title: "JavaScript Internals: A Beautiful Mess"
classes: wide
toc: true
---

> To understand a program you must become both the machine and the program. -Alan Perlis

# Introduction
A browser, such as Google Chrome, works behind the scenes by using a combination of compilers and interpreters to parse and execute JavaScript code seen on web pages. [Google's comic](https://www.google.com/googlebooks/chrome/med_00.html) gives a fantastic visual picture of this. When you view a webpage, the browser does more than simply show the text and graphics; it also executes the underlying code to make everything interactive. Here's when compilers and interpreters come in handy.
For example, an interpreter within the browser reads and executes JavaScript without first converting it into machine code. This enables quick execution and the dynamic behavior of current web pages. To improve performance, current browsers like as Chrome use just-in-time (JIT) compilers, which turn frequently used code into highly optimized machine code, resulting in faster execution. This combination of interpretation for flexibility and compilation for performance allows current browsers to execute web apps in an efficient and powerful manner.

# JavaScript Execution Context
## Call Stack & Event Loop
## Memory Management
### Memory Layout
V8's heap is divided into multiple spaces.
- Read-only space
- New space
- Old space
- Code space
- Map space
- Large object space
- Code large object space
- New large object space
New space:
- Most new objects end up here.
- Memory given out **linearly**.
- Divided into two regions.
Old space:
- Objects that survived two GC runs.
- Less likely to be moved around.

### Garbage Collection
The memory heap is the location where objects and data are dynamically allocated during execution. Garbage collection techniques are used in JavaScript engines to discover and reclaim memory used by no longer required objects. 
The garbage collector also runs every time a JavaScript file is executed, so it has probably been run more than any other component in v8. On top of that, it introduces a great deal of uncertainty in the heap layout. Every time the garbage collector runs, it moves objects around and changes the heap layout completely, and it’s difficult to predict when it is going to run. There’s also [concurrent garbage collection](https://v8.dev/blog/concurrent-marking), which adds even more uncertainty to the mix.
V8 Heap is split into different regions called generations garbage collection 
- Young generation using semi-spaces and **Copying Collection** with parallel Scavenger (marks, copies and updates pointers at the same time)
- Old generation using **[Mark-Sweep](https://wingolog.org/archives/2023/12/08/v8s-mark-sweep-nursery)/Mark-Sweep-Compact** with incremental marking (tri-color marking and write barriers)
Readmore :
- [Garbage Collection: V8’s Orinoco](https://medium.com/@nikolay.veretelnik/garbage-collection-v8s-orinoco-452b70761f0c)
- [Trash talk: the Orinoco garbage collector](https://v8.dev/blog/trash-talk)
- [High-Performance Garbage Collection For C++](https://v8.dev/blog/high-performance-cpp-gc)
- [A tour of V8: Garbage Collection](https://jayconrod.com/posts/55/a-tour-of-v8--garbage-collection)
- [Concurrent marking in V8](https://v8.dev/blog/concurrent-marking) 
- [Getting garbage collection for free](https://v8.dev/blog/free-garbage-collection) 
- [Jank Busters Part One](https://v8.dev/blog/jank-busters) 
- [Jank Busters Part Two: Orinoco](https://v8.dev/blog/orinoco)

#### GC Bugs
- [Chrome in-the-wild bug analysis: CVE-2021-37975](https://github.blog/security/vulnerability-research/chrome-in-the-wild-bug-analysis-cve-2021-37975/)
- [Triggering garbage collection with rejected promises to cause use-after-free in Chrome](https://securitylab.github.com/research/garbage-collection-uaf-chrome_gc/)

# JavaScript Primitives and Objects
## **JavaScript Primitives**
1. **Symbol**:
    - Unique and immutable, often used as object property keys to avoid property name collisions.
2. **String**:
    - A sequence of characters used to represent and manipulate text.
3. **Boolean**:
    - Represents logical true/false values, crucial for control flow.
4. **Number**:
    - Includes:
        - **Int**: Integer values.
        - **BigInt**: For working with arbitrarily large integers beyond the safe limit of `Number`.
        - **Double**: Represents floating-point numbers, which are more common in JavaScript.
5. **Undefined**:
    - A variable that has been declared but not yet assigned a value.
6. **Null**:
    - Represents the intentional absence of any object value.

## **JavaScript Native Objects**
### **Prototype-based Object Model**
1. **Class**:
    - JavaScript supports classes built on prototypes, allowing object-oriented design patterns like inheritance.
2. **Inheritance**:
    - **`this.__proto__`**: Points to the object's prototype, showing inheritance.
    - **`super`**: Calls functions on an object's parent.
3. **`this` in Arrow Functions**:
    - Arrow functions don't have their own `this` context, so they bind `this` lexically.

### **Object Properties Access (Dot(`.`) and Bracket(`[]`) Notation)**
1. **Attributes**:
    - **value**: Holds the data.
    - **writeable**: Defines if the property can be modified.
    - **enumerable**: Determines if the property shows up during iteration, like `Object.keys()` or the `in` operator.
    - **configurable**: Controls if the property can be deleted or changed, such as defining a setter or getter.
2. **Computed Properties**:
    - **Setter/Getter**: Enable functions to be called when getting or setting property values, making them dynamic.

## **Special JavaScript Objects**
1. **Array**:
    - **Typed**: Typed arrays allow for working with raw binary data in buffers.
    - **Length**: Automatically updates as items are added or removed.
2. **RegExp**:
    - Regular expressions for pattern matching within strings.
3. **ArrayBuffer**:
    - A low-level binary data buffer, essential for handling binary data, often used with typed arrays.
4. **Function**:
    - **Passed as object**: Functions are first-class objects and can be passed around like other objects.
    - **Hoisted**: Function declarations are hoisted, allowing them to be called before their definition.
    - **Arrow Function**:
        - **Compact**: Shorter syntax for anonymous functions.
    - **Constructor**: Functions can also serve as constructors to create new objects.
    - **Store as Variable**: Functions can be assigned to variables, passed as arguments, or returned from other functions.
    - **Variable**:
        - **Weakly Typed**: JavaScript variables don’t require a specific type.
        - **Coerced**: Variables are often coerced into different types implicitly.

# JavaScript Engines: V8 Internals
- JavaScript uses “[prototype-based-inheritance](https://262.ecma-international.org/6.0/#sec-objects)”, where each object has a reference to a prototype object or “shape” whose properties it incorporates.
- The compiler works ahead of time by utilizing a "Profiler" to monitor and watch code that needs to be optimized. If there is a "hot function," the compiler converts it into efficient machine code for execution. Otherwise, if it detects that a previously optimized "hot function" is no longer being utilized, it will "deoptimize" it and return it to bytecode.

## Ignition (V8’s Interpreter)
### Ignition Bytecode
- Ignition is a register machine
- Implicit accumulator register
- Argument registers a0, a1, ...
- General purpose registers r0, r1, ...
- Ldar/Star = load/store to accumulator

### Type Feedback
Type feedback is crucial for optimizations:
- Generic addition in javascript is very complex
- Addition of integers is very simple
- Integer feedback allows to lower instruction to integer addition

## Turbofan Pipeline

# Memory and Buffer Management in JavaScript
## Understanding Buffers

## Typed Arrays and Array Buffers

# Async JavaScript Internals
Promises and Async/Await
Callback Hell & Solutions

### **Proxy**
1. **Property Lookup**: Custom behavior for property access.
2. **Assignment**: Intercepts property assignment.
3. **Function Invocation**: Controls function calls within a proxy object.

### **Callbacks**
1. **Function Arguments**: JavaScript's standard built-in callback system often passes functions as parameters.
2. **Object Converters**:
    - **`valueOf`**: Converts an object to a primitive.
    - **`toString`**: Converts an object to a string representation.
3. **Property Getters and Setters**:
    - These trigger automatically when properties are accessed or modified, allowing for more controlled property behavior.

# Common JavaScript Vulnerabilities
Cross-Site Scripting (XSS)

Prototype Pollution

# What's Next?


# Source
https://zerotomastery.io/cheatsheets/javascript-cheatsheet-the-advanced-concepts/
https://medium.com/dailyjs/understanding-v8s-bytecode-317d46c94775
https://www.amazon.com/Professional-JavaScript-Developers-Matt-Frisbie/dp/1119366445
https://eloquentjavascript.net/
https://github.com/getify/You-Dont-Know-JS
https://en.wikipedia.org/wiki/Compiler
https://www.google.com/googlebooks/chrome/med_00.html
https://web.dev/articles/howbrowserswork
https://browser.engineering/
https://www.maxiferreira.com/blog/browser-engineering-1/