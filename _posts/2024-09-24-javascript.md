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

---
# JavaScript Execution Context
## Event Loop
- The event loop continuously monitors the Call Stack and the Callback Queue. If the call stack is empty, the event loop executes the first task in the callback queue.
- Workflow: 
    - JavaScript executes synchronous code and adds functions to the call stack.
    - The browser's Web APIs support asynchronous functions (such as setTimeout and fetch).
    - Once the asynchronous task is completed, the relevant callback is added to the callback queue.
    - The event loop checks whether the call stack is empty. If it is, the next job from the callback queue is moved to the call stack and executed.
    - If any microtasks are available, they are executed before the next callback in the queue.

### Call Stack
- JavaScript is a single threaded programming language, which means it has a single Call Stack. Therefore, it can do one thing at a time.
- The Call Stack is a data structure that tracks where we are in the program. 
- If we enter a function, we move it to the top of the stack. When we return from a function, we pop off the top of the stack. That is all the stack can do. Each element in the Call Stack is called a **Stack Frame**. 
- Running code on a single thread can be simple since you don't have to deal with the complex problems that arise in multi-threaded settings, such as deadlocks.

### Web APIs
- JavaScript in the browser has access to a variety of Web APIs, including setTimeout(), XMLHttpRequest, and DOM event listeners. These APIs enable JavaScript to offload operations that might otherwise stall the main thread, such as making network queries or waiting for a timeout.

### Callback Queue (Task Queue)
- When an asynchronous job (such as an API response or a timeout) is ready to be handled, the related callback is added to the Callback Queue.

### Microtasks
- These are tasks that have a greater priority than other tasks in the queue. Callbacks for MutationObserver and Promises are two examples. Microtasks are completed before the next task in the callback queue.

### Readmore about Event Loop
- [Jake Archibald - In The Loop](https://www.youtube.com/watch?v=cCOL7MC4Pl0)
- [Philip Roberts: What is an event loop?](https://www.youtube.com/watch?v=8aGhZQkoFbQ)
- [The Node.js Event Loop](https://nodejs.org/en/learn/asynchronous-work/event-loop-timers-and-nexttick#the-nodejs-event-loop)
- [Using microtasks in JavaScript with queueMicrotask()](https://developer.mozilla.org/en-US/docs/Web/API/HTML_DOM_API/Microtask_guide)
- [Further Adventures of the Event Loop - Erin Zimmer](https://www.youtube.com/watch?v=u1kqx6AenYw)

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

---
# JavaScript Primitives and Objects
## JavaScript Primitives
- **Symbol**
    - Unique and immutable, often used as object property keys to avoid property name collisions.
- **String**
    - A sequence of characters used to represent and manipulate text.
- **Boolean**
    - Represents logical true/false values, crucial for control flow.
- **Number**
    - **Int**: Integer values.
    - **BigInt**: For working with arbitrarily large integers beyond the safe limit of `Number`.
    - **Double**: Represents floating-point numbers, which are more common in JavaScript.
- **Undefined**
    - A variable that has been declared but not yet assigned a value.
- **Null**
    - Represents the intentional absence of any object value.

## JavaScript Native Objects
### Prototype-based Object Model
- **Class**
    - JavaScript supports classes built on prototypes, allowing object-oriented design patterns like inheritance.
- **Inheritance**
    - **`this.__proto__`**: Points to the object's prototype, showing inheritance.
    - **`super`**: Calls functions on an object's parent.
- **`this` in Arrow Functions**
    - Arrow functions don't have their own `this` context, so they bind `this` lexically.

### Object Properties Access
- **Attributes**:
    - **value**: Holds the data.
    - **writeable**: Defines if the property can be modified.
    - **enumerable**: Determines if the property shows up during iteration, like `Object.keys()` or the `in` operator.
    - **configurable**: Controls if the property can be deleted or changed, such as defining a setter or getter.
- **Computed Properties**:
    - **Setter/Getter**: Enable functions to be called when getting or setting property values, making them dynamic.

## Object Representation
### Map
- The **Map** in V8 represents an internal pointer to the object’s **HiddenClass**, which contains metadata describing the structure (or "shape") of the object. This shape provides a blueprint for the engine to understand where properties are stored, the object's prototype, and other relevant characteristics.
- **HiddenClasses** work like a blueprint for object structure, allowing properties to be stored at **fixed offsets** in memory. This makes property access highly efficient because V8 can simply compute the offset instead of performing a costly lookup.
- New hidden classes are only created when **named properties** are added or deleted. Adding an array-indexed property (e.g., `arr[0]`) does **not** trigger a new hidden class because arrays are handled differently in V8.
- Contains
	- **The type** of the object, whether it’s a regular object, array, or function.
	- **The size** of the object in memory, critical for understanding memory allocation.
	- **Type of Array Elements**: For arrays, the type of elements (e.g., integers, floats, or objects) is stored.
	- The object’s **prototype**, which is part of the prototype chain lookup for inherited properties and methods.
	- **Instance Descriptor**: Points to a **DescriptorArray** that holds information about the properties of the object.
	- **Back_Pointer**: Helps V8 keep track of previous states when transitioning between hidden classes.

### Properties 
- A pointer to an object containing **named** properties.
- **Fast Properties**: When the object has a small, fixed set of properties, V8 uses a **fast path** to store properties in a linear structure. Each property can be accessed with a fixed offset based on the object’s hidden class (map).
- **Slow Properties**: When objects undergo significant changes, such as properties being added and removed frequently, V8 switches to a **slow properties path**, using a **dictionary** structure to manage properties.
- Shape transitions only occur for **fast properties**. If an object moves to **slow properties**, it no longer shares its hidden class structure with other objects. Instead, it has a unique dictionary representation that allows for arbitrary property changes.

### Element
- Represent the **array-like properties** (numbered properties) of an object. JavaScript arrays and objects can have properties with numeric keys, and V8 handles these differently from named properties.
- Arrays and objects can have numeric properties, which are stored in a dedicated **element store** in V8.
- A pointer to an object containing **numbered** properties.
- V8 makes a clear distinction between different types of elements allows the engine to optimize access patterns:
	- **SMI_ELEMENTS**: Small integers.
	- **DOUBLE_ELEMENTS**: Floating-point numbers.
	- **ELEMENTS**: Generic objects
- V8 optimizes operations on **packed arrays** (arrays without holes). Arrays with contiguous elements are much faster to access than those with missing values (**holey arrays**). Operations on packed arrays are more efficient because the engine can access elements sequentially in memory without checks.

### In-Object Properties
- Pointers to named properties that were defined at object initialization. They are stored directly in the object itself, rather than in an external property storage structure.
- If an object’s structure changes (e.g., more properties are added than initially defined), V8 may need to store some properties in an **external property store** (e.g., the properties dictionary for slow properties).

## Special JavaScript Objects
- **Array**:
    - **Typed**: Typed arrays allow for working with raw binary data in buffers.
    - **Length**: Automatically updates as items are added or removed.
- **RegExp**:
    - Regular expressions for pattern matching within strings.
- **ArrayBuffer**:
    - A low-level binary data buffer, essential for handling binary data, often used with typed arrays.
- **Function**:
    - **Passed as object**: Functions are first-class objects and can be passed around like other objects.
    - **Hoisted**: Function declarations are hoisted, allowing them to be called before their definition.
    - **Arrow Function**:
        - **Compact**: Shorter syntax for anonymous functions.
    - **Constructor**: Functions can also serve as constructors to create new objects.
    - **Store as Variable**: Functions can be assigned to variables, passed as arguments, or returned from other functions.
    - **Variable**:
        - **Weakly Typed**: JavaScript variables don’t require a specific type.
        - **Coerced**: Variables are often coerced into different types implicitly.

---
# JavaScript Engines: V8 Internals
- JavaScript uses “[prototype-based-inheritance](https://262.ecma-international.org/6.0/#sec-objects)”, where each object has a reference to a prototype object or “shape” whose properties it incorporates.

## Ignition (V8’s Interpreter)
- When JavaScript is first loaded, it is translated to bytecode and executed by Ignition, V8's interpreter. Ignition is quick to load code, but slow to execute it repeatedly since interpreting bytecode is less efficient than running compiled machine code.

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

#### Value Edges
- Value Edges shows the flow of values between operations in JavaScript code. 
- They demonstrate how the output of one operation is utilized as an input for another. 
- These edges represent dependencies in which the result of a calculation or the value of an object is required as input for later actions. 

##### *Inputs to Functions/Comparisons/Operations*
- Value edges capture how values travel across distinct sections of code. For example, if a function call requires specific arguments or a comparison operation requires two operands, the inputs are connected together by value edges.

##### *Relaxed Execution Order*
- In many circumstances, the execution order of actions connected by value edges can be changed. This means that the optimizer can reorder the sequence in which values are computed to increase speed, as long as the relationships between values are preserved.

#### Control Edges
- Control edges define the program's control flow, indicating which activities are dependent on the execution of others. 
- These edges guarantee that the execution sequence conforms to the program's logical structure, such as managing conditional branching, loops, and jumps in the control flow.

##### *Represents the Control Flow Graph*
- The control flow graph (CFG) represents all potential pathways through the program during execution. Control edges guarantee that the order of actions corresponds to the planned control flow.

##### *Solid Lines in Turbolizer*
- The Turbolizer tool represents both control and value edges as solid lines, making it simple to see how the optimizer manages both data and control flow in a unified manner.

#### Effect Edges
- Effect Edges control dependencies between stateful actions, which are activities that have side effects or change the state of the application. They guarantee that side effects are executed in the right sequence, preventing assumptions about the program's state from being invalidated.

##### *Ordering of Stateful Operations*
- Effect edges ensure the right sequence of actions that change the program's state. For example, if one operation writes to a variable while another reads from it, the effect edge guarantees that the write comes first.

##### *Side Effects Impact Later Operations*
- Certain activities, like as updating an object, writing to the console, or interacting with the DOM, might have side effects that influence subsequent operations. Effect edges handle these dependencies to ensure program correctness.

##### *Not Invalidating Assumptions*
- When the optimizer reorders stateful actions, the program's logic may fail if assumptions about the state are violated. Effect edges guarantee that such assumptions are upheld, preserving the right order of actions.

##### *Effect Chain*
- Even actions with minimal side effects may require wiring into the effect chain to preserve appropriate order. Certain mathematical operations, for example, might be re-ordered unless they are part of an effect chain that is triggered by earlier state changes.

### Readmore Ignition
- [Firing up the Ignition interpreter](https://v8.dev/blog/ignition-interpreter)
- [Ignition: Jump-starting an Interpreter for V8](https://docs.google.com/presentation/d/1HgDDXBYqCJNasBKBDf9szap1j4q4wnSHhOYpaNy5mHU/edit#slide=id.g1357e6d1a4_0_58)
- [Ignition: An Interpreter for V8](https://docs.google.com/presentation/d/1OqjVqRhtwlKeKfvMdX6HaCIu9wpZsrzqpIVIwQSuiXQ/edit)
- [Ignition Design Document](https://docs.google.com/document/d/11T2CRex9hXxoJwbYqVQ32yIPMh0uouUZLdyrtmMoL44/edit?ts=56f27d9d#heading=h.6jz9dj3bnr8t)
- [Ignition: Register Equivalence Optimization](https://docs.google.com/document/d/1wW_VkkIwhAAgAxLYM0wvoTEkq8XykibDIikGpWH7l1I/edit?ts=570d7131#heading=h.6jz9dj3bnr8t)
- [Understanding V8’s Bytecode](https://medium.com/dailyjs/understanding-v8s-bytecode-317d46c94775)
- [Blazingly Fast Parsing, Part 2](https://v8.dev/blog/preparser)
- [A guided tour through Chrome's javascript compiler](https://docs.google.com/presentation/d/1DJcWByz11jLoQyNhmOvkZSrkgcVhllIlCHmal1tGzaw/edit#slide=id.p)

## Sparkplug (V8’s non optimizing compiler)
- Sparkplug converts the bytecode directly to machine code without doing extensive optimization. Its goal is to create machine code faster than the optimizing compiler (Turbofan), allowing execution to proceed more quickly, particularly for smaller or short-lived routines that may not benefit from severe optimization.

### Readmore Sparkplug
- [Sparkplug — a non-optimizing JavaScript compiler](https://v8.dev/blog/sparkplug)
- [Sparkplug](https://docs.google.com/document/d/13c-xXmFOMcpUQNqo66XWQt3u46TsBjXrHrh4c045l-A/edit)
- [Sparkplug, the new lightning-fast V8 baseline JavaScript compiler](https://medium.com/@yanguly/sparkplug-v8-baseline-javascript-compiler-758a7bc96e84)

## Maglev (V8's Mid Tier Compiler)
- Maglev creates **less optimized** code than the top-tier JIT compiler, TurboFan, but it compiles quicker. JIT compilers are prevalent in Javascript engines, with the expectation that many layer compilers will provide a better compromise between compilation time and runtime optimization.
- Maglev converts bytecodes into SSA (Static Single-Assignment) nodes, which are defined in the file [maglev-ir.h](https://source.chromium.org/chromium/chromium/src/+/376123191e0361a8639c1783abcf2490a506c748:v8/src/maglev/maglev-ir.h).
- Maglev's compilation process consists of two optimization phases: [building a graph](https://source.chromium.org/chromium/chromium/src/+/376123191e0361a8639c1783abcf2490a506c748:v8/src/maglev/maglev-compiler.cc;l=403) from SSA nodes and [optimizing Phi value](https://source.chromium.org/chromium/chromium/src/+/376123191e0361a8639c1783abcf2490a506c748:v8/src/maglev/maglev-compiler.cc;l=421) representations.

### Readmore Maglev
- [Maglev - V8’s Fastest Optimizing JIT](https://v8.dev/blog/maglev)
- [Maglev Doc](https://docs.google.com/document/d/13CwgSL4yawxuYg3iNlM-4ZPCB8RgJya6b8H_E2F-Aek/edit#heading=h.djws22xta9wz)
- [Maglev Compiler](https://chromium.googlesource.com/v8/v8/+/f73f3b3b5122b806d898c8799da2c104d6bc2c56/src/maglev/maglev-compiler.cc)
- [Maglev](https://chromium.googlesource.com/v8/v8/+/refs/heads/main/src/maglev/)

## Turbofan (V8’s JIT Compiler)
- V8's Turbofan Compiler converts Ignition bytecode into assembly.
- Turbofan translates the bytecode into "[Sea of Nodes](https://darksi.de/d.sea-of-nodes/)" and subsequently to assembly.
- Turbofan is an optimizing compiler that use interpreter feedback to do "speculative" optimizations.
- The compiler works ahead of time by utilizing a "Profiler" to monitor and watch code that needs to be optimized. If there is a "hot function," the compiler converts it into efficient machine code for execution. Otherwise, if it detects that a previously optimized "hot function" is no longer being utilized, it will "deoptimize" it and return it to bytecode.

### Turbofan Graph Building
- Readmore: [bytecode-graph-builder.cc](https://source.chromium.org/chromium/v8/v8.git/+/main:src/compiler/bytecode-graph-builder.cc)

### Turbofan Optimize Graph
- Almost all optimization takes place on the sea of nodes.
	- Top-down and bottom-up graph transformations.
	- Separates transformations from error-prone computations. 
	- Local reasoning results in gradual transformations.

### Turbofan Optimization Pipeline
- Speculations
	- **Assumptions** are made about the object's type
	- $Speculative.*
		- Example : `SpeculativeNumberBitwiseAnd`
- Reductions
	- **Strategies** used during optimizations to reduce Nodes
	- Nodes might be optimized away
	- $Reduce.*
		- Example : `ReduceWordNAnd`
- Multiple Phases
	- Optimizations pipeline contains different phases
	- Example : Typer Phase, Type Lowering Phase, Effect Linearization Phase

### Readmore Turbofan
- [An Introduction to Speculative Optimization in V8](https://benediktmeurer.de/2017/12/13/an-introduction-to-speculative-optimization-in-V8/)
- [Digging into the TurboFan JIT](https://v8.dev/blog/turbofan-jit)
- [Deoptimize me not, v8](https://darksi.de/a.deoptimize-me-not/)
- [How to start JIT-ting](https://darksi.de/4.how-to-start-jitting/)
- [Sea of Nodes](https://darksi.de/d.sea-of-nodes/)
- [Turbofan Docs](https://v8.dev/docs/turbofan)
- [Hooking up the Ignition to the Turbofan](https://docs.google.com/presentation/d/1chhN90uB8yPaIhx_h2M3lPyxPgdPmkADqSNAoXYQiVE/edit)
- [Tale of Turbofan](https://benediktmeurer.de/2017/03/01/v8-behind-the-scenes-february-edition)
- [Ignition+TurboFan and ES2015](https://benediktmeurer.de/2016/11/25/v8-behind-the-scenes-november-edition)
- [CodeStubAssembler Redux](https://docs.google.com/presentation/d/1u6bsgRBqyVY3RddMfF1ZaJ1hWmqHZiVMuPRw_iKpHlY/edit#slide=id.p)
- [Overview of the Turbofan Compiler](https://docs.google.com/presentation/d/1H1lLsbclvzyOF3IUR05ZUaZcqDxo7_-8f4yJoxdMooU/edit)
- [Turbofan IR](https://docs.google.com/presentation/d/1Z9iIHojKDrXvZ27gRX51UxHD-bKf1QcPzSijntpMJBM)
- [Turbofan’s JIT Design](https://docs.google.com/presentation/d/1sOEF4MlF7LeO7uq-uThJSulJlTh--wgLeaVibsbb3tc)
- [Fast Arithmetic for Dynamic Languages](https://docs.google.com/a/google.com/presentation/d/1wZVIqJMODGFYggueQySdiA3tUYuHNMcyp_PndgXsO1Y)
- [Deoptimization in V8](https://docs.google.com/presentation/d/1Z6oCocRASCfTqGq1GCo1jbULDGS-w-nzxkbVF7Up0u0)
- [Turbofan a new code generation architecture for V8](https://docs.google.com/presentation/d/1_eLlVzcj94_G4r9j9d_Lj5HRKFnq6jgpuPJtnmIBs88)
- [An Internship on Lazyness Slides](https://docs.google.com/presentation/d/1AVu1wiz6Deyz1MDlhzOWZDRn6g_iFkcqsGce1F23i-M/edit)
- [An internship on laziness: lazy unlinking of deoptimized functions](https://v8.dev/blog/lazy-unlinking)
- [Turbofan: Function Context Specification](https://docs.google.com/document/d/1CJbBtqzKmQxM1Mo4xU0ENA7KXqb1YzI6HQU8qESZ9Ic/edit)
- [Turbofan: Rest Parameters and Arguments Exotic Objects optimization plan](https://docs.google.com/document/d/1DvDx3Xursn1ViV5k4rT4KB8HBfBb2GdUy3wzNfJWcKM/edit)
- [Turbofan Developer Tools Integration](https://docs.google.com/document/d/1zl0IA7dbPffvPPkaCmLVPttq4BYIfAe2Qy8sapkYgRE)
- [Turbofan Inlining](https://docs.google.com/document/d/1l-oZOW3uU4kSAHccaMuUMl_RCwuQC526s0hcNVeAM1E)
- [Turbofan Inlining Heuristics](https://docs.google.com/document/d/1VoYBhpDhJC4VlqMXCKvae-8IGuheBGxy32EOgC2LnT8)
- [TurboFan redundant bounds and overflow check elimination](https://docs.google.com/document/d/1R7-BIUnIKFzqki0jR4SfEZb3XmLafa04DLDrqhxgZ9U)
- [Turbofan Lazy deoptimization without code patching](https://docs.google.com/document/d/1ELgd71B6iBaU6UmZ_lvwxf_OrYYnv0e4nuzZpK05-pg)
- [Turbofan Register Allocator](https://docs.google.com/document/d/1aeUugkWCF1biPB4tTZ2KT3mmRSDV785yWZhwzlJe5xY)
- [Projection nodes in TurboFan](https://docs.google.com/document/d/1C9P8T98P1T_r2ymuUFz2jFWLUL7gbb6FnAaRjabuOMY/edit)
- [Builtin optimization guards in TurboFan](https://docs.google.com/document/d/1R0zPOsX95L-x19PTfMJhIcnDcuwZJ9zl04gpmCwIyjs/edit)
- [Investigation of (transpiled) class performance in V8](https://docs.google.com/document/d/1paV2Re7YAqwJlMvunToRV8ZqBXHzSaoXiLrWJsmZDVg/edit)
- [In-place field representation changes](https://docs.google.com/document/d/10CbqmRs-i8Jy0IE3ToEP25_FD8gj2kEHvfd3N0icN3g/preview)
- [ES2015 and beyond performance plan](https://docs.google.com/document/d/1EA9EbfnydAmmU_lM8R_uEMQ-U_v4l9zulePSBkeYWmY/edit)
- [Fast string concatenation in JavaScript](https://docs.google.com/document/d/1o-MJPAddpfBfDZCkIHNKbMiM86iDFld7idGbNQLuKIQ/preview)
- [Context-sensitive JavaScript operators in TurboFan](https://docs.google.com/document/d/1x3ittDiaRksghx9My0Lom8Bau2iJELJCjpH2jW9q6tE/edit)
- [Fast frozen & sealed elements in V8](https://docs.google.com/document/d/1X6zO5F_Zojizn2dmo_ftaOWsY8NltPHUhudBbUzMxnc/preview)
- [Faster calls with arguments mismatch](https://docs.google.com/document/d/156nh17BpyLNmDmONyC_ZQUbi0h6goNysds2DLXcAzVc/preview)
- [Faster Runtime API Calls](https://docs.google.com/document/d/1vjddeZw9HmAWOy2pkJtDOW0KtAueLpvGlY1od-uijd0/edit#heading=h.n1atlriavj6v)
- [Adventures in JIT compilation: Part 1 - an interpreter](https://eli.thegreenplace.net/2017/adventures-in-jit-compilation-part-1-an-interpreter/ "Permalink to Adventures in JIT compilation: Part 1 - an interpreter")
- [Adventures in JIT compilation: Part 2 - an x64 JIT](https://eli.thegreenplace.net/2017/adventures-in-jit-compilation-part-2-an-x64-jit/ "Permalink to Adventures in JIT compilation: Part 2 - an x64 JIT")
- [Escape Analysis in Turbofan](https://docs.google.com/presentation/d/1YdpdI1aeBnlchyAYvjw1Alm3QcoenPm6x4D7DA4pQUY/edit#slide=id.p)
- [A crash course in just-in-time (JIT) compilers](https://hacks.mozilla.org/2017/02/a-crash-course-in-just-in-time-jit-compilers/)

---
# Memory and Buffer Management in JavaScript
When dealing with binary data or interfacing with Web APIs, memory management becomes more explicit. This is where buffers, typed arrays, and array buffers come into play.

## Understanding Buffers
Buffers are typically used to handle raw binary data, which is not natively supported by JavaScript's conventional data types such as strings and arrays. Buffers allow you to process binary streams of data more effectively.

### Key Characteristics of Buffers:
- **Fixed Size**: Buffers have a predefined size, which means once created, the amount of memory they consume cannot change.
- **Raw Binary Data**: Buffers store raw bytes, unlike JavaScript strings that are encoded in UTF-16.
- **Useful in Network and File Operations**: Buffers are particularly useful in low-level operations like interacting with network protocols, reading or writing binary files, or working with images, videos, and other media streams.

## Array Buffers
- An **ArrayBuffer** is a generic, fixed-length block of raw memory.
- An ArrayBuffer doesn’t have any methods to manipulate this data; you need to use a **Typed Array** or **DataView** to access the memory.
- Use Cases:
	- **WebGL**: Typed arrays are essential in WebGL for handling 3D graphics, where binary data representing vertices, textures, and shaders needs to be passed directly to the GPU.
	- **Networking**: When working with low-level protocols or binary streaming over WebSockets or other network interfaces.
	- **File I/O**: Reading and writing binary files (e.g., images, audio files) using File APIs or interacting with file system streams in Node.js.
	- **Multimedia**: Manipulating images, audio, and video data for encoding/decoding purposes.
	- **WebAssembly**: Typed arrays are used in conjunction with WebAssembly to provide memory access for high-performance applications.

### Typed Arrays
- Typed Arrays are array-like objects that provide a view over an ArrayBuffer, allowing you to read and write binary data directly.
- Common Typed Array Types:
	- **`Int8Array`**: 8-bit signed integer
	- **`Uint8Array`**: 8-bit unsigned integer (no negative values)
	- **`Int16Array`**: 16-bit signed integer
	- **`Uint16Array`**: 16-bit unsigned integer
	- **`Float32Array`**: 32-bit floating-point number
	- **`Float64Array`**: 64-bit floating-point number

### DataView
- Another way to manipulate ArrayBuffers, offering more flexibility than typed arrays.
- Let you to access any area of the buffer using *a specific byte offset* and *value type*. This is important when dealing with complex binary formats in which data types may not be properly aligned.

---
# Asynchronous
## Promises
- A JavaScript object that indicates the eventual success (or failure) of an asynchronous action and its value. It serves as a placeholder for the outcome of an operation that hasn't yet finished but will at some time in the future.
- Key Concepts
	- **Pending**: The initial state. The operation is ongoing, and its result isn't available yet.
	- **Fulfilled**: The operation has completed successfully, and the promise now holds the resulting value.
	- **Rejected**: The operation failed, and the promise holds the reason for failure (typically an error object).

## Async/Await
- Async/Await is a more current and syntactically clear approach to working with promises. Async/await was introduced in ES2017 (ES8) and allows asynchronous code to be expressed in a more synchronous, linear way, which improves readability and maintainability.
- Key Concepts
	- `async function`: Declares an asynchronous function that implicitly returns a promise. 
	- `await`: Pauses the execution of an `async` function until a promise settles (fulfills or rejects). It allows you to retrieve the resolved value without chaining `.then()` calls.

## Callbacks
- **Function Arguments**: JavaScript's standard built-in callback system often passes functions as parameters.
- **Object Converters**:
    - **`valueOf`**: Converts an object to a primitive.
    - **`toString`**: Converts an object to a string representation.
- **Property Getters and Setters**:
    - These trigger automatically when properties are accessed or modified, allowing for more controlled property behavior.
- **Proxy**: Proxies are often referred to as "meta-programming" tools because they let you modify the behavior of language constructs.
    - **Property Lookup**: Custom behavior for property access.
    - **Assignment**: Intercepts property assignment.
    - **Function Invocation**: Controls function calls within a proxy object.
    - Structure:
        - **Target Object**: The object being proxied. It can be any type of object (including arrays or functions).
        - **Handler Object**: This object contains *traps* (intercepting methods) that define custom behavior for operations performed on the proxy.
- **Object Overrides**
	- Extending the default behavior of built-in objects such as Object, Array, Function, and User-defined objects.
	- **Overriding Prototype Methods**: In JavaScript, you may override methods on the built-in object prototype chain.  
	- **Overriding Constructors**: You may override object constructors to change how objects are formed.
    - [List of symbols](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol)

---
# Sources
- [Web Browser Engineering](https://browser.engineering/)
- [Advanced JavaScript Cheat Sheet](https://zerotomastery.io/cheatsheets/javascript-cheatsheet-the-advanced-concepts/)
- [Understanding V8’s Bytecode](https://medium.com/dailyjs/understanding-v8s-bytecode-317d46c94775)
- [Professional JavaScript for Web Developers](https://www.amazon.com/Professional-JavaScript-Developers-Matt-Frisbie/dp/1119366445)
- [Eloquent JavaScript](https://eloquentjavascript.net/)
- [You Don't Know JS Yet](https://github.com/getify/You-Dont-Know-JS)
- [Compiler](https://en.wikipedia.org/wiki/Compiler)
- [How browsers work](https://web.dev/articles/howbrowserswork)
- [Web Browser Engineering](https://www.maxiferreira.com/blog/browser-engineering-1/)
