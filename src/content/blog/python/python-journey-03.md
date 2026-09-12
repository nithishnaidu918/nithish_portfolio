---
title: "Python Journey #3 - Loops, Numerical Lists, Slicing & Tuples"
description: "Learning for loops, range(), numerical lists, list slicing, copying lists, tuples, and Python code style."
date: "2026-09-12"
category: "Python"
tags: ["Python", "Lists", "Loops", "Range", "Slicing", "Tuples", "PEP8"]
---

# Python Journey #3 - Loops, Numerical Lists, Slicing & Tuples



```python
## 1. Working with Lists

Lists become especially useful when we want to perform the same operation on many elements.

magicians = ["alice", "david", "carolina"]

Instead of accessing every element separately:

print(magicians[0])
print(magicians[1])
print(magicians[2])

we can use a loop.






##2. Looping Through a List

Use a for loop to process each element.

magicians = ["alice", "david", "carolina"]

for magician in magicians:
    print(magician)

Output:

alice
david
carolina

The loop takes each element one at a time and stores it temporarily in magician.








##3. for Loop Structure
for variable in list:
    # code to execute

Example:

names = ["Nithish", "Rahul", "Arjun"]

for name in names:
    print(f"Hello {name}!")







##4. Multiple Statements Inside a Loop

A loop can contain multiple statements.

magicians = ["alice", "david", "carolina"]

for magician in magicians:
    print(f"{magician.title()}, that was a great trick!")
    print(f"I can't wait to see your next trick, {magician.title()}.\n")

All indented statements execute during every iteration.













##5. Indentation

Python uses indentation to define blocks of code.

for magician in magicians:
    print(magician)

The print() statement belongs to the loop because it is indented.

Important

Python convention:

4 spaces = one indentation level









##6. Indentation Errors

Incorrect:

for magician in magicians:
print(magician)

This causes:

IndentationError

Unnecessary indentation is also incorrect:

message = "Hello"

    print(message)
Important

After statements ending with :, the following block is normally indented.

Examples:

for ...
if ...
def ...
class ...












##7. The Colon :

A colon tells Python that an indented block is starting.

Incorrect:

for magician in magicians
    print(magician)

Correct:

for magician in magicians:
    print(magician)












##8. Numerical Lists with range()

range() is commonly used to generate sequences of numbers.

for number in range(1, 5):
    print(number)

Output:

1
2
3
4
Important

The ending value is not included.

range(1, 5)

produces:

1, 2, 3, 4










##9. Converting range() to a List

range() produces a range object.

Use list() to convert it into a list.

numbers = list(range(1, 6))

print(numbers)

Output:

[1, 2, 3, 4, 5]












##10. range() With a Step

Syntax:

range(start, stop, step)

Example:

numbers = list(range(2, 11, 2))

print(numbers)

Output:

[2, 4, 6, 8, 10]

Here:

start = 2
stop  = 11
step  = 2










##11. Creating a List of Squares

We can combine range() with calculations.

squares = []

for number in range(1, 6):
    square = number ** 2
    squares.append(square)

print(squares)

Output:

[1, 4, 9, 16, 25]
Important Pattern
result = []

for item in something:
    result.append(calculation)

Later, this can be written more concisely using a list comprehension.












##12. Basic Statistics

Python provides built-in functions for numerical lists.

numbers = [1, 2, 3, 4, 5]
Minimum
min(numbers)

Result:

1
Maximum
max(numbers)

Result:

5
Sum
sum(numbers)

Result:

15

These functions are frequently useful when working with data.










##13. List Slicing

A slice allows us to work with part of a list.

players = ["charles", "martina", "michael", "florence", "eli"]

Get the first three elements:

print(players[0:3])

Output:

['charles', 'martina', 'michael']
Important

The ending index is not included.

players[0:3]

means indexes:

0, 1, 2













##14. Omitting the Start Index
print(players[:3])

This means:

Start from the beginning and stop before index 3.

Output:

['charles', 'martina', 'michael']









##15. Omitting the End Index
print(players[2:])

This means:

Start at index 2 and continue to the end.

Output:

['michael', 'florence', 'eli']








##16. Copying a List with Slicing

Use [:] to create a separate copy.

my_foods = ["pizza", "falafel", "carrot cake"]

friend_foods = my_foods[:]

Now they are separate lists.

my_foods.append("ice cream")
friend_foods.append("cookies")

Each list can now be changed independently.

Important

This is different:

friend_foods = my_foods

Here, both variables refer to the same list.















##17. Tuples

A tuple is similar to a list but is immutable.

List:

dimensions = [200, 50]

Tuple:

dimensions = (200, 50)

Tuples normally use parentheses ().









##18. Accessing Tuple Elements

Tuples use indexing just like lists.

dimensions = (200, 50)

print(dimensions[0])

Output:

200











##19. Tuples Cannot Be Modified

Lists can be changed:

dimensions = [200, 50]

dimensions[0] = 250

Tuples cannot:

dimensions = (200, 50)

dimensions[0] = 250

This causes:

TypeError

because tuples are immutable.









##20. When to Use Tuples

Use a tuple when the values should not change.

Example:

screen_size = (1920, 1080)
List
colors = ["red", "blue", "green"]

Mutable.

Tuple
dimensions = (1920, 1080)

Immutable.








##21. Looping Through a Tuple

Tuples can also be used with for loops.

dimensions = (1920, 1080)

for dimension in dimensions:
    print(dimension)

Output:

1920
1080








##22. Reassigning a Tuple

You cannot change an individual tuple element:

dimensions[0] = 2500

But you can assign a completely new tuple to the variable:

dimensions = (1920, 1080)

dimensions = (2560, 1440)

print(dimensions)

Output:

(2560, 1440)

The original tuple was not modified. The variable now refers to a new tuple.









##23. Python Code Style

Python follows coding conventions described mainly by PEP 8.

The goal is:

Write code that is easy for humans to read.







##24. Variable Naming

Use lowercase names with underscores.

Good:

first_name = "Nithish"
user_age = 21

This style is called snake_case.






##25. Spaces Around Operators

Prefer:

age = 21
total = price + tax

Instead of:

age=21
total=price+tax

Readable spacing makes code easier to understand.








##26. Consistent Indentation

Use 4 spaces for indentation.

for name in names:
    print(name)

Avoid mixing tabs and spaces.








##27. Blank Lines

Use blank lines to separate logical sections.

name = "Nithish"
age = 21


print(name)
print(age)

Don't use excessive blank lines.







##28. Long Lists

Long lists can be written across multiple lines.

languages = [
    "Python",
    "Java",
    "C++",
    "JavaScript",
    "Go",
]

This improves readability.








##29. Useful Comments

Comments should explain something that isn't obvious.

Good:

# Convert Celsius to Fahrenheit
fahrenheit = (celsius * 9 / 5) + 32

Avoid comments that simply repeat the code:

# Add 1 to age
age = age + 1








##30. Important Concepts to Remember
Lists

Ordered and mutable collections.

Tuples

Ordered and immutable collections.

Index

A position in a sequence, starting from 0.

Slice

A way to extract part of a sequence.

for loop

Repeats code for each item in an iterable.

range()

Produces a sequence of numbers.

Indentation

Defines code blocks in Python.





##PEP 8

A style guide for writing readable Python code.

