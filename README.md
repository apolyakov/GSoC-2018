## GSoC 2018 Final Evaluation
This blog post documents the work I did for GSoC'18 on the LLVM Compiler Infrastructure project.

## Title
[Re-implement lldb-mi to correctly use the LLDB public SB API.](http://llvm.org/OpenProjects.html#lldb-reimplement-lldb-mi)

## Introduction
First of all, let's take a look at some basics and definitions.  
* **LLDB** is a next generation, high-performance debugger. It is built as a set of reusable components which highly leverage existing libraries in the larger LLVM Project, such as the Clang expression parser and LLVM disassembler.  
* **The Machine Interface (MI) Driver** is a standalone executable that sits between a client IDE (a GUI debugger for example) and a debugging API (LLDB), translating MI commands into equivalent LLDB actions. It also listens to events from the debugger such as “hit a breakpoint” and translates the event into an appropriate MI response for the client to interpret. This allows IDEs that would normally drive the GNU Debugger (GDB), or other back ends that understand the MI commands, to work with LLDB, with very similar functionality. It also worths noting that MI is a **text interface**.

In our case, the **lldb-mi** is the MI Driver and **LLDB** is used as a back end for debugging.

## Description
