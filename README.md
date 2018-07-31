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
The lldb-mi should provide an implementation for all the MI commands, but current support is incomplete and, more importantly, some commands are implemented using wrong abstraction layer. Instead of asking the LLDB to execute some command _(e.g. SBCommandInterpreter::HandleCommand)_ and then scraping and processing its textual output, it should be using the methods and data structures provided by the public SB API.  

My task as a GSoC student was to get rid of using _HandleCommand_, reimplement lldb-mi commands to correctly use public SB API, add new API if needed. It will reduce maintenance effort for the project since the public API is guaranteed to remain stable.

## Team
* Student(me)
  * [Alexander Polyakov](https://reviews.llvm.org/p/apolyakov)
* Mentor
  * [Adrian Prantl](https://reviews.llvm.org/p/aprantl/) 

The LLVM Community was also very helpful in reviewing patches on [reviews.llvm.org](reviews.llvm.org) and responding to my queries via email. Amongst the very helpful community members are: [Greg Clayton](https://reviews.llvm.org/p/clayborg/), [Pavel Labath](https://reviews.llvm.org/p/labath/), [Jim Ingham](https://reviews.llvm.org/p/jingham/) and [Stella Stamenova](https://reviews.llvm.org/p/stella.stamenova/).

## What I did
In my proposal I've noted **eleven** commands and methods which use _HandleCommand_ and should be reimplemented. They are:
1. exec-continue
1. exec-next
1. exec-step
1. exec-next-instruction
1. exec-step-instruction
1. exec-finish
1. exec-interrupt
1. symbol-list-lines
1. data-info-line
1. target-select
1. CMICmnLLDBDebuggerHandleEvents::HandleProcessEventStateSuspended

After been accepted by LLVM Compiler Infrastructure as a GSoC'18 student I first worked on improvement of _break-insert_ command to get more familiar with the code base and a review process. The main task was to add support of pending breakpoints. For example:
```
# start lldb-mi session without debug target
./lldb-mi
# create a breakpoint on function with name hit_me, we don't now what is the hit_me,
# but we want to stop on this function if the target has it
-break-insert hit_me
# choose a target
-file-exec-and-symbols some_binary_with_hit_me
# run target
-exec-run
```
The pending breakpoints support means that if we execute above mentioned sequence of commands we will hit the breakpoint on _hit_me_ and stop. Also, if selected binary doesn't have _hit_me_ function we will not stop. You can see [the commit](https://github.com/llvm-mirror/lldb/commit/26acfa38acd28a4fe345ef9d1268c8959f24c319).
