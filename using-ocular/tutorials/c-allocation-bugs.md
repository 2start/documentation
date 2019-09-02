# Identifying Incorrect or Zero Memory Allocation Bugs in C

A common heap related bug is arithmetic operations that are performend on parameter of memory allocation functions, such as malloc(). In certain cases, the arithmetic operations can lead to integer overflow (resulting in less memory being allocated) or the arithmetic operation computes to zero. These conditions may lead to exploitable vulnerabilities. This tutorial uses ShiftLeft Ocular to determine if such a condition exists, using three allocation scenarios.

## allocation.c

``
#include <stdio.h>
#include <stdlib.h>

int getNumber() {
    int number = atoi("8");
    number = number + 10;
    return number;
}

void *scenario1(int x) {
    void *p = malloc(x);
    return p;
}

void *scenario2(int y) {
    int z = 10;
    void *p = malloc(y * z);
    return p;
}

void *scenario3() {
    int a = getNumber();
    void *p = malloc(a);                                                                                                                                                                      
    return p;
}
``

## Analysis

First, identify the call sites of malloc in the code. There are three call sites in this example

```
cpg.method.callOut.name("malloc").map(x => (x.location.filename, x.lineNumber.get)).l

List[(String, Integer)] = List(
  ("../../Projects/tarpitc/src/alloc/allocation.c", 23),
  ("../../Projects/tarpitc/src/alloc/allocation.c", 17),
  ("../../Projects/tarpitc/src/alloc/allocation.c", 11)
)
```

## atoi Case 

The following query examines dataflow from return of atoi to argument of malloc, at all malloc call sites:

```scala
val sink = cpg.method.callOut.name("malloc").argument
var source = cpg.method.name("atoi").methodReturn

sink.reachableBy(source).flows.p 
```

The query returns

```
 _______________________________________________________________________________________________
 | tracked    | lineNumber| method               | file                                         |
 |==============================================================================================|
 | ret        | N/A       | atoi                 | N/A                                          |
 | atoi("8")  | 5         | getNumber            | ../../Projects/tarpitc/src/alloc/allocation.c|
 | p2         | N/A       | <operator>.assignment| N/A                                          |
 | p1         | N/A       | <operator>.assignment| N/A                                          |
 | number     | 5         | getNumber            | ../../Projects/tarpitc/src/alloc/allocation.c|
 | number     | 6         | getNumber            | ../../Projects/tarpitc/src/alloc/allocation.c|
 | p1         | N/A       | <operator>.addition  | N/A                                          |
 | ret        | N/A       | <operator>.addition  | N/A                                          |
 | number + 10| 6         | getNumber            | ../../Projects/tarpitc/src/alloc/allocation.c|
 | p2         | N/A       | <operator>.assignment| N/A                                          |
 | p1         | N/A       | <operator>.assignment| N/A                                          |
 | number     | 6         | getNumber            | ../../Projects/tarpitc/src/alloc/allocation.c|
 | number     | 7         | getNumber            | ../../Projects/tarpitc/src/alloc/allocation.c|
 | ret        | 4         | getNumber            | ../../Projects/tarpitc/src/alloc/allocation.c|
 | getNumber()| 23        | scenario3            | ../../Projects/tarpitc/src/alloc/allocation.c|
 | p2         | N/A       | <operator>.assignment| N/A                                          |
 | p1         | N/A       | <operator>.assignment| N/A                                          |
 | a          | 23        | scenario3            | ../../Projects/tarpitc/src/alloc/allocation.c|
 | a          | 24        | scenario3            | ../../Projects/tarpitc/src/alloc/allocation.c|
```

To actually check if the flow passed through some arithmetic operations along the way, use the query

```scala
sink.reachableBy(source).flows.passes(".*(multiplication|addition).*").p
```

The query returns the same flow above if the condition is true.


## Arithmetic Operation

The following query looks for a parameter of some function to argument of malloc, at all malloc call sites while passing through an arithmetic operation

```scala
val sink = cpg.method.callOut.name("malloc").argument
var source = cpg.method.name(".*scenario.*").parameter


ocular> sink.reachableBy(source).flows.passes(".*multiplication.*").p 
```

The query returns 

```
  _______________________________________________________________________________________________
 | tracked| lineNumber| method                   | file                                         |
 |==============================================================================================|
 | y      | 15        | scenario2                | ../../Projects/tarpitc/src/alloc/allocation.c|
 | y      | 17        | scenario2                | ../../Projects/tarpitc/src/alloc/allocation.c|
 | p1     | N/A       | <operator>.multiplication| N/A                                          |
 | ret    | N/A       | <operator>.multiplication| N/A                                          |
 | y * z  | 17        | scenario2                | ../../Projects/tarpitc/src/alloc/allocation.c|
```


To make the source more generic, examine all parameters of all functions using the query

```scala
var source = cpg.method.parameter
```


## Filtering Out Sink of Interest

This query defines a source by:

1. Examining all call sites of all methods.
2. Filtering out the methods having malloc (sink of interest) and any arithmetic oepration.
3. Making the source a return (methodReturn) of the actual methods of the call sites, which are `atoi` and `getnumber`. 
4. Find flows from the methods to the malloc call site argument as sink.      

```
val sink = cpg.method.callOut.name("malloc").argument
var source = cpg.method.callOut.nameNot(".*(<operator>|malloc).*").calledMethod.methodReturn 

sink.reachableBy(source).flows.p 
```
The query returns 

```
  _______________________________________________________________________________________________
 | tracked    | lineNumber| method               | file                                         |
 |==============================================================================================|
 | ret        | N/A       | atoi                 | N/A                                          |
 | atoi("8")  | 5         | getNumber            | ../../Projects/tarpitc/src/alloc/allocation.c|
 | p2         | N/A       | <operator>.assignment| N/A                                          |
 | p1         | N/A       | <operator>.assignment| N/A                                          |
 | number     | 5         | getNumber            | ../../Projects/tarpitc/src/alloc/allocation.c|
 | number     | 6         | getNumber            | ../../Projects/tarpitc/src/alloc/allocation.c|
 | p1         | N/A       | <operator>.addition  | N/A                                          |
 | ret        | N/A       | <operator>.addition  | N/A                                          |
 | number + 10| 6         | getNumber            | ../../Projects/tarpitc/src/alloc/allocation.c|
 | p2         | N/A       | <operator>.assignment| N/A                                          |
 | p1         | N/A       | <operator>.assignment| N/A                                          |
 | number     | 6         | getNumber            | ../../Projects/tarpitc/src/alloc/allocation.c|
 | number     | 7         | getNumber            | ../../Projects/tarpitc/src/alloc/allocation.c|
 | ret        | 4         | getNumber            | ../../Projects/tarpitc/src/alloc/allocation.c|
 | getNumber()| 22        | scenario3            | ../../Projects/tarpitc/src/alloc/allocation.c|
 | p2         | N/A       | <operator>.assignment| N/A                                          |
 | p1         | N/A       | <operator>.assignment| N/A                                          |
 | a          | 22        | scenario3            | ../../Projects/tarpitc/src/alloc/allocation.c|
 | a          | 23        | scenario3            | ../../Projects/tarpitc/src/alloc/allocation.c|

  _______________________________________________________________________________________________
 | tracked    | lineNumber| method               | file                                         |
 |==============================================================================================|
 | ret        | 4         | getNumber            | ../../Projects/tarpitc/src/alloc/allocation.c|
 | getNumber()| 22        | scenario3            | ../../Projects/tarpitc/src/alloc/allocation.c|
 | p2         | N/A       | <operator>.assignment| N/A                                          |
 | p1         | N/A       | <operator>.assignment| N/A                                          |
 | a          | 22        | scenario3            | ../../Projects/tarpitc/src/alloc/allocation.c|
 | a          | 23        | scenario3            | ../../Projects/tarpitc/src/alloc/allocation.c|
```

You can use passes and passesNot to check if operations were performed on the malloc argument.


## CFG Analysis

Use this query to examine a specific malloc call site (specifically line 23, obtained from the first query of listing all callsites) and then look at the CFG, listing expressions, linenumbers, etc. 

```
ocular> cpg.method.callOut.name("malloc").filter(_.lineNumber(23)).repeat(_.cfgPrev).emit().map( x=> (x.code, x.location.filename, x.lineNumber.get)).l 
res99: List[(String, String, Integer)] = List(
  ("a", "alloc/allocation.c", 23),
  ("p", "alloc/allocation.c", 23),
  ("a = getNumber()", "alloc/allocation.c", 22),
  ("getNumber()", "alloc/allocation.c", 22),
  ("a", "alloc/allocation.c", 22)
)
```
