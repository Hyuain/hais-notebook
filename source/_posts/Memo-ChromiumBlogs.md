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

1. Introduction:
   1. JIT is a way of executing computer code that involves compilation **during execution of a program** (at **run time**) rather than before execution.
   2. This may consist of [source code](https://en.wikipedia.org/wiki/Source_code) translation but is more commonly [bytecode](https://en.wikipedia.org/wiki/Bytecode) translation to [machine code](https://en.wikipedia.org/wiki/Machine_code), which is then executed directly.
   3. JIT compilation is a **combination** of the two traditional approaches to translation to machine code—[ahead-of-time compilation](https://en.wikipedia.org/wiki/Ahead-of-time_compilation) (**AOT**), and [**interpretation**](https://en.wikipedia.org/wiki/Interpreter_(computing))—and combines some advantages and drawbacks of both.
   4. Roughly, JIT compilation combines **the speed of compiled code** with **the flexibility of interpretation**, with **the overhead of an interpreter** and **the additional overhead of compiling and [linking](https://en.wikipedia.org/wiki/Linker_(computing))** (not just interpreting).
2. Design:
   1. Source code is translated to an intermediate representation known as **bytecode**.
   2. **Bytecode** is not the machine code for any particular computer, and may be portable among computer architectures. The **bytecode** may then be **interpreted** by, or run on **a virtual machine**. 
   3. Traditional **interpreted virtual machine** vs. **JIT**
      1. The traditional **interpreted virtual machine**: simply **interpret the bytecode**, generally with much lower performance.
      2. **The JIT compiler**: reads the bytecodes in many sections (or in full, rarely) and **compiles** them dynamically **into machine code** so the program can run faster. This can be done per-file, per-function or even on any arbitrary code fragment; the code can be compiled when it is about to be executed (hence the name "just-in-time"), and then **cached and reused** later without needing to be recompiled.
   4. Because a JIT must render and execute a native binary image at runtime, true machine-code JITs necessitate platforms that allow for data to be executed at runtime, making using such JITs on a [Harvard architecture](https://en.wikipedia.org/wiki/Harvard_architecture)-based machine impossible; the same can be said for certain operating systems and virtual machines as well.
3. Startup time delay or warm-up time: the time taken to load and compile the bytecode. The more optimization JIT performs, the better the code it will generate, but the initial delay will also increase. 

## Garbage Collection

> V8 uses a generational garbage collector with the JavaScript heap split into a small young generation for newly allocated objects and a large old generation for long living objects. see [this blog](https://v8.dev/blog/free-garbage-collection).

There are several kinds of garbage collection:

1. Tracing garbage collection
   1. Based on **reachability** (An object is *reachable* if it is referred to by a root, or is referred to by a reachable object; that is, if it can be reached from the roots by following references.
   2. If an object is **not reachable**, there is no way the mutator could ever access it, and therefore it **cannot be live**.
   3.  In each collection cycle, some or all of the objects are condemned (those objects are candidates for recycling), and the graph is traced to find which of the condemned objects are reachable. Those that were not reachable may be reclaimed.
   4. Typical strategies:
      1. *Copying garbage collection*: relocate reachable objects, and then reclaim objects are left behind.
      2. *Generational garbage collection*: see next part.
2. Reference counting
   1. Keeping a count in each object, of how many references there are to the object.
   2. The reference count is incremented for each new reference, and is decremented if a reference is overwritten, or if the referring object is recycled.
   3. Problems:
      1. 


### Generational Garbage Collection

> See [this website](https://www.memorymanagement.org/glossary/g.html#term-generational-garbage-collection).

1. Generational garbage collection is **tracing garbage collection** that makes use of the **generational hypothesis**.
2. **Objects** (object is a contiguous block of memory forming a single logical structure, objects are the units of allocation, deallocation, etc.) are **gathered** together **in generations**. New objects are allocated in the *youngest* or *nursery* generation, and [promoted](https://www.memorymanagement.org/glossary/p.html#term-promotion) to *older* generations if they survive. Objects in older generations are [condemned](https://www.memorymanagement.org/glossary/c.html#term-condemned-set) less frequently, saving CPU time.

It is typically rare for an object to refer to a younger object. Hence, objects in one generation typically have few [references](https://www.memorymanagement.org/glossary/r.html#term-reference) to objects in younger generations. This means that the [scanning](https://www.memorymanagement.org/glossary/s.html#term-scan) of old generations in the course of collecting younger generations can be done more efficiently by means of [remembered sets](https://www.memorymanagement.org/glossary/r.html#term-remembered-set).

In some purely functional languages (that is, without update), all references are backwards in time, in which case remembered sets are unnecessary.
