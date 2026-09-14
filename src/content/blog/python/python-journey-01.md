---
title: "Python Journey #1 "
description: "Learning Python fundamentals including comments, variables, strings, numbers, arithmetic, constants, and the Zen of Python."
date: "2026-09-11"
category: "Python"
tags: ["Python", "Basics", "Variables", "Strings", "Numbers"]
---

# Python Journey #1 - Python 


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




A **list** is an ordered and changeable collection of values.

## 12. Creating a List

Lists use square brackets `[]`.

cars = ["BMW", "Audi", "Tesla"]

Each value is called an element.




## 13. List Indexing

Python uses zero-based indexing.

BMW    → 0
Audi   → 1
Tesla  → 2

Access an element using its index:

print(cars[0])

Output:

BMW





## 14. Changing Elements

Lists are mutable, meaning their elements can be changed.

cars = ["BMW", "Audi", "Tesla"]

cars[1] = "Mercedes"

print(cars)

Output:

['BMW', 'Mercedes', 'Tesla']






## 15. Adding Elements

4.1 append()

Adds an element to the end.

cars.append("Toyota")


4.2 insert()

Adds an element at a specific position.

cars.insert(1, "Honda")





## 16. Removing Elements


16.1 del

Removes an element using its index.

del cars[1]



16.2 pop()

Removes and returns an element.

car = cars.pop()

By default, it removes the last element.



16.3 remove()

Removes an element using its value.

cars.remove("Audi")
Remember
del      → remove by index
pop()    → remove and return
remove() → remove by value




## 17. Sorting Lists


17.1 sort()

Changes the original list.

cars.sort()

Reverse sorting:

cars.sort(reverse=True)



17.2 sorted()

Returns a sorted version without changing the original list.

sorted(cars)
Remember
sort()   → changes the original list
sorted() → returns a sorted version





## 18. Reversing a List

reverse() reverses the current order.

cars.reverse()

It does not mean alphabetical reverse sorting.




## 19. Finding the Length

Use len() to find the number of elements.

cars = ["BMW", "Audi", "Tesla"]

print(len(cars))

Output:

3




## 20. Negative Indexing

Python allows negative indexes to access elements from the end.

BMW    → -3
Audi   → -2
Tesla  → -1

For example:

print(cars[-1])

Output:

Tesla





## 21. Index Errors

If a list contains 3 elements, the valid indexes are:

0, 1, 2

This causes an error:

print(cars[3])

because index 3 does not exist.

The error is:

IndexError




## 22. Important List Methods
Method	Purpose
append()	Add to end
insert()	Add at a position
remove()	Remove by value
pop()	Remove and return
sort()	Sort the list
reverse()	Reverse the list





## 23. Working with Lists

Lists become especially useful when we want to perform the same operation on many elements.

magicians = ["alice", "david", "carolina"]

Instead of accessing every element separately:

print(magicians[0])
print(magicians[1])
print(magicians[2])

we can use a loop.





## 24. Looping Through a List

Use a for loop to process each element.

magicians = ["alice", "david", "carolina"]

for magician in magicians:
    print(magician)

Output:

alice
david
carolina

The loop takes each element one at a time and stores it temporarily in magician.





## 25. for Loop Structure
for variable in list:
    # code to execute

Example:

names = ["Nithish", "Rahul", "Arjun"]

for name in names:
    print(f"Hello {name}!")




##26. Multiple Statements Inside a Loop

A loop can contain multiple statements.

magicians = ["alice", "david", "carolina"]

for magician in magicians:
    print(f"{magician.title()}, that was a great trick!")
    print(f"I can't wait to see your next trick, {magician.title()}.\n")

All indented statements execute during every iteration.





##27. Indentation

Python uses indentation to define blocks of code.

for magician in magicians:
    print(magician)

The print() statement belongs to the loop because it is indented.

Important

Python convention:

4 spaces = one indentation level






## 28. Indentation Errors

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







## 29. The Colon :

A colon tells Python that an indented block is starting.

Incorrect:

for magician in magicians
    print(magician)

Correct:

for magician in magicians:
    print(magician)




##30. Numerical Lists with range()

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




## 31. Converting range() to a List

range() produces a range object.

Use list() to convert it into a list.

numbers = list(range(1, 6))

print(numbers)

Output:

[1, 2, 3, 4, 5]




## 32. range() With a Step

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




## 33. Creating a List of Squares

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





##34. Basic Statistics

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





##35. List Slicing

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





##36. Omitting the Start Index
print(players[:3])

This means:

Start from the beginning and stop before index 3.

Output:

['charles', 'martina', 'michael']




##37. Omitting the End Index
print(players[2:])

This means:

Start at index 2 and continue to the end.

Output:

['michael', 'florence', 'eli']





##38. Copying a List with Slicing

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





##39. Tuples

A tuple is similar to a list but is immutable.

List:

dimensions = [200, 50]

Tuple:

dimensions = (200, 50)

Tuples normally use parentheses ().





##40. Accessing Tuple Elements

Tuples use indexing just like lists.

dimensions = (200, 50)

print(dimensions[0])

Output:

200






##41. Tuples Cannot Be Modified

Lists can be changed:

dimensions = [200, 50]

dimensions[0] = 250

Tuples cannot:

dimensions = (200, 50)

dimensions[0] = 250

This causes:

TypeError

because tuples are immutable.




##42. When to Use Tuples

Use a tuple when the values should not change.

Example:

screen_size = (1920, 1080)
List
colors = ["red", "blue", "green"]

Mutable.

Tuple
dimensions = (1920, 1080)

Immutable.




##43. Looping Through a Tuple

Tuples can also be used with for loops.

dimensions = (1920, 1080)

for dimension in dimensions:
    print(dimension)

Output:

1920
1080




##44. Reassigning a Tuple

You cannot change an individual tuple element:

dimensions[0] = 2500

But you can assign a completely new tuple to the variable:

dimensions = (1920, 1080)

dimensions = (2560, 1440)

print(dimensions)

Output:

(2560, 1440)

The original tuple was not modified. The variable now refers to a new tuple.




##45. Python Code Style

Python follows coding conventions described mainly by PEP 8.

The goal is:

Write code that is easy for humans to read.





##46. Variable Naming

Use lowercase names with underscores.

Good:

first_name = "Nithish"
user_age = 21

This style is called snake_case.




##47. Spaces Around Operators

Prefer:

age = 21
total = price + tax

Instead of:

age=21
total=price+tax

Readable spacing makes code easier to understand.





##48. Consistent Indentation

Use 4 spaces for indentation.

for name in names:
    print(name)

Avoid mixing tabs and spaces.





##49. Blank Lines

Use blank lines to separate logical sections.

name = "Nithish"
age = 21


print(name)
print(age)

Don't use excessive blank lines.




##50. Long Lists

Long lists can be written across multiple lines.

languages = [
    "Python",
    "Java",
    "C++",
    "JavaScript",
    "Go",
]

This improves readability.




##51. Useful Comments

Comments should explain something that isn't obvious.

Good:

# Convert Celsius to Fahrenheit
fahrenheit = (celsius * 9 / 5) + 32

Avoid comments that simply repeat the code:

# Add 1 to age
age = age + 1



##52. Important Concepts to Remember
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








## 53. Basic `if` Statement
age = 20

if age >= 18:
    print("You are an adult.")

Output:

You are an adult.

Python checks the condition:

20 >= 18 → True

Because it is True, the indented code runs.




##54. if with a List

We can combine for loops and if statements.

cars = ["audi", "bmw", "subaru", "toyota"]

for car in cars:
    if car == "bmw":
        print(car.upper())
    else:
        print(car.title())

Output:

Audi
BMW
Subaru
Toyota





##55. Conditional Tests

A conditional test evaluates to either:

True
False

These are Python's Boolean values.

Example:

age = 21

print(age == 21)

Output:

True




##56. Equality ==

Use == to check whether two values are equal.

car = "bmw"

if car == "bmw":
    print("It's a BMW.")
Important

= means assignment:

car = "bmw"

== means comparison:

car == "bmw"




##57. Inequality !=

Use != to check whether two values are different.

age = 20

if age != 18:
    print("Age is not 18.")





##58. Comparison Operators

Python provides several comparison operators:

>     greater than
<     less than
>=    greater than or equal
<=    less than or equal
==    equal
!=    not equal

Example:

age = 21

if age >= 18:
    print("You can vote.")



##59. and

and requires both conditions to be true.

age = 25
has_id = True

if age >= 18 and has_id:
    print("Access granted.")

Both conditions are True, so the code runs.




##60. or

or requires at least one condition to be true.

day = "Saturday"

if day == "Saturday" or day == "Sunday":
    print("Weekend!")





##61. not

not reverses a Boolean value.

is_raining = False

if not is_raining:
    print("Go outside.")




##62. Checking Membership with in

in checks whether a value exists inside a collection.

requested_toppings = ["mushrooms", "onions", "pineapple"]

if "mushrooms" in requested_toppings:
    print("Adding mushrooms.")

Output:

Adding mushrooms.






##63. Checking Membership with not in

not in checks whether a value does not exist.

if "pepperoni" not in requested_toppings:
    print("We need pepperoni!")




##64. if-else

Use else when there are two possible outcomes.

age = 16

if age >= 18:
    print("Adult")
else:
    print("Minor")

Output:

Minor

Structure:

if condition:
    # True

else:
    # False





##65. if-elif-else

Use elif when checking multiple conditions.

age = 12

if age >= 18:
    print("Adult")
elif age >= 13:
    print("Teenager")
else:
    print("Child")

Output:

Child

Python checks conditions from top to bottom.

The first matching condition runs.





##66. Multiple elif Statements

You can use multiple elif blocks.

age = 30

if age < 5:
    price = 0
elif age < 18:
    price = 10
elif age < 65:
    price = 20
else:
    price = 10

print(price)

Output:

20




##67. Using if with Lists

We can use conditions while looping through a list.

requested_toppings = [
    "mushrooms",
    "green peppers",
    "extra cheese"
]

for topping in requested_toppings:
    if topping == "green peppers":
        print("Sorry, we are out of green peppers.")
    else:
        print(f"Adding {topping}.")

Output:

Adding mushrooms.
Sorry, we are out of green peppers.
Adding extra cheese.
Important Pattern
List
  ↓
for loop
  ↓
if statement
  ↓
Action




##68. Checking Whether a List Is Empty

Python allows us to directly test a list.

requested_toppings = []

if requested_toppings:
    print("We have toppings.")
else:
    print("Are you sure you want a plain pizza?")

Output:

Are you sure you want a plain pizza?

An empty list is considered False.

A non-empty list is considered True.

This is called truthiness.





##69. Checking Multiple Lists

We can check whether requested items are available.

available_toppings = [
    "mushrooms",
    "olives",
    "green peppers",
    "pepperoni"
]

requested_toppings = [
    "mushrooms",
    "french fries",
    "pepperoni"
]

for topping in requested_toppings:
    if topping in available_toppings:
        print(f"Adding {topping}.")
    else:
        print(f"Sorry, we don't have {topping}.")

Output:

Adding mushrooms.
Sorry, we don't have french fries.
Adding pepperoni.
General Pattern
for item in requested_items:
    if item in available_items:
        # process item
    else:
        # handle unavailable item





##70. Boolean Variables

Boolean variable names can make conditions easier to understand.

is_logged_in = True
has_permission = False
is_active = True

Then:

if is_logged_in:
    print("Welcome!")

This makes the code easy to read.






##71. PEP 8 Style

Python follows PEP 8 conventions for readable code.

Use spaces around operators:

if age >= 18:
    print("Adult")

Use 4 spaces for indentation:

if age >= 18:
    print("Adult")

Prefer readable conditions:

if age >= 18:

instead of:

if(age>=18):

Use meaningful Boolean names:

is_logged_in = True
has_permission = False




##72. Important Operators
==       equal
!=       not equal
>        greater than
<        less than
>=       greater than or equal
<=       less than or equal

and      both conditions must be true
or       at least one condition must be true
not      reverses True/False

in       exists inside a collection
not in   does not exist inside a collection




##73. if Statement Structure

The main structure is:

if condition:
    # code if True

elif another_condition:
    # code if True

else:
    # fallback

Remember:

if     → first condition
elif   → another condition
else   → everything else