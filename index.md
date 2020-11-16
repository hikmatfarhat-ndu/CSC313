---
layout: index
published: true
---

## Description

This course provides a detailed coverage of **standard data structures** with an emphasis on 
complexity analysis. Topics include: **Asymptotic analysis**, **vectors**, **linked lists**,
**stacks**, **queues**, **trees** and **balanced trees**, **hashing**, **priority queues** 
and **heaps**, **sorting**. Standard graph algorithms  such as DFS, BFS, shortest paths and minimum spanning trees are also covered.
We will be making heavy use of C++ so it is important you read the c++ review early before the course really takes off.

The best way to study for this course is to follow the sequence:
1. Read the the (mostly theoretical) **lecture slides (pdf)** to gain an understanding of the main concepts without the encumbrance of detailed code. The algorithms in these slides are written in  pseudo code to illustrate the different operations on data structures and algorithms.

2. Read the corresponding  section in the CODE menu. This contains an  **annotated C++ implementation**  of the data structures and algorithms used in that chapter.

3. Finally, the code below  is a repository containing the complete source code of the
 examples we discuss in class.


This [repository](https://github.com/NDU-CSC313/inclass) contains all the code that we will be discussing in class. 
__Note__: all header files are in the include directory.
To build the code create a build directory then cd to it and type

> cmake ..

This will create a **Visual C++ solution** containing multiple projects where each project 
is an example we discuss in class. Note that many of these projects use different 
portions of the **same** source file. Each example is "activated" by enabling a different preprocessor variable. If you select an example project you will see that only the 
relevant portion of the code is highlighted (i.e. "enabled") by defining (i.e. #define) 
a corresponding preprocessor variable and the rest is not.

If you are using Xcode on Mac then type

> cmake -G Xcode ..


