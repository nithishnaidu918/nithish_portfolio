---
title: "Python Journey #1 - Python Basics"
description: "Learning Python fundamentals including comments, variables, strings, numbers, arithmetic, constants, and the Zen of Python."
date: "2026-09-11"
category: "Python"
tags: ["Python", "Basics", "Variables", "Strings", "Numbers"]
---

# Python Journey #1 - Python Basics

My first Python learning session covering the fundamental building blocks of Python.



```python
## 1. Comments

Comments are notes in code that Python does not execute.
# This is a comment
name = "Nithish"  # This is also a comment



## 2. Variables

A variable is a name that refers to a value.

name = "Nithish"
age = 21

print(name)
print(age)
Variables can be changed:
age = 20
age = 21
Python is case-sensitive, so name and Name are different variables.
Use snake_case for variable names:
first_name = "Nithish"
user_age = 21


##3. Strings

A string is text.

name = "Nithish"
message = 'Hello World'

Both single and double quotes can be used.

print("Hello World")




##4. String Methods

Python provides methods for working with strings.

name = "nithish"

print(name.title())
print(name.upper())
print(name.lower())

Output:

Nithish
NITHISH
nithish

.strip() removes whitespace around a string:

name = "  Nithish  "

print(name.strip())





##5. f-Strings

f-Strings make it easy to insert variables into strings.

name = "Nithish"
age = 21

message = f"My name is {name} and I am {age} years old."

print(message)

Output:

My name is Nithish and I am 21 years old.






##6. Numbers
Integer

Whole numbers are called integers (int).

age = 21
year = 2026
Float

Numbers with decimal values are called floats (float).

price = 99.99
temperature = 36.5






##7. Basic Arithmetic

Python can perform mathematical operations.

5 + 3    # Addition
5 - 3    # Subtraction
5 * 3    # Multiplication
5 / 2    # Division
2 ** 3   # Exponent
5 // 2   # Floor division
5 % 2    # Remainder

Results:

8
2
15
2.5
8
2
1





##8. Order of Operations

Python follows normal mathematical precedence.

2 + 3 * 4

Result:

14

Parentheses can change the order:

(2 + 3) * 4

Result:

20





##9. Underscores in Numbers

Underscores make large numbers easier to read.

population = 1_400_000_000

Python treats it the same as:

population = 1400000000




##10. Constants

Python does not have a special constant keyword.
By convention, uppercase names are used for values that should not change.

PI = 3.14159
MAX_USERS = 100

This is a convention, not a restriction.





##11. The Zen of Python

Python has a collection of principles called the Zen of Python.

Run:

import this

Some important ideas are:

Beautiful is better than ugly.
Explicit is better than implicit.
Simple is better than complex.
Readability counts.

The main idea is:
Write simple, clear, and readable Python code.
