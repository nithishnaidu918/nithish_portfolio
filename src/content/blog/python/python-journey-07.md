---
title: "Python Journey #7 - Functions"
description: "Learning Python functions, parameters, arguments, return values, arbitrary arguments, modules, imports, and function styling."
date: "2026-09-12"
category: "Python"
tags: ["Python", "Functions", "Arguments", "Return", "Modules", "Imports"]
---

# Python Journey #7 - Functions

A **function** is a reusable block of code that performs a specific task.

Functions help us avoid repetition, organize code, and reuse logic.

---


```python

## 1. Defining a Function

Use the `def` keyword to define a function.

def greet_user():
    print("Hello!")

Defining a function does not execute it.

Call it using:

greet_user()
Basic structure
def function_name():
    # code





## 2. Calling a Function

A function runs when we call it.

def greet():
    print("Hello")
    print("Welcome to Python")

greet()

Output:

Hello
Welcome to Python








## 3. Parameters and Arguments

Functions can receive information.

def greet_user(username):
    print(f"Hello, {username}!")

Call:

greet_user("Nithish")

Output:

Hello, Nithish!

Here:

username  → parameter
"Nithish" → argument
Remember

Parameter = variable in the function definition.

Argument = actual value passed to the function.









## 4. Multiple Parameters

A function can have multiple parameters.

def describe_pet(animal_type, pet_name):
    print(f"I have a {animal_type}.")
    print(f"My {animal_type}'s name is {pet_name}.")

Call:

describe_pet("hamster", "Harry")

Output:

I have a hamster.
My hamster's name is Harry.








## 5. Positional Arguments

Arguments can be assigned according to their position.

describe_pet("hamster", "Harry")

Means:

animal_type → "hamster"
pet_name    → "Harry"

Order matters.

describe_pet("Harry", "hamster")

would give the wrong meaning.










## 6. Keyword Arguments

You can explicitly specify which parameter receives each value.

describe_pet(
    animal_type="hamster",
    pet_name="Harry"
)

The order no longer matters:

describe_pet(
    pet_name="Harry",
    animal_type="hamster"
)

These are called keyword arguments.











## 7. Default Parameter Values

A parameter can have a default value.

def describe_pet(pet_name, animal_type="dog"):
    print(f"I have a {animal_type}.")
    print(f"My {animal_type}'s name is {pet_name}.")

Now:

describe_pet("Willie")

uses the default:

I have a dog.
My dog's name is Willie.

You can override it:

describe_pet("Harry", "hamster")
Important

Parameters without defaults generally come before parameters with defaults.

def describe_pet(pet_name, animal_type="dog"):









## 8. Returning Values

Functions don't always need to print results.

They can return a value.

def get_formatted_name(first_name, last_name):
    full_name = f"{first_name} {last_name}"
    return full_name

Use the returned value:

name = get_formatted_name("Nithish", "Ch")

print(name)

Output:

Nithish Ch
Mental model
function
   ↓
process
   ↓
return result
   ↓
use result











## 9. print() vs return

print() displays something:

def add(a, b):
    print(a + b)

return sends a value back to the caller:

def add(a, b):
    return a + b

Now the result can be reused:

result = add(10, 20)

new_result = result * 2
Remember
print()  → displays a result
return   → sends a result back









## 10. Returning Early

A function stops immediately when it reaches return.

def check_age(age):
    if age < 18:
        return "Minor"

    return "Adult"

Example:

print(check_age(20))

Output:

Adult

Once return executes, the function ends.









## 11. Returning Multiple Values

Python can return multiple values.

def calculate(a, b):
    return a + b, a * b

Use:

addition, multiplication = calculate(5, 3)

print(addition)
print(multiplication)

Output:

8
15

Technically, Python returns a tuple:

(8, 15)













## 12. Passing a List to a Function

Functions can accept lists.

def greet_users(names):
    for name in names:
        print(f"Hello, {name}!")

Call:

users = ["Nithish", "Rahul", "Arjun"]

greet_users(users)

Output:

Hello, Nithish!
Hello, Rahul!
Hello, Arjun!

The function receives the entire list.










## 13. Modifying a List Inside a Function

Functions can modify mutable objects such as lists.

unprinted_designs = [
    "phone case",
    "robot pendant",
    "dodecahedron"
]

completed_models = []

Function:

def print_models(unprinted_designs, completed_models):
    while unprinted_designs:
        current_design = unprinted_designs.pop()

        print(f"Printing: {current_design}")
        completed_models.append(current_design)

Call:

print_models(unprinted_designs, completed_models)

The function modifies both lists.










## 14. Passing a Copy of a List

Sometimes we don't want the original list to be modified.

Use slicing:

print_models(
    unprinted_designs[:],
    completed_models
)

The [:] creates a copy.

Important
original_list[:] → separate copy
original_list    → same list

This is important when working with mutable objects.












## 15. Arbitrary Positional Arguments: *args

Sometimes a function needs to accept any number of positional arguments.

def make_pizza(*toppings):
    print(toppings)

Now we can pass one or many:

make_pizza("mushrooms")

or:

make_pizza("mushrooms", "olives", "pepperoni")

The * collects all extra positional arguments into a tuple.

*toppings
    ↓
extra positional arguments
    ↓
tuple

We can loop through them:

def make_pizza(*toppings):
    print("Making pizza with:")

    for topping in toppings:
        print(f"- {topping}")










## 16. Arbitrary Keyword Arguments: **kwargs

** collects extra keyword arguments into a dictionary.

def build_profile(first, last, **user_info):
    user_info["first"] = first
    user_info["last"] = last

    return user_info

Call:

profile = build_profile(
    "Nithish",
    "Ch",
    location="India",
    field="AI Engineering"
)

user_info contains:

{
    "location": "India",
    "field": "AI Engineering",
    "first": "Nithish",
    "last": "Ch"
}
Remember
*args
→ extra positional arguments
→ tuple

**kwargs
→ extra keyword arguments
→ dictionary










## 17. Modules

As programs grow, putting every function into one file becomes difficult.

A module is a Python file containing reusable code.

Example:

pizza.py
def make_pizza(size, *toppings):
    print(f"Making a {size}-inch pizza.")

    for topping in toppings:
        print(f"- {topping}")

Another file can import it.

making_pizzas.py
import pizza

pizza.make_pizza(
    16,
    "pepperoni",
    "mushrooms"
)

Output:

Making a 16-inch pizza.
- pepperoni
- mushrooms









## 18. Importing a Specific Function

Instead of importing the entire module:

import pizza

you can import a specific function:

from pizza import make_pizza

Then:

make_pizza(16, "pepperoni")

No pizza. prefix is required.











## 19. Module Aliases

You can give a module a shorter name.

import pizza as p

Then:

p.make_pizza(16, "pepperoni")

You can also alias a function:

from pizza import make_pizza as mp

Then:

mp(16, "pepperoni")











## 20. Importing Everything

Python allows:

from pizza import *

But this is generally not recommended.

It can make it difficult to know where functions came from and may cause naming conflicts.

Prefer:

import pizza

or:

from pizza import make_pizza









## 21. Function Naming

Use descriptive function names.

Good:

def calculate_average():

Avoid unclear names:

def ca():

Python convention is lowercase with underscores:

def get_user_name():

This is called snake_case.










## 22. Function Indentation

Use 4 spaces for indentation.

def greet_user():
    print("Hello")

Consistent indentation improves readability.










## 23. Blank Lines Around Functions

Top-level functions should be separated by blank lines.

def function_one():
    pass


def function_two():
    pass

This improves readability.










## 24. Function Parameters Style

Avoid unnecessary spaces.

Good:

def greet_user(name):

Not:

def greet_user( name ):










## 25. Docstrings

A docstring explains what a function does.

def greet_user(name):
    """Print a greeting for the given user."""
    print(f"Hello, {name}!")

You can inspect the documentation with:

help(greet_user)










## 26. Function Cheat Sheet
Define
def greet():
    print("Hello")

Call
greet()

Parameter
def greet(name):

Argument
greet("Nithish")

Return
def add(a, b):
    return a + b

List parameter
def process(items):
    for item in items:
        print(item)

Arbitrary positional arguments
def pizza(*toppings):
toppings → tuple

Arbitrary keyword arguments
def profile(**info):
info → dictionary

Module
import pizza
Specific function
from pizza import make_pizza




## 27. Important Distinctions


## Function vs Module

Function
→ reusable block of code

Module
→ Python file containing reusable code



## print() vs return

print()
→ displays something

return
→ sends a value back to the caller






## *args vs **kwargs

*args
→ extra positional arguments
→ tuple

**kwargs
→ extra keyword arguments
→ dictionary



## Parameter vs Argument

Parameter
→ variable in function definition

Argument
→ actual value passed to function
