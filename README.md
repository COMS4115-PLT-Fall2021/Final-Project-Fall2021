# COMS W4115 Final Project: Optimization with Control Flow and Data Flow Analysis

## Course Summary

Course: COMS 4115 Programming Languages and Translators (Fall 2021)  
University: Columbia University.  
Instructor: Prof. Baishakhi Ray


## Logistics
* **Announcement Date:** Monday, November 29, 2021
* **Due Date:** Monday, December 20, 2021 by 11:59 PM.
* **Total Points:** 100

## Description



In recent years, there has been an increasing trend towards the incorporation of programs into a smart devices, such as smart phones (e.g., iPhone), wearable devices (e.g., iWatch) and VR devices (you must be excited about [*Metaverse*](https://en.wikipedia.org/wiki/Metaverse)!!). However, the amount of memory available in these devices are often limited, compared with your desktop/laptop, due to considerations such as space, weight, power consumption, or price. At the same time, there is an increasing desire to support more software in such devices, such as encryption software, speech or image processing software or even high-revolution video games. This makes it desirable to try to reduce the size of applications where possible. 

<p float="left">
  <img src="wearables.jpg" width="300" />
  <img src="vr.jpg" width="330" />
</p>

In this final project, we will explore the use of compiler techniques, specifically optimization techiques, to accomplish code reduction. We have learned control flow and data flow analysis in class, and as two main techniques used by the compilers to achieve optimizations, we will use these analyzing techniques to optimize and compact code. We have two options for you: 

1. Unreachable Function Identification and Elimination with **Control Flow Analysis**
2. Dead Instruction Identification and Elimination with **Data Flow Analysis**

These two tasks are designed to have the same difficulties. Based on your interests, you can choose either one of them as your final project.

### Important Notes

### You MUST choose one option as your final project, and you should ONLY choose one to activate your GitHub Classroom. During grading, if we find that the same GitHub id has repositories for both options, your final project will NOT be graded.

## Option-1: Unreachable Function Identification and Elimination with Control Flow Analysis

In class and in previous programming assignments, we learned about __control flow analysis__. In this assignment, we will use such analysis techniques to optimize code. Particularly in this task, you will **design a tool to remove the unnecessary functions that will never not be executed**.

As you might already notice in your software development practice, human developers tend to keep some unreachable functions in the source code for a long time. In some cases, developers tend to keep some unreachable helper functions in the program as a backup and sometimes they forget to remove all outdated functions. All these functions will be unreachable and not be executed by the machine, and your task is to identify such functions and remove them.   

We assume that every program will have an entry function (in C, this is typically `main`). Consider the following program:

```c++
 1. int add(int n, int m) {
 2.     return n + m;
 3. }
 4. int mult(int m, int n) {
 5.     int res = 0, t = 5, x = m * n;
 6.     while(res != x) {
 7.         res = res + m;
 8.     }
 9.     return res;
10. }
11. int fact(int n) {
12.     if (n == 0) return 1;
13.     else {
14.         return mult(n, fact(add(n, -1)));
15.     }
16. }
17. int main() {
18.     int n = 9, m = 8;
19.     print(mult(m, n));
20.     return 0;
21. }
```  

Here, the entry function is `main`. Among the other functions, `fact` and `add` are never executed when starting from `main`. Thus, our objective for this optimization is to remove such functions that are never used or considered *unreachable* or *dead* in the code.

To begin identifying unreachable functions, we need to extract and analyze a [call graph](https://en.wikipedia.org/wiki/Call_graph). A call graph is a graph that shows the relationships among direct function calls in a program. Every node of a call graph represents a function call, and an edge from a node `A` to a node `B` in the call graph indicates that function `B` is invoked by function `A`, *i.e.*, a function call to `B` is made inside of function `A`'s body. Note that analyzing indirect function calls through function pointers requires a more sophisticated analysis technique, so for simplicity, assume that all function calls are direct function calls.

Once we identify all functions that are never called starting from `main`, our objective is to remove those from the code.

### Sub-task 1: Identification of Dead Functions
In this task, you will first implement the `getCallGraph` to extract the call graph. This function takes a vector of `Function *` containing all the functions in the current Module, and you need to create a hash map (a common way to store graph structure) for the call graph. You will then implement `getDeadFunctions` function and return a vector of `Function *` denoting the unreachable/dead functions (_i.e._, functions that are not reachable from `main`). This function takes in a `vector<Function *>` containing all functions in the module, `map<Function *, vector<Function *>>` containing the call graph, and a `Function *` indication the pointer to the entry function.

### Sub-task 2: Removal of Dead Functions
Once you find all the dead functions, you will implement the `removeDeadFunctions` function to actually perform the removal of all these dead functions. Note that our definition of "dead" _does NOT mean_ a function is not called from anywhere in the code. It simply refers to the fact that a function is not reachable from `main`. Thus, while you are removing a dead function, make sure that you properly update any possible call sites of that function. Given this hint, you will probably need to do some research to find suitable APIs to accomplish this task.

## Option-2: Dead Instruction Identification and Elimination with Data Flow Analysis

In compiler theory, dead instructions elimination is a compiler optimization to remove code which does not affect the program results. Removing such instructions has several benefits: it shrinks program size, an important consideration in some contexts, and it allows the running program to avoid executing irrelevant operations, which reduces its running time. It can also enable further optimizations by simplifying program structure. Dead instruction includes many types, but for this part of the assignment, we will do more micro-level optimization. In particular, we will eliminate **any instructions that define variables that are never used later**; we refer to such instructions as *dead* instructions. Consider the following IR code:

```ll
 1. define dso_local i32 @mult(i32 %m, i32 %n) #0 {
 2. entry:
 3.   %add = add nsw i32 %m, %n
 4.   %mul = mul nsw i32 %m, %n
 5.   br label %while.cond
 6. while.cond:               
 7.   %res.0 = phi i32 [ 0, %entry ], [ %add1, %while.body ]
 8.   %cmp = icmp ne i32 %res.0, %mul
 9.   br i1 %cmp, label %while.body, label %while.end
10. while.body:                                      
11.   %add1 = add nsw i32 %res.0, %m
12.   br label %while.cond
13. while.end:
14.   ret i32 %res.0
15. }
``` 
Note that, in the code above, there is no control path that uses `%add`. Thus, the instruction that defined `%add` (line 3) does not have any impact on the rest of the code. This instruction is an example of a dead instruction, and we can safely remove it from our IR. To analyze whether a defined variable is used later in the code, you will be asked to implement a function to do the liveness analysis first.

### Sub-task 1: Liveness Analysis
As you have already learned about liveness analysis, there are data flow equations that you can solve to determine liveness of variables at every node. You will now have the opportunity to solve these data flow equations by implementing the algorithm! Because solving data flow equations is an iterative process, we have provided for you the skeleton code for the data flow iteration. Inside the while loop, you will need to solve the data flow equations and update the LIVE_IN and LIVE_OUT sets after visiting every instruction. The loop is controlled by a Boolean variable called possibleToUpdate. Modify this variable as necessary (i.e., set this variable to "false" when you determine that your algorithm has converged). 

### Sub-task 2: Identification of Dead Instructions
Once we successfully extract data flow variables (_i.e._, `DEF`, `USE`, `LIVE_IN`, and `LIVE_OUT`), we call the `VariableLivenessUtil::removeDeadInstructions` function to check for and remove any dead instructions. Your task is to decide whether an instruction should or should not be removed.

You will need to implement the `VariableLivenessUtil::isDeadInstruction` function, which decides whether an instruction should or should not be removed. You do not have to actually remove the instruction; we have already done that for you, as indicated above. Please read through the comments in the code for further instructions and hints.   


## Start Your Final Projects!
Activate Option-1: https://classroom.github.com/a/dP45gKTb

Activate Option-2: https://classroom.github.com/a/wK3--iKB


