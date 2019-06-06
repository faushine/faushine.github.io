---
layout: post
title: "07-Separate compilation"
date:   2019-06-06
author: Faushine
tags: 
- C++
---
# Separate compilation

Recall: 
- declaration: specifies existence eg.type
  
  ```c++
  struct Node;
  void print(std::ostream &out, const Node &n);
  extern int MAX;
  ```

- definition: variables(and functions) have space allocated; fll type info is known
  
  ```c++
  eg. struct nODE {
      int value;
      Node *next;
  }
  ```

```c++
void print(std::ostream &out, const node &n){
    out<<'('<< n.value << "," << hex << n.next<<')';
}
int MAX=10;
```

## module: collection of data, types, functions

- ideally serves one main purpose (Single Responsibility Principle)

- a module is made up of:

    - interfaces (*.h, *.hpp) contain type definitions (maybe declarations, but eventually need defn) global variable and function declarations

    - implementations(*.cpp, *.cc) contains function definitions and global variable definitions

- by using "separate compilation, only recompile what you have to and "link" the pieces

- separate compilation implicates using the "-c" flag when compiling an implementation file (.cc) to create the corresponding object file (.o) and then linking all .o files together to create the executable.

    &rarr; recompile only what changed, then relink everything.

- in order to automate this, need to find a way to figure out dependencies and what changed

## make

- by default, make looks for a file called "makefile" or "Makefile"

- the file lists all dependencies, and what command (or commands) to run

    ```bash
    <target> : <dependencies>
        \t command 1
        \t command 2
    clean:
        rm *.o a.out

    make clean
    ```

- vec-example 2

    CXX &larr; makes's variable to define c++ compiler

    CXXFLAGS &larr; makes var for compiler options

    make knows how to create a .o file from a .cc without you specifying, &rarr; but still listing dependencies

    stop comp

- can't have multiple definitions (see example 3)

- use "include guards" to prevent this

    ```c++
    #ifndef <randomname>
    #define <randomname>

    ...

    #endif

    #ifndef VEC_H
    #define VEC_H

    ...

    #endif
    ```

