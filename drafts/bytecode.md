---
title: Getting low-level
date: 2019-04-28
tags: Rabbitholing, Bytecode, V8, Engines, Java, JavaScript
---

## Down the compilation rabbit hole

Inspired by [this doc](https://docs.google.com/document/d/1hWb-lQW4NSG9yRpyyiAA_9Ktytd5lypLnVLhPX9vamE/edit#) about the high cost of array destructuring in Chrome's V8 engine, I did some rabbitholing on what actually happens to source code on its way to execution.

Some of the questions I explored include:

* What is bytecode? How do you read its syntax?
* How is JavaScript executed by the V8 engine?
* How are other languages compiled and executed?

## Tunnels followed

* [Bytecode (Wikipedia)](https://en.wikipedia.org/wiki/Bytecode)
* [Bytecode basics: A first look at the bytecodes of the Java virtual machine](https://www.javaworld.com/article/2077233/bytecode-basics.html)
* [Java bytecode (Wikipedia)](https://en.wikipedia.org/wiki/Java_bytecode)
* [Understanding V8’s bytecode](https://medium.com/dailyjs/understanding-v8s-bytecode-317d46c94775)
* [JavaScript engines - how do they even? (JSConf EU 2017)](https://www.youtube.com/watch?v=p-iiEDtpy6I)

## Rabbits found

### Bytecode

Bytecode is an instruction set for a virtual machine (VM). An intermediate format between source code and machine code, bytecode is often compiled from source code and then passed to the VM. It's so named because each instruction consists of a one-byte **opcode** (a code that specifies the operation to be performed) followed by any **operands** necessary for that operation.

The purposes of bytecode are to:
* Ease interpretation
* Allow code to run cross-platform, thanks to the VM

When Java source code is compiled, one bytecode stream is created for each method in a class.

An example Java bytecode stream:

    03 3b 84 00 01 1a 05 68 3b a7 ff f9

The same bytecode disassembled:

    0:  iconst_0      // 03
    1:  istore_0      // 3b
    2:  iinc 0, 1     // 84 00 01
    3:  iload_0       // 1a
    4:  iconst_2      // 05
    5:  imul          // 68
    6:  istore_0      // 3b
    7:  goto -7       // a7 ff f9

Bytecode execution in a JVM is based around pushing and popping values onto/from a stack. The opcodes indicate what type of variable they are pushing onto the stack. For example, `iload` pushes an integer, whereas `fload` pushes a float. This makes it clear how to interpret the bytes that follow the opcode.

### How do JavaScript engines work?

A difficulty of compiling JavaScript is that as a dynamically typed language, it gives little information to the compiler about the variables it's passing around. Without this information, it's hard to generate fast machine code.

The trick to making a JavaScript engine fast is just-in-time (JIT) compilation. Unlike a language like C++, which is compiled ahead of time to an executable that is then run, JavaScript must be compiled and executed at the same time. As it executes, feedback from the running code is used to recompile the code to speed up its performance.

So modern JavaScript engines have not one but two compilers:
* Baseline compiler
    * Called **Ignition** in V8
    * Normal slower one
* Optimizing compiler
    * Called **Turbofan** in V8
    * Recompiles "hot" functions (those that you're using a lot and are therefore worth speeding up).
    * As it runs a function a few times, it collects info about the types being passed in. When it decides it's hot, it uses the gathered information to recompile the code.
    * When it recompiles a function, it assumes that the patterns it has noticed in the types being passed into a function will still hold true.
    * If a type is passed into the function that violates these assumptions, it will be de-optimized and returned to the baseline compiler.
