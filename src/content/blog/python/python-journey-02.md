---
title: "Python Journey #2"
description: "Learning Python dictionaries, key-value pairs, modifying data, looping, membership checks, and nested data structures."
date: "2026-09-12"
category: "Python"
tags: ["Python", "Dictionaries", "Key-Value", "Nesting"]
---

# Python Journey #2

```python

A **dictionary** stores information as **key-value pairs**.

alien = {
    "color": "green",
    "points": 5
}

Here:

"color"  → "green"
"points" → 5
Keys → "color", "points"
Values → "green", 5

Dictionaries use {}.




##1. Creating a Dictionary
alien = {
    "color": "green",
    "points": 5
}




##2. Accessing Values

Access a value using its key.

print(alien["color"])
print(alien["points"])

Output:

green
5

Syntax:

dictionary["key"]




##3. Adding Key-Value Pairs

Dictionaries are mutable, so new information can be added.

alien["x_position"] = 0
alien["y_position"] = 25

Now the dictionary contains:

color
points
x_position
y_position




##4. Starting with an Empty Dictionary

You can create an empty dictionary using {}.

alien = {}

alien["color"] = "green"
alien["points"] = 5

This is useful when the information is created gradually.




##5. Modifying Values

Assign a new value to an existing key.

alien = {
    "color": "green",
    "points": 5
}

alien["color"] = "yellow"

The existing "color" value is replaced.




##6. Practical Dictionary Example

Dictionaries are useful for representing objects with multiple properties.

alien = {
    "x_position": 0,
    "y_position": 25,
    "speed": "medium"
}

We can modify its position:

alien["x_position"] += 1

We can also use dictionary values in conditions:

if alien["speed"] == "fast":
    alien["x_position"] += 3
elif alien["speed"] == "medium":
    alien["x_position"] += 2
else:
    alien["x_position"] += 1

This shows how dictionaries and if statements can work together.




##7. Removing Key-Value Pairs

Use del to remove a specific key-value pair.

alien = {
    "color": "green",
    "points": 5
}

del alien["points"]

Now:

print(alien)

Output:

{'color': 'green'}
Important
del alien["points"]

removes one key-value pair.

But:

del alien

deletes the entire dictionary variable.




##8. Looping Through a Dictionary

Use .items() to access both keys and values.

user = {
    "username": "nithish",
    "first": "Nithish",
    "last": "Ch"
}

for key, value in user.items():
    print(f"{key}: {value}")

Output:

username: nithish
first: Nithish
last: Ch

.items() provides:

key + value




##9. Looping Through Keys

Use .keys():

for key in user.keys():
    print(key)

You can also simply write:

for key in user:
    print(key)

Both loop through the dictionary's keys.





##10. Looping Through Values

Use .values():

for value in user.values():
    print(value)

Output:

nithish
Nithish
Ch
Remember
.items()   → keys + values
.keys()    → keys
.values()  → values




##11. Checking Whether a Key Exists

Use in to check for a key.

if "username" in user:
    print("Username exists.")

Use not in when checking that a key does not exist.

if "email" not in user:
    print("Email doesn't exist.")




##12. Nesting

Nesting means putting one data structure inside another.

Common forms include:

List of dictionaries
Dictionary of dictionaries
List inside a dictionary
Dictionary inside a dictionary



##13. List of Dictionaries

Suppose we have multiple aliens.

alien_0 = {
    "color": "green",
    "points": 5
}

alien_1 = {
    "color": "yellow",
    "points": 10
}

alien_2 = {
    "color": "red",
    "points": 15
}

We can store them in one list:

aliens = [alien_0, alien_1, alien_2]

Then loop through them:

for alien in aliens:
    print(alien)

Output:

{'color': 'green', 'points': 5}
{'color': 'yellow', 'points': 10}
{'color': 'red', 'points': 15}

This is called a list of dictionaries.





##14. Dictionary Inside a Dictionary

A dictionary can contain other dictionaries.

users = {
    "nithish": {
        "first": "Nithish",
        "last": "Ch",
        "age": 21
    },

    "rahul": {
        "first": "Rahul",
        "last": "Kumar",
        "age": 22
    }
}

Access nested data:

print(users["nithish"]["age"])

Output:

21

The access pattern is:

users
  ↓
"nithish"
  ↓
"age"
  ↓
21





##15. Looping Through Nested Dictionaries
for username, user_info in users.items():
    print(f"Username: {username}")

    full_name = f"{user_info['first']} {user_info['last']}"
    print(f"Name: {full_name}")

Here:

users.items()
      ↓
username + user_info
      ↓
user_info contains another dictionary





##16. List Inside a Dictionary

A dictionary value can be a list.

person = {
    "name": "Nithish",
    "skills": ["Python", "Machine Learning", "PyTorch"]
}

Access the complete list:

print(person["skills"])

Access an individual element:

print(person["skills"][0])

Output:

Python

Loop through the list:

for skill in person["skills"]:
    print(skill)

Output:

Python
Machine Learning
PyTorch




##17. Dictionaries Can Store Different Data Types

Dictionary values can be different types.

data = {
    "name": "Nithish",
    "age": 21,
    "height": 5.8,
    "skills": ["Python", "ML"],
    "active": True,
    "address": {
        "city": "Kakinada"
    }
}

Values can be:

string
integer
float
list
Boolean
dictionary

This makes dictionaries very useful for representing structured data.





##18. Dictionary Cheat Sheet

Create:

user = {
    "name": "Nithish",
    "age": 21
}

Access:

user["name"]

Add:

user["email"] = "example@email.com"

Modify:

user["age"] = 22

Delete:

del user["age"]

Loop through everything:

for key, value in user.items():
    print(key, value)

Keys:

user.keys()

Values:

user.values()

Check existence:

"name" in user





##19. List vs Dictionary

A list uses indexes:

skills = ["Python", "ML"]

print(skills[0])

A dictionary uses keys:

user = {
    "name": "Nithish"
}

print(user["name"])
Remember
List       → index
Dictionary → key




##20. Important Dictionary Concepts
Dictionary
    ↓
key → value
    ↓
Access
    ↓
Add
    ↓
Modify
    ↓
Delete
    ↓
Loop
    ↓
Nesting







## 21. The `input()` Function

`input()` allows a program to receive information from the user.

name = input("What is your name? ")

print(f"Hello, {name}!")

Example:

What is your name? Nithish
Hello, Nithish!

The text inside input() is called the prompt.





##22. input() Always Returns a String

This is very important.

age = input("How old are you? ")

print(type(age))

If the user enters 21, the result is still:

<class 'str'>

So Python stores:

"21"

not:

21




##23. Converting Input to an Integer

Use int() when you need an integer.

age = int(input("How old are you? "))

print(type(age))

Now the type is:

<class 'int'>

For decimal numbers, use float():

height = float(input("Enter your height: "))




##24. Using Input in Calculations

This causes an error:

age = input("How old are you? ")

print(age + 1)

because age is a string.

Correct:

age = int(input("How old are you? "))

print(age + 1)

If the user enters 21:

22





##25. The while Loop

A while loop repeatedly executes code while a condition is True.

current_number = 1

while current_number <= 5:
    print(current_number)
    current_number += 1

Output:

1
2
3
4
5

The condition is checked before each iteration.




##26. while Loop Structure

Basic syntax:

while condition:
    # code to execute

Example:

number = 1

while number <= 10:
    print(number)
    number += 1

Something inside the loop usually needs to change the condition.





##27. Infinite Loops

Be careful with:

number = 1

while number <= 5:
    print(number)

number never changes, so the condition remains True.

The loop continues forever.

This is called an infinite loop.

Correct:

number = 1

while number <= 5:
    print(number)
    number += 1




##28. while with User Input

A while loop can repeatedly ask the user for input.

prompt = "Tell me something, and I will repeat it."
prompt += "\nEnter 'quit' to end the program: "

message = ""

while message != "quit":
    message = input(prompt)

    if message != "quit":
        print(message)

The loop continues until the user enters:

quit




##29. Using a Flag

A flag is a Boolean variable used to control a loop.

active = True

while active:
    message = input("Enter something: ")

    if message == "quit":
        active = False
    else:
        print(message)

Initially:

active = True

The loop continues.

When:

active = False

the loop stops.





##30. break

break immediately exits a loop.

while True:
    message = input("Enter something: ")

    if message == "quit":
        break

    print(message)

Even though:

while True:

would normally continue forever, break stops it.





##31. continue

continue skips the rest of the current iteration and starts the next iteration.

number = 0

while number < 10:
    number += 1

    if number % 2 == 0:
        continue

    print(number)

Output:

1
3
5
7
9

When the number is even, continue skips print().




##32. Processing a List with while

A while loop can process a list until it becomes empty.

unconfirmed_users = ["alice", "brian", "candace"]
confirmed_users = []

while unconfirmed_users:
    current_user = unconfirmed_users.pop()

    print(f"Verifying user: {current_user}")
    confirmed_users.append(current_user)

The condition:

while unconfirmed_users:

means:

Continue while the list is not empty.

Each pop() removes one item.

Eventually:

["alice", "brian", "candace"]
        ↓
["alice", "brian"]
        ↓
["alice"]
        ↓
[]
        ↓
STOP




##33. Removing All Instances of a Value

Suppose:

pets = ["cat", "dog", "cat", "rabbit", "cat"]

We want to remove every "cat".

while "cat" in pets:
    pets.remove("cat")

print(pets)

Output:

['dog', 'rabbit']
Important Pattern
while value in my_list:
    my_list.remove(value)

The loop continues until the value no longer exists.





##34. Using while with Dictionaries

A while loop can repeatedly collect data and store it in a dictionary.

responses = {}

polling_active = True

while polling_active:
    name = input("What is your name? ")
    response = input("Which mountain would you like to climb? ")

    responses[name] = response

    repeat = input("Would you like another person to respond? (yes/no) ")

    if repeat == "no":
        polling_active = False

The dictionary grows as users provide information.

For example:

{}
 ↓
{"Nithish": "Everest"}
 ↓
{
    "Nithish": "Everest",
    "Rahul": "Kilimanjaro"
}






##35. A Common Interactive Pattern

A very common Python pattern is:

while True:
    data = input("Enter something: ")

    if data == "quit":
        break

    # process data

This pattern is useful for interactive programs.





##36. for vs while
for

Use for when processing each item in a collection.

names = ["Alice", "Bob", "Charlie"]

for name in names:
    print(name)

Think:

for
→ Do this for every item.
while

Use while when continuing until a condition changes.

password = ""

while password != "python123":
    password = input("Enter password: ")

Think:

while
→ Keep doing this while the condition is True.




##37. input() Cheat Sheet

Get text:

name = input("Name: ")

Get an integer:

age = int(input("Age: "))

Get a decimal:

height = float(input("Height: "))

Remember:

input() → always returns a string





##38. while Cheat Sheet

Basic loop:

while condition:
    # code

Exit immediately:

break

Skip the current iteration:

continue

Control with a Boolean:

active = True

while active:
    ...

Process until a list is empty:

while my_list:
    item = my_list.pop()




##39. Important Differences
Concept	Purpose
input()	Get data from the user
int()	Convert to integer
float()	Convert to decimal
while	Repeat while condition is true
break	Exit loop immediately
continue	Skip current iteration
Flag	Boolean variable controlling a loop








## 40. Defining a Function

Use the `def` keyword to define a function.

def greet_user():
    print("Hello!")

Defining a function does not execute it.

Call it using:

greet_user()
Basic structure
def function_name():
    # code




## 41. Calling a Function

A function runs when we call it.

def greet():
    print("Hello")
    print("Welcome to Python")

greet()

Output:

Hello
Welcome to Python





## 42. Parameters and Arguments

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





## 43. Multiple Parameters

A function can have multiple parameters.

def describe_pet(animal_type, pet_name):
    print(f"I have a {animal_type}.")
    print(f"My {animal_type}'s name is {pet_name}.")

Call:

describe_pet("hamster", "Harry")

Output:

I have a hamster.
My hamster's name is Harry.





## 44. Positional Arguments

Arguments can be assigned according to their position.

describe_pet("hamster", "Harry")

Means:

animal_type → "hamster"
pet_name    → "Harry"

Order matters.

describe_pet("Harry", "hamster")

would give the wrong meaning.




## 45. Keyword Arguments

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





## 46. Default Parameter Values

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






## 47. Returning Values

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




## 48. print() vs return

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





## 49. Returning Early

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





## 50. Returning Multiple Values

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






## 51. Passing a List to a Function

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




## 52. Modifying a List Inside a Function

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




## 53. Passing a Copy of a List

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




## 54. Arbitrary Positional Arguments: *args

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




## 55. Arbitrary Keyword Arguments: **kwargs

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




## 56. Modules

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





## 57. Importing a Specific Function

Instead of importing the entire module:

import pizza

you can import a specific function:

from pizza import make_pizza

Then:

make_pizza(16, "pepperoni")

No pizza. prefix is required.




## 58. Module Aliases

You can give a module a shorter name.

import pizza as p

Then:

p.make_pizza(16, "pepperoni")

You can also alias a function:

from pizza import make_pizza as mp

Then:

mp(16, "pepperoni")




## 59. Importing Everything

Python allows:

from pizza import *

But this is generally not recommended.

It can make it difficult to know where functions came from and may cause naming conflicts.

Prefer:

import pizza

or:

from pizza import make_pizza




## 60. Function Naming

Use descriptive function names.

Good:

def calculate_average():

Avoid unclear names:

def ca():

Python convention is lowercase with underscores:

def get_user_name():

This is called snake_case.




## 61. Function Indentation

Use 4 spaces for indentation.

def greet_user():
    print("Hello")

Consistent indentation improves readability.






## 62. Blank Lines Around Functions

Top-level functions should be separated by blank lines.

def function_one():
    pass


def function_two():
    pass

This improves readability.





## 63. Function Parameters Style

Avoid unnecessary spaces.

Good:

def greet_user(name):

Not:

def greet_user( name ):




## 64. Docstrings

A docstring explains what a function does.

def greet_user(name):
    """Print a greeting for the given user."""
    print(f"Hello, {name}!")

You can inspect the documentation with:

help(greet_user)





## 65. Function Cheat Sheet
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




## 66. Important Distinctions


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
