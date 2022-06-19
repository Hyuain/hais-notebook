---
title: Topics From Chromium Blogs
date: 2022-06-16 10:30:50
tags:
  - Chromium
  - Blogs
categories:
  - [Memo]
---

Read some articles in chromium blogs or relative wikis, and take these memos.

<!-- more -->

# Architecture

## Multi-process Architecture

>  [Multi-process Architecture](https://blog.chromium.org/2008/09/multi-process-architecture.html) - Thursday, September 11, 2008
>
>  Overview: One browser process + many renderer processes (roughly one for each tab) + plugin processes (one for each plugin)

1. Three different types of processes: browser, renderers, and plug-ins.
   1. *Browser*. There's only **one** browser process, which manages the tabs, windows, and "chrome" of the browser.  This process also handles all interactions with the disk, network, user input, and display, but it makes no attempt to parse or render any content from the web.
   2. *Renderers*. The browser process creates **many** renderer processes, each responsible for rendering web pages.  The renderer processes contain all the complex logic for handling HTML, JavaScript, CSS, images, and so on.  We achieve this using the open source WebKit rendering engine, which is also used by Apple's Safari web browser.  Each renderer process is run in a sandbox, which means it has almost **no direct access to your disk, network, or display**.  **All interactions with web apps, including user input events and screen painting, must go through the browser process.**  This lets the browser process monitor the renderers for suspicious activity, killing them if it suspects an exploit has occurred.
   3. *Plug-ins*. The browser process also creates **one process for each type of plug-in that is in use**, such as Flash, Quicktime, or Adobe Reader.  These processes just contain the plug-ins themselves, along with some glue code to let them interact with the browser and renderers.
2. Create browser process -> Create one renderer process for each instance of a web site you visit.
3. You can think of this as using **a different process for each tab in the browser**, but **allowing two tabs to share a process** if they are related to each other and are showing the same site.

# Security



# JavaScript

## JIT: Just-in-time compilation

> V8 uses [TurboFan JIT](https://v8.dev/blog/turbofan-jit).
>
> Overview:
>
> 1. JIT = AOT + interpretation
> 2. Source code -> bytecode -> machine code
> 3. Generating better machine code increase initial delay 

1. Introduction:
   1. JIT is a way of executing computer code that involves compilation **during execution of a program** (at **run time**) rather than before execution.
   2. This may consist of [*source code*](https://en.wikipedia.org/wiki/Source_code) translation but is more commonly [*bytecode*](https://en.wikipedia.org/wiki/Bytecode) translation to [*machine code*](https://en.wikipedia.org/wiki/Machine_code), which is then executed directly.
   3. JIT compilation is a **combination** of the two traditional approaches to translation to machine code - [*ahead-of-time compilation*](https://en.wikipedia.org/wiki/Ahead-of-time_compilation) (*AOT*), and [*interpretation*](https://en.wikipedia.org/wiki/Interpreter_(computing)) - and combines some advantages and drawbacks of both.
   4. Roughly, JIT compilation combines **the speed of compiled code** with **the flexibility of interpretation**, with **the overhead of an interpreter** and **the additional overhead of compiling and [linking](https://en.wikipedia.org/wiki/Linker_(computing))** (not just interpreting).
2. Design:
   1. Source code is translated to an intermediate representation known as *bytecode*.
   2. *Bytecode* is not the machine code for any particular computer, and may be portable among computer architectures. The *bytecode* may then be **interpreted** by, or run on **a virtual machine**. 
   3. *Traditional interpreted virtual machine* vs. *JIT*
      1. *The traditional interpreted virtual machine*: simply **interpret the bytecode**, generally with much lower performance.
      2. *The JIT compiler*: reads the bytecodes in many sections (or in full, rarely) and **compiles** them dynamically **into machine code** so the program can run faster. This can be done per-file, per-function or even on any arbitrary code fragment; the code can be compiled when it is about to be executed (hence the name "just-in-time"), and then **cached and reused** later without needing to be recompiled.
   4. Because a JIT must render and execute a native binary image at runtime, true machine-code JITs necessitate platforms that allow for data to be executed at runtime, making using such JITs on a [*Harvard architecture*](https://en.wikipedia.org/wiki/Harvard_architecture)-based machine impossible; the same can be said for certain operating systems and virtual machines as well.
3. *Startup time delay* or *warm-up time*: the time taken to load and compile the bytecode. The more optimization JIT performs, the better the code it will generate, but the initial delay will also increase. 

## Ignition Interpreter

> See [this blog](https://v8.dev/blog/ignition-interpreter).

1. Code is initially compiled by *baseline compiler*, which can generated non-optimized machine code quickly.
2. The compiled code is analyzed during runtime and optionally re-compiled dynamically with a more advanced optimizing compiler for peak performance.
3. JITed machine code can consume a significant amount of memory, so V8 uses Ignition to compile JavaScript functions to a concise bytecode. This bytecode is then executed by a interpreter.

## Garbage Collection

> See [this blog](https://v8.dev/blog/free-garbage-collection).
>
> Overview: Two ways to find garbage: tracing garbage collection & reference counting

There are several kinds of garbage collection:

1. *Tracing garbage collection*
   1. Based on *reachability* (An object is **reachable** if it is **referred to by a root**, or is **referred to by a reachable object**; that is, if it can be reached from the roots by following references.)
   2. *Object*: A contiguous block of memory forming a single logical structure, objects are the units of allocation, deallocation, etc.
   3. If an object is **not reachable**, there is no way the mutator could ever access it, and therefore it **cannot be live**.
   4. In each collection cycle, some or all of the objects are *condemned* (those objects are candidates for recycling), and the graph is traced to find which of the condemned objects are reachable. Those that were not reachable may be reclaimed.
   5. Typical scenes:
      1. *Copying garbage collection*: relocate reachable objects, and then reclaim objects are left behind.
      2. *Generational garbage collection*: see next part.
2. *Reference counting*
   1. Keeping a count in each object, of how many references there are to the object.
   2. The reference count is incremented for each new reference, and is decremented if a reference is overwritten, or if the referring object is recycled.
   3. Advantages:
      1. Can reclaim objects promptly.
      2. Pauses are typically fairly short, but removing a single reference may cause the recycling of a large number of objects at once.
      3. It can be implemented without any support from the language or compiler.

   4. Problems:
      1. **The reference count filed size is limited**, system will break down if there are too many references.
      2. **Performance penalty**, since the count need to be maintained after every modification of pointers.
      3. **Object will become larger** to store the reference count.
      4. **Cyclic data structure** problems.
   5. Typical scenes:
      1. *Distributed garbage collection*: The garbage collection in a system where objects might not reside in the same address space or even on the same machine. Because the costs of synchronization and communication between processes are particularly high for a *tracing garbage collector*, so other techniques, including *weighted reference counting*, are commonly used instead.


### Generational Garbage Collection

> See [this website](https://www.memorymanagement.org/glossary/g.html#term-generational-garbage-collection).
>
> Overview: Based on "young is more likely to die than old"

1. Generational garbage collection is *tracing garbage collection* that makes use of the *generational hypothesis*.
2. *Generational hypothesis (also say Infant mortality)*: in most cases, young objects are much more likely to die than old objects.
3. Objects are gathered together **in generations**. New objects are allocated in the *youngest* or *nursery* generation, and promoted to (or become) *older* generations if they survive. **Objects in older generations are condemned less frequently**, saving CPU time.

### V8's Garbage Collection Engine

> V8 uses a generational garbage collector. See [this article](https://v8.dev/blog/trash-talk).
>
> Overview:
>
> 1. Split into young (nursery and intermediate) and old.
> 2. Major GC collects garbage in the entire heap, including marking, sweeping and copaction.
> 3. Minor GC is responsible for the young generation, using semi-space allocation strategy.
> 4. Orinoco is a project try to improve GC performance.

#### Generational Layout

Split the JavaScript heap into:

1. **A small young generation** for newly allocated objects (split further into *nursery* and *intermediate* sub-generations).
2.  **A large old generation** for long living objects.

Objects are first allocated into the *nursery*. If they survive the next GC, they remain in the young generation but are considered *intermediate*. If they survive yet another GC, they are moved into the old generation.

![Generational layout of GC in V8](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Memo-ChromiumBlogs-GC-1.svg)

#### Major GC (Full Mark-Compact)

The Major GC collects garbage from the entire heap.

1. *Marking*: Marking reachable objects from the root.
2. *Sweeping*: Add the gaps in memory left by dead objects into a *free-list*. In the future when we want to allocate memory, we just look at the free-list and find an appropriately sized chunk of memory.
3. *Compaction*: if needed, evacuate/compact some pages, to make use of the small and scattered gaps within the memory left behind by dead objects.

#### Minor GC (Scavenger)

The Minor GC (Scavenger) collects garbage in the young generation.

1. It uses a *semi-space allocation* strategy, where *nursery* object are initially allocated in the young generation’s active semi-space.
2. Once that semi-space becomes full, a *scavenge* operation (also use marking to check reachability) will move live objects to other semi-space, and become *intermediate*. If a object is already *intermediate*, it will be promoted to the old generation.
3. The new semi-spaces becomes active, and all the remaining objects in old semi-space are discarded.

Unlike Major GC, in scavenging we actually do these three steps - marking, evacuating, and pointer-updating - all interleaved, rather than in distinct phases.

#### Orinoco

Orinoco is the codename of the GC project and try to obtain better performance.

1. **Parallel**: *Main Thread* and *Helper Threads* do a roughly equal amount of work at the same time.

![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Memo-ChromiumBlogs-GC-2.svg)

2. **Incremental**: The main thread does a small amount of work intermittently.

![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Memo-ChromiumBlogs-GC-3.svg)

3. **Concurrent**:  The main thread executes JavaScript constantly, and helper threads do GC work totally in the background.

![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Memo-ChromiumBlogs-GC-4.svg)

## RegExp

### Lookbehind assertions

> See [this blog](https://v8.dev/blog/regexp-lookbehind-assertions).
>
> Overview:
>
> 1. V8's lookbehind implementation supports patterns of arbitrary length (can use `*` `+`)
> 2. Some strange features because of the implementation

1. Two ways to implement lookbehind assertions:
   1. Perl: Requires fixed length, so cannot use `*` `+`. Because the regular expression engine can step back by that fixed length and match.
   2. .NET framework: Can match patterns of arbitrary length. It simply matches the lookbehind pattern backwards (reading characters against the normal read direction).
2. A capturing group with a *quantifier captures* the last match. Usually, that is the right-most match. But inside a lookbehind assertion, we match from right to left, therefore the left-most match is captured:

```js
/h(?=\w)/.exec('hodor');  // ['h']
/h(?=(\w))/.exec('hodor');  // ['h', 'o']
/h(?=(\w){2})/.exec('hodor');  // ['h', 'd']
/h(?=(\w)+)/.exec('hodor');  // ['h', 'r']

/(?<=(\w){2})r/.exec('hodor'); // ['r', 'd']
/(?<=(\w)+)r/.exec('hodor'); // ['r', 'h']
```

3. A capturing group can be referenced via back reference after it has been captured. Usually, the back reference has to be to the right of the capture group. Otherwise, it would match the empty string, as nothing has been captured yet. However, inside a lookbehind assertion, the match direction is reversed:

```js
/(?<=(o)d\1)r/.exec('hodor'); // null
/(?<=\1d(o))r/.exec('hodor'); // ['r', 'o']
```

## Object Iteration

### ECMA Specification

> See [this document](https://tc39.es/ecma262/#sec-ordinaryownpropertykeys).

1. JavaScript objects mostly behave like dictionaries, with *string* keys and *arbitrary objects* as values.
2. The specification treat *integer-indexed properties* and *other properties* differently during *iteration*. Other than that, the different properties behave mostly the same.

### Fast Properties In V8

> See [this blog](https://v8.dev/blog/fast-properties).

#### Named Properties vs. Elements

![A basic JavaScript object looks like in memory](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Memo-ChromiumBlogs-OI-1.png)

1. Elements and properties are stored in two separate data structures which makes adding and accessing properties or elements more efficient for different usage patterns.
2. *Elements*:
   1. Most of time, V8 represents *elements* as *simple arrays* internally, since methods (like `pop` `slice` in `Array.prototype`) access properties in consecutive ranges.
   2. But sometimes switch to a sparse *dictionary-based* representation to save memory.
3. *Named properties*:
   1. They are stored in a similar way in a separate *array*.
   2. We cannot simply use the *key* to deduce their *position* within the properties array, we need some additional metadata - In V8, every JavaScript object has a *HiddenClass* associated.
   3. The *HiddenClass* stores information about the *shape* of an object, a *mapping* from property names to indices into the properties, and so on.

#### HiddenClasses and DescriptorArrays

![Overview of HiddenClass in V8](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Memo-ChromiumBlogs-OI-2.png)

1. The first field of any object on V8 heap points to a HiddenClass.
2. The third bit field of HiddenClass stores the number of properties, and a pointer to the *descriptor array*.
3. The *descriptor array* contains information about *named properties*, like *the name itself* and *the position* where value is stored. (there is no entry of *indexed properties* in the *descriptor array*)

4. Objects with the same structure (e.g. the same named properties in the same order) share the same HiddenClass.

![HiddenClass change when add new properties](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Memo-ChromiumBlogs-OI-3.png)

5. V8 creates a *transition tree* that links the HiddenClasses together.

![Transition tree](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Memo-ChromiumBlogs-OI-4.png)

6. If we create a **new object** that gets a different property added, in this case property `"d"`, V8 creates a **separate branch** for the new HiddenClasses.

![Transition tree with branches](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Memo-ChromiumBlogs-OI-5.png)

7. Adding *array-indexed properties* does **not** create new HiddenClasses.

#### The three different kinds of named properties

1. A simple object such as `{a: 1, b: 2}` can have various internal presentations in V8.
2. While JavaScript objects behave like simple *dictionaries* from outside, V8 tries to avoid dictionaries because they hamper certain optimizations such as *inline caches*.

##### 1. In-object Properties

In-object properties stored directly on the object themselves. These are the fastest properties available in V8.

![In-object and fast properties](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Memo-ChromiumBlogs-OI-6.png)

##### 2. Fast Properties

1. Fast properties are stored in the *linear properties store*, and can simply accessed by index. 
2. To get from the property name actual position in the properties store, we have to consult the *descriptor array* on *HiddenClass*.

##### 3. Slow Properties

![Object with slow properties](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Memo-ChromiumBlogs-OI-7.png)

1. Many properties get added and deleted from an object. They are slow properties.
2. An object with slow properties has a self-contained dictionary as a properties store. **All the properties meta information** is no longer stored in the descriptor array on the HiddenClass but **directly in the properties' dictionary**.
3. Properties can be added and removed without updating the HiddenClass.
4. *Inline caches* don't work with those dictionary properties.

#### Elements or array-indexed properties

There are several types of elements.

##### Packed or Holey Elements

To ways to get hole: delete an indexed element or just don't defined it (like `[1,,3]`).

```js
const o = ['a', 'b', 'c'];
console.log(o[1]);          // Prints 'b'.

delete o[1];                // Introduces a hole in the elements store.
console.log(o[1]);          // Prints 'undefined'; property 1 does not exist.
o.__proto__ = {1: 'B'};     // Define property 1 on the prototype.

console.log(o[0]);          // Prints 'a'.
console.log(o[1]);          // Prints 'B'.
console.log(o[2]);          // Prints 'c'.
console.log(o[3]);          // Prints undefined
```

![Holey elements example](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Memo-ChromiumBlogs-OI-8.png)

1. Given that element properties are self-contained (we don't store information about element on the HiddenClass), we need a special value (called `the_hole`) to mark properties that are not present.
2. This is crucial for the performance of Array functions. Because if we know that there are no holes (the elements store is packed), we can perform local operations without expensive lookups on the prototype chain.

#####  Fast or Dictionary Elements

1. Fast elements are simple *VM-internal arrays* where the *property index* maps to the in *index in the elements store*.
2. The simple presentation is rather wasteful for very large *sparse/holey arrays*. V8 creates a *dictionary* where we store a *key-value-descriptor triplets*.

```js
const sparseArray = [];
sparseArray[9999] = 'foo'; // Creates an array with dictionary elements.
```

3. V8 resorts to slow elements whenever you define an indexed properties with a custom descriptor.

```js
const array = [];
// 0 is slow element
Object.defineProperty(array, 0, {value: 'fixed' configurable: false});
console.log(array[0]);      // Prints 'fixed'.
array[0] = 'other value';   // Cannot override index 0.
console.log(array[0]);      // Still prints 'fixed'.
```

4. Array functions perform considerably slower on objects with slow elements.

##### Semi and Double Elements

1.  If you only store integers in an Array, a common use-case, the GC does not have to look at the array, as integers are directly encoded as so called *small integers (Smis)* in place.
2. Unlike Smis, floating point numbers are usually represented as full objects occupying several words. However, V8 stores raw doubles for pure double arrays to avoid memory and performance overhead.

```js
const a1 = [1,   2, 3];  // Smi Packed
const a2 = [1,    , 3];  // Smi Holey, a2[1] reads from the prototype
const b1 = [1.1, 2, 3];  // Double Packed
const b2 = [1.1,  , 3];  // Double Holey, b2[1] reads from the prototype
```

##### Special Elements

There are more types: TypedArrays, string wrappers, arguments objects.

##### The ElementsAccessor

![What happens when call array methods](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Memo-ChromiumBlogs-OI-9.png)
