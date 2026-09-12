---
title: "Python Journey #4 - If Statements"
description: "Learning conditional tests, comparisons, Boolean logic, if-elif-else statements, membership checks, truthiness, and conditional logic with lists."
date: "2026-09-12"
category: "Python"
tags: ["Python", "If Statements", "Conditionals", "Boolean", "PEP8"]
---

# Python Journey #4 - If Statements

`if` statements allow Python to **make decisions** based on whether a condition is `True` or `False`.

---



```python

## 1. Basic `if` Statement
age = 20

if age >= 18:
    print("You are an adult.")

Output:

You are an adult.

Python checks the condition:

20 >= 18 → True

Because it is True, the indented code runs.






##2. if with a List

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





##3. Conditional Tests

A conditional test evaluates to either:

True
False

These are Python's Boolean values.

Example:

age = 21

print(age == 21)

Output:

True






##4. Equality ==

Use == to check whether two values are equal.

car = "bmw"

if car == "bmw":
    print("It's a BMW.")
Important

= means assignment:

car = "bmw"

== means comparison:

car == "bmw"








##5. Inequality !=

Use != to check whether two values are different.

age = 20

if age != 18:
    print("Age is not 18.")








##6. Comparison Operators

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







##7. and

and requires both conditions to be true.

age = 25
has_id = True

if age >= 18 and has_id:
    print("Access granted.")

Both conditions are True, so the code runs.







##8. or

or requires at least one condition to be true.

day = "Saturday"

if day == "Saturday" or day == "Sunday":
    print("Weekend!")







##9. not

not reverses a Boolean value.

is_raining = False

if not is_raining:
    print("Go outside.")








##10. Checking Membership with in

in checks whether a value exists inside a collection.

requested_toppings = ["mushrooms", "onions", "pineapple"]

if "mushrooms" in requested_toppings:
    print("Adding mushrooms.")

Output:

Adding mushrooms.









##11. Checking Membership with not in

not in checks whether a value does not exist.

if "pepperoni" not in requested_toppings:
    print("We need pepperoni!")








##12. if-else

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









##13. if-elif-else

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










##14. Multiple elif Statements

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









##15. Using if with Lists

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







##16. Checking Whether a List Is Empty

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










##17. Checking Multiple Lists

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









##18. Boolean Variables

Boolean variable names can make conditions easier to understand.

is_logged_in = True
has_permission = False
is_active = True

Then:

if is_logged_in:
    print("Welcome!")

This makes the code easy to read.











##19. PEP 8 Style

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









##20. Important Operators
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










##21. if Statement Structure

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