---
title: "Python Journey #3"
description: "Learning Python classes, objects, attributes, methods, inheritance, modules, and the standard library."
date: "2026-09-12"
category: "Python"
tags: ["Python", "Classes", "Objects", "OOP", "Inheritance", "Modules"]
---

# Python Journey #3 - Classes

A **class** is a blueprint for creating objects.

A class can define:

- **Attributes** → data
- **Methods** → behavior

---



```python
## 1. Creating a Class

Use the `class` keyword.
class Car:
    pass

Create objects from the class:

car1 = Car()
car2 = Car()

car1 and car2 are instances of Car.

Mental model
Class
  ↓
Blueprint
  ↓
Instances
  ↓
car1, car2, car3...




## 2. The __init__() Method

__init__() is a special method that runs automatically when an instance is created.

class Dog:
    def __init__(self, name, age):
        self.name = name
        self.age = age

When we create:

my_dog = Dog("Willie", 6)

Python initializes the object with:

name → Willie
age  → 6





## 3. The self Parameter

self refers to the current instance.

self.name = name
self.age = age

This stores data inside that particular object.

We use self constantly when working with classes.




## 4. Creating Instances

Create an instance:

my_dog = Dog("Willie", 6)

Access its attributes:

print(my_dog.name)
print(my_dog.age)

Output:

Willie
6





## 5. Attributes

An attribute is data stored inside an instance.

class Dog:
    def __init__(self, name, age):
        self.name = name
        self.age = age

Here:

self.name
self.age

are attributes.

For:

my_dog = Dog("Willie", 6)

we have:

my_dog.name → "Willie"
my_dog.age  → 6





## 6. Methods

A method is a function defined inside a class.

class Dog:
    def sit(self):
        print("The dog is sitting.")

sit() is a method.

Call it using:

my_dog.sit()
Remember
Attribute
→ stores data

Method
→ performs an action





## 7. Working with Classes and Instances

A class can contain both attributes and methods.

class Dog:

    def __init__(self, name, age):
        self.name = name
        self.age = age

    def sit(self):
        print(f"{self.name} is sitting.")

    def roll_over(self):
        print(f"{self.name} rolled over!")

Create an instance:

my_dog = Dog("Willie", 6)

Call methods:

my_dog.sit()
my_dog.roll_over()

Output:

Willie is sitting.
Willie rolled over!





## 8. Multiple Instances

One class can create many independent objects.

my_dog = Dog("Willie", 6)
your_dog = Dog("Lucy", 3)

Each object has its own data:

print(my_dog.name)
print(your_dog.name)

Output:

Willie
Lucy

The objects share the class definition but have their own instance data.




## 9. Default Attributes

An attribute can have a default value.

class Car:

    def __init__(self, make, model):
        self.make = make
        self.model = model
        self.odometer_reading = 0

Create a car:

my_car = Car("Toyota", "Camry")

print(my_car.odometer_reading)

Output:

0

Every new car starts with an odometer value of 0.




## 10. Modifying an Attribute

You can directly modify an attribute:

my_car.odometer_reading = 100

But it is often better to provide a method for related behavior.

class Car:

    def __init__(self, make, model):
        self.make = make
        self.model = model
        self.odometer_reading = 0

    def read_odometer(self):
        print(f"This car has {self.odometer_reading} miles.")

Call:

my_car.read_odometer()



## 11. Updating an Attribute Through a Method

Create a method to update the attribute:

class Car:

    def __init__(self, make, model):
        self.make = make
        self.model = model
        self.odometer_reading = 0

    def update_odometer(self, mileage):
        self.odometer_reading = mileage

Then:

my_car.update_odometer(500)

print(my_car.odometer_reading)

Output:

500




## 12. Inheritance

Inheritance allows one class to inherit attributes and methods from another class.

Parent class:

class Car:

    def __init__(self, make, model):
        self.make = make
        self.model = model

    def describe(self):
        print(f"{self.make} {self.model}")

Child class:

class ElectricCar(Car):
    pass

ElectricCar inherits from Car.

Now:

my_tesla = ElectricCar("Tesla", "Model 3")

my_tesla.describe()

The describe() method comes from Car.




## 13. Parent and Child Classes

Terminology:

Car
 ↓
Parent class / superclass
 ↓
ElectricCar
 ↓
Child class / subclass

This:

class ElectricCar(Car):

means:

Create ElectricCar based on Car.




## 14. Adding Attributes to a Child Class

A child class can have additional attributes.

class ElectricCar(Car):

    def __init__(self, make, model):
        super().__init__(make, model)
        self.battery_size = 75

Now:

my_tesla = ElectricCar("Tesla", "Model 3")

print(my_tesla.make)
print(my_tesla.model)
print(my_tesla.battery_size)

Output:

Tesla
Model 3
75




## 15. super()

super() is used to call functionality from the parent class.

super().__init__(make, model)

This means:

Call the parent class's __init__() method.

Conceptually:

ElectricCar.__init__()
        ↓
super()
        ↓
Car.__init__()
        ↓
make + model initialized
        ↓
ElectricCar adds battery_size




## 16. Adding Methods to a Child Class

A child class can have its own methods.

class ElectricCar(Car):

    def __init__(self, make, model):
        super().__init__(make, model)
        self.battery_size = 75

    def describe_battery(self):
        print(f"This car has a {self.battery_size}-kWh battery.")

Now the object can use both:

Car methods
+
ElectricCar methods

Example:

my_tesla.describe()
my_tesla.describe_battery()




## 17. Overriding Methods

A child class can provide its own version of a parent method.

Parent:

class Car:

    def fill_gas_tank(self):
        print("Filling the gas tank.")

Child:

class ElectricCar(Car):

    def fill_gas_tank(self):
        print("This car doesn't need a gas tank.")

Now:

my_tesla.fill_gas_tank()

Output:

This car doesn't need a gas tank.

The child method overrides the parent method.




## 18. Importing Classes

As programs grow, classes can be separated into modules.

car.py
class Car:

    def __init__(self, make, model):
        self.make = make
        self.model = model

    def describe(self):
        print(f"{self.make} {self.model}")

Another file can import it.

my_cars.py
from car import Car

my_car = Car("Toyota", "Camry")

my_car.describe()

This keeps projects organized.




## 19. Importing Multiple Classes

If a module contains multiple classes:

class Car:
    pass


class ElectricCar(Car):
    pass

You can import both:

from car import Car, ElectricCar

Then:

my_car = Car("Toyota", "Camry")
my_tesla = ElectricCar("Tesla", "Model 3")




## 20. Importing an Entire Module

You can import the complete module:

import car

Then access the class through the module:

my_car = car.Car("Toyota", "Camry")

This makes it clear where Car came from.




## 21. Python Standard Library

Python comes with many useful modules that don't need to be installed separately.

This collection is called the Python Standard Library.

Random numbers
from random import randint

number = randint(1, 10)

print(number)
Files and directories
from pathlib import Path

path = Path("data.txt")
Dates and times
from datetime import datetime

now = datetime.now()

print(now)

Some useful modules:

random       → random numbers
math         → mathematical functions
datetime     → dates and times
pathlib      → files and directories
json         → JSON data
os           → operating-system functionality
sys          → Python/runtime information
collections  → specialized data structures
re           → regular expressions





## 22. Complete Class Example
class Car:

    def __init__(self, make, model):
        self.make = make
        self.model = model
        self.odometer_reading = 0

    def describe(self):
        print(f"{self.make} {self.model}")

    def read_odometer(self):
        print(f"Odometer: {self.odometer_reading} miles")


class ElectricCar(Car):

    def __init__(self, make, model):
        super().__init__(make, model)
        self.battery_size = 75

    def describe_battery(self):
        print(f"Battery: {self.battery_size} kWh")

Create an instance:

my_tesla = ElectricCar("Tesla", "Model 3")

Inherited methods:

my_tesla.describe()
my_tesla.read_odometer()

Child method:

my_tesla.describe_battery()

Output:

Tesla Model 3
Odometer: 0 miles
Battery: 75 kWh




## 23. Class Cheat Sheet
Class
class Dog:

→ Blueprint for objects.

Instance
my_dog = Dog(...)

→ Object created from a class.

Constructor
def __init__(self):

→ Initializes an instance.

Attribute
self.name

→ Stores data.

Method
def sit(self):

→ Defines behavior.

Inheritance
class ElectricCar(Car):

→ Child inherits from parent.

Parent initialization
super().__init__(...)

→ Calls the parent initializer.

Module
car.py

→ Python file containing reusable code.

Import
from car import Car

→ Makes a class available in another file.






## 24. The Core Class Mental Model
CLASS
  ↓
Blueprint
  ↓
Attributes + Methods
  ↓
Create instance
  ↓
Object

Example:

Dog
│
├── name        ← attribute
├── age         ← attribute
│
├── sit()       ← method
└── roll_over() ← method

Then:

Dog class
   ↓
 ┌───────┬───────┬───────┐
 ↓       ↓       ↓
Dog 1   Dog 2   Dog 3

Each instance has its own data while sharing the class's behavior.




## 25. Important Distinctions
## Attribute vs Method

Attribute
→ stores data

Method
→ performs an action


## Class vs Instance

Class
→ blueprint

Instance
→ object created from the blueprint


## Parent vs Child

Parent class
→ provides reusable functionality

Child class
→ inherits and can extend or override it


## Module vs Class

Module
→ Python file containing reusable code

Class
→ blueprint for creating objects







## 26. Reading From a File

Python can read text files using `Path`.
from pathlib import Path

path = Path("pi_digits.txt")
contents = path.read_text()

print(contents)
Key idea
Path()
  ↓
Locate file
  ↓
read_text()
  ↓
Read contents




## 27. Using Path

pathlib provides a modern way to work with file paths.

from pathlib import Path

path = Path("pi_digits.txt")

path represents the location of the file.

contents = path.read_text()

reads the file contents.




## 28. Reading Line by Line

If we want to process each line separately:

lines = path.read_text().splitlines()

for line in lines:
    print(line)

splitlines() converts the file contents into a list of lines.





## 29. Working With File Contents

read_text() returns a string, so we can use normal string methods.

from pathlib import Path

path = Path("hello.txt")
contents = path.read_text()

print(contents.upper())

Output:

HELLO PYTHON




## 30. Removing Extra Whitespace

Files may contain a newline at the end.

contents = path.read_text().rstrip()

rstrip() removes whitespace from the right side.

Example:

"Hello Python\n"

becomes:

"Hello Python"




## 31. Writing to a File

Use write_text() to write data.

from pathlib import Path

path = Path("programming.txt")

path.write_text("I love Python!")

If the file doesn't exist, Python creates it.



## 32. Writing Multiple Lines

Use \n for a new line.

from pathlib import Path

contents = "Python\nJava\nC++\n"

path = Path("languages.txt")
path.write_text(contents)

The file contains:

Python
Java
C++





## 33. Important write_text() Warning

If the file already exists:

path.write_text("New content")

the previous contents are replaced.

So:

Python
Java
C++

becomes:

New content



## 34. Encoding

You can explicitly specify the encoding.

contents = path.read_text(encoding="utf-8")

and:

path.write_text("Hello", encoding="utf-8")

UTF-8 is a common choice for text files.





## 35. Exceptions

An exception is an error that occurs while a program is running.

Example:

print(5 / 0)

produces:

ZeroDivisionError

Another example:

int("hello")

produces:

ValueError

And:

numbers = [1, 2, 3]
print(numbers[10])

produces:

IndexError






## 36. try and except

We can handle exceptions using try and except.

try:
    # code that might fail
except:
    # handle the error

Example:

try:
    print(5 / 0)
except ZeroDivisionError:
    print("You can't divide by zero.")

Instead of crashing, the program handles the error.



## 37. Handling ValueError

Suppose the user must enter a number:

age = int(input("How old are you? "))

If they enter:

twenty

Python raises ValueError.

We can handle it:

try:
    age = int(input("How old are you? "))
except ValueError:
    print("Please enter a number.")





## 38. else With Exceptions

else runs only when no exception occurs.

try:
    age = int(input("How old are you? "))
except ValueError:
    print("Please enter a number.")
else:
    print(f"You are {age} years old.")

Flow:

try
 ↓
Success? ── YES → else
   │
   NO
   ↓
except




## 39. finally

finally runs whether an exception occurs or not.

try:
    print(10 / 2)
except ZeroDivisionError:
    print("Can't divide by zero.")
finally:
    print("Finished.")

Output:

5.0
Finished.

finally is useful for operations that must happen regardless of success or failure, such as cleanup.




## 40. Handling File Errors

Trying to read a missing file produces FileNotFoundError.

from pathlib import Path

path = Path("missing.txt")

try:
    contents = path.read_text()
except FileNotFoundError:
    print("The file doesn't exist.")

Now the program handles the error instead of crashing.




## 41. pass

Sometimes we intentionally want to ignore an error.

try:
    contents = path.read_text()
except FileNotFoundError:
    pass

pass means:

Do nothing.

Be careful with this because silently ignoring errors can make debugging harder.




## 42. Storing Data

Programs often need data to survive after the program ends.

Program running
      ↓
User enters data
      ↓
Program closes
      ↓
Data still exists

A simple approach is storing data in a file.

A common format for structured data is JSON.



## 43. JSON

JSON stands for:

JavaScript Object Notation

It is commonly used for storing and exchanging structured data.

Example:

{
    "name": "Nithish",
    "age": 21,
    "skills": ["Python", "Machine Learning"]
}

Python provides the json module:

import json





## 44. Python Dictionary → JSON

Suppose we have:

import json

user = {
    "name": "Nithish",
    "age": 21,
    "skills": ["Python", "ML"]
}

Convert it to a JSON string:

json_data = json.dumps(user)

print(json_data)




## 45. Saving JSON to a File
from pathlib import Path
import json

user = {
    "name": "Nithish",
    "age": 21,
    "skills": ["Python", "ML"]
}

path = Path("user.json")

contents = json.dumps(user)
path.write_text(contents)

Now the data is stored in user.json.





## 46. Reading JSON Data
from pathlib import Path
import json

path = Path("user.json")

contents = path.read_text()
user = json.loads(contents)

print(user)

Flow:

JSON file
   ↓
read_text()
   ↓
JSON string
   ↓
json.loads()
   ↓
Python dictionary




## 47. Important JSON Functions

There are four important functions:

Function	Purpose
json.dumps()	Python object → JSON string
json.loads()	JSON string → Python object
json.dump()	Python object → JSON file
json.load()	JSON file → Python object
Memory trick
dumps → string
loads → string

dump → file
load → file



## 48. Practical JSON Example

Save a favorite number:

from pathlib import Path
import json

number = 42

path = Path("favorite_number.json")

contents = json.dumps(number)
path.write_text(contents)

Later, read it:

contents = path.read_text()
number = json.loads(contents)

print(f"Your favorite number is {number}.")

Output:

Your favorite number is 42.

The value survives after the program ends because it was stored in a file.





## 49. Combining Exceptions + JSON

Real programs often combine file handling, JSON, and exceptions.

from pathlib import Path
import json

path = Path("user.json")

try:
    contents = path.read_text()
    user = json.loads(contents)
except FileNotFoundError:
    user = {
        "name": "Nithish",
        "age": 21
    }

    path.write_text(json.dumps(user))
else:
    print("Loaded saved data.")

print(user)

This pattern allows a program to:

Try loading saved data
Handle a missing file
Create default data
Save the data
Reuse it later




## 50. File + Exception Cheat Sheet
Read
from pathlib import Path

path = Path("data.txt")
contents = path.read_text()
Read lines
lines = path.read_text().splitlines()
Write
path.write_text("Hello")
Handle errors
try:
    ...
except SomeError:
    ...
Run only on success
try:
    ...
except:
    ...
else:
    ...
Always execute
finally:
    ...
JSON
json.dumps()
json.loads()
json.dump()
json.load()





## 51. Why Testing?

Suppose we have:

def get_formatted_name(first, last):
    return f"{first} {last}"

We could manually test it:

print(get_formatted_name("Nithish", "Ch"))

But imagine having 100 functions.

Manually checking everything becomes inefficient.

Automated tests allow us to quickly verify that existing functionality still works after code changes.





## 52. unittest

Python provides a built-in testing framework called unittest.

import unittest

No additional installation is required.

A test class usually inherits from:

unittest.TestCase

Example:

class NamesTestCase(unittest.TestCase):
    ...




## 53. Testing a Function

Suppose we have:

name_function.py
def get_formatted_name(first, last):
    return f"{first} {last}"

We can create:

test_name_function.py
import unittest

from name_function import get_formatted_name


class NamesTestCase(unittest.TestCase):

    def test_first_last_name(self):
        result = get_formatted_name("Nithish", "Ch")
        self.assertEqual(result, "Nithish Ch")

If the test passes:

.
----------------------------------------------------------------------
Ran 1 test

OK

OK means the test passed.




## 54. Understanding a Test
Import unittest
import unittest

Provides Python's testing tools.

Import the function
from name_function import get_formatted_name

This is the function we want to test.

Create a test class
class NamesTestCase(unittest.TestCase):

This gives the class testing functionality.

Create a test method
def test_first_last_name(self):

Test methods normally start with:

test_

This tells unittest that the method is a test.




## 55. assertEqual()

One of the most important assertions is:

self.assertEqual(actual, expected)

Example:

self.assertEqual(result, "Nithish Ch")

The test checks:

Actual result
     ↓
"Nithish Ch"

Expected result
     ↓
"Nithish Ch"

     ↓

Equal?
  ↓
 PASS

If the values are different, the test fails.





## 56. A Failing Test

Suppose we write:

self.assertEqual(result, "Nithish Kumar")

But the function returns:

Nithish Ch

The test fails.

This is useful because the test tells us that the actual behavior doesn't match our expectation.



## 57. Testing Multiple Cases

We can test different inputs:

class NamesTestCase(unittest.TestCase):

    def test_first_last_name(self):
        result = get_formatted_name("Nithish", "Ch")
        self.assertEqual(result, "Nithish Ch")

    def test_another_name(self):
        result = get_formatted_name("Rahul", "Kumar")
        self.assertEqual(result, "Rahul Kumar")

Now we have two independent tests.



## 58. Testing Variations

Suppose our function supports an optional middle name:

def get_formatted_name(first, last, middle=""):
    if middle:
        return f"{first} {middle} {last}"
    else:
        return f"{first} {last}"

We should test both cases:

def test_first_last_name(self):
    result = get_formatted_name("Nithish", "Ch")
    self.assertEqual(result, "Nithish Ch")

def test_first_middle_last_name(self):
    result = get_formatted_name("Nithish", "Ch", "Kumar")
    self.assertEqual(result, "Nithish Kumar Ch")
Testing principle

Test the normal case and important variations.




## 59. Testing a Class

We can also test classes.

Example:

class AnonymousSurvey:

    def __init__(self, question):
        self.question = question
        self.responses = []

    def show_question(self):
        print(self.question)

    def store_response(self, new_response):
        self.responses.append(new_response)

We want to verify that responses are stored correctly.




## 60. Creating a Class Test
import unittest

from survey import AnonymousSurvey


class TestAnonymousSurvey(unittest.TestCase):

    def test_store_single_response(self):
        question = "What language did you first learn?"

        survey = AnonymousSurvey(question)

        survey.store_response("Python")

        self.assertIn("Python", survey.responses)

This checks whether "Python" was successfully stored.




## 61. assertIn()

assertIn() checks whether a value exists inside a collection.

self.assertIn("Python", survey.responses)

For example:

survey.responses = ["Python"]

Then:

"Python" ∈ survey.responses

is true, so the test passes.



## 62. Testing Multiple Responses

We can test several responses:

def test_store_three_responses(self):
    question = "What language did you first learn?"

    survey = AnonymousSurvey(question)

    responses = ["Python", "Java", "C++"]

    for response in responses:
        survey.store_response(response)

    for response in responses:
        self.assertIn(response, survey.responses)

This verifies that all three responses were stored.



## 63. setUp()

When multiple tests need the same setup, we can use setUp().

Instead of repeatedly creating:

survey = AnonymousSurvey(question)

we can prepare it once per test.

class TestAnonymousSurvey(unittest.TestCase):

    def setUp(self):
        question = "What language did you first learn?"
        self.survey = AnonymousSurvey(question)

    def test_store_single_response(self):
        self.survey.store_response("Python")

        self.assertIn("Python", self.survey.responses)




## 64. How setUp() Works

setUp() runs before each test method.

Test 1
  ↓
setUp()
  ↓
test_store_single_response()


Test 2
  ↓
setUp()
  ↓
test_store_three_responses()

Each test receives a fresh object.

This helps keep tests independent from each other.




## 65. Important Assertion Methods
assertEqual()

Checks equality.

self.assertEqual(actual, expected)

Example:

self.assertEqual(2 + 2, 4)
assertNotEqual()

Checks that values are different.

self.assertNotEqual(2 + 2, 5)
assertTrue()

Checks whether something is True.

self.assertTrue(age >= 18)
assertFalse()

Checks whether something is False.

self.assertFalse(age < 18)
assertIn()

Checks whether an item exists.

self.assertIn("Python", languages)
assertNotIn()

Checks whether an item does not exist.

self.assertNotIn("Ruby", languages)
assertIsNone()

Checks for None.

self.assertIsNone(result)
assertIsNotNone()

Checks that a value isn't None.

self.assertIsNotNone(result)




## 66. Test Naming

Test names should clearly describe what they test.

Good:

def test_store_single_response(self):

Good:

def test_user_can_add_item(self):

Less useful:

def test1(self):

A good test name should tell us what behavior is being tested.



## 67. Why Testing Matters

Consider an AI application:

Data preprocessing
      ↓
Feature engineering
      ↓
Model
      ↓
Prediction
      ↓
API

If we change preprocessing, we might accidentally break another part of the system.

With automated tests:

Change code
    ↓
Run tests
    ↓
PASS → behavior still works
FAIL → investigate

Testing gives us confidence when modifying code.





## 68. Good Testing Principles

A good test should generally:

1. Test one behavior
def test_store_single_response(self):

Focus on one specific behavior.

2. Be repeatable

Running the test multiple times should give consistent results.

3. Be independent

One test shouldn't depend on another test.

4. Be clear

When a test fails, it should be easy to understand what behavior broke.





## 69. Complete Mental Model
                  TESTING
                     │
          ┌──────────┴──────────┐
          ↓                     ↓
     Test function          Test class
          ↓                     ↓
      unittest              unittest
          ↓                     ↓
      TestCase              TestCase
          ↓                     ↓
   test_method()             setUp()
          ↓                     ↓
   assertions              test_method()
                                ↓
                            assertions






## 70. Core Syntax
import unittest


class MyTestCase(unittest.TestCase):

    def setUp(self):
        # prepare test data
        pass

    def test_something(self):
        # run code
        # check result
        self.assertEqual(actual, expected)