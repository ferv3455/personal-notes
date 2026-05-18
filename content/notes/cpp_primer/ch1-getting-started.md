---
title: Chapter 1 Getting Started
weight: 1
type: docs
---

## 1.1 Writing a Simple C++ Program

- A function definition has four elements: return type, function name, parameter list, and function body.
- `main` is required to have return type `int`, which is a status indicator (0 for success, nonzero values for error types defined by the system, e.g. -1).
  - The value returned from `main` is accessed in a system-dependent manner: `echo %ERRORLEVEL%` (Windows) or `echo $?` (UNIX).
  - Return value -1 will be shown as 255 (unsigned char representation).

## 1.2 A First Look at Input/Output

- C++ relies on the standard library to provide IO, mostly from `iostream`.
  - Two fundamental **stream** types: `istream` for input and `ostream` for output.
  - Four IO objects: `cin`, `cout`, `cerr`, and `clog`. The system associates these objects with the window in which the program is executed.
- `#include` directive must be written on a single line and outside any function. The name inside the angle brackets refers to a header.
- The `<<` operator takes two operands - `ostream` and the value to be output - and returns the `ostream` object. In this way we can chain multiple output operations together. The `>>` operator is used for input and works similarly to `<<`.
- `std::endl` is a special value called a **manipulator**, which ends a line and flushes the buffer associated with the device.
  - During debugging, it should be ensured that the stream is always flushed to avoid leaving debugging information in the buffer.
- The **scope operator** `::` is used to explicitly specify the namespace. All names defined by the standard library are in the `std` namespace.

## 1.3 A Word about Comments

- ***Warning: an incorrect comment is worse than no comment at all!***
- Two kinds of comments in C++: single-line (`//`, ends with a newline) and multi-line (delimiters `/*` and `*/`).
  - For multi-line comments, it is a good idea to begin each line with a `*` to indicate the comment's extent.
  - One comment pair cannot appear inside another. The compiler error messages may be confusing.
  - **The best way to comment a block of code during debugging is to insert single-line comments at the beginning of each line.**

```cpp
#include <iostream> 
/*
 *  Simple main function:
 *  Read two numbers and write their sum 
 */
int main()
{
    // prompt user to enter two numbers
    std::cout << "Enter two numbers:" << std::endl; 
    int v1 = 0, v2 = 0;   // variables to hold the input we read 
    std::cin >> v1 >> v2; // read input
    std::cout << "The sum of " << v1 << " and " << v2
              << " is " << v1 + v2 << std::endl;
    return 0;
}
```

- **The compiler reads from left to right to decide whether a delimiter is part of a string.** When a double-quote appears first, the delimiter belongs to the string. When a delimiter appears first, the double-quote belongs to the comment.

```cpp
std::cout << "/*";                // legal: /*
std::cout << "*/";                // legal: */
// std::cout << /* "*/" */;       // illegal
std::cout << /* "*/" /* "/*" */;  // legal: /*
```


## 1.4 Flow of Control

- `for (init-statement; condition; expression) statement` and `while (condition) statement` are used for loops with definite and indefinite iteration.
  - A block is a sequence of zero or more statements enclosed by curly braces. It can be used as a statement.
- `if` is used for conditional execution.
- Read until end-of-file: `while (std::cin >> value)`.
  - When we use an `istream` as a condition, the state of the stream will be tested. It becomes invalid when the end-of-file is reached or when an input operation fails (incorrect format).
    - Incorrect format: the input will be processed with best effort. For example, if the input is `1.2` for integers, the stream will read `1` and leave `.` in the input buffer.
  - Entering EOF from keyboard: Ctrl+Z (Windows) / Ctrl+D (UNIX) + Return/Enter.
- Compilation errors: syntax errors, type errors, declaration errors. Correct errors in the sequence they are reported: a single error can have a cascading effect. Recompile after each fix.

## 1.5 Introducing Classes

- Every class defines a type. The type name is the same as the class name. The author of the class defines all the actions that can be performed on objects of that class.
- Use the dot operator and the call operator to access and call a member function.
