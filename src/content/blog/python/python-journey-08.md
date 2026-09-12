---
title: "Python Journey #8 - Classes"
description: "Learning Python classes, objects, attributes, methods, inheritance, modules, and the standard library."
date: "2026-09-12"
category: "Python"
tags: ["Python", "Classes", "Objects", "OOP", "Inheritance", "Modules"]
---

# Python Journey #8 - Classes

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