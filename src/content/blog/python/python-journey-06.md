---
title: "Python Journey #6 - User Input and while Loops"
description: "Learning user input, type conversion, while loops, flags, break, continue, and processing lists and dictionaries."
date: "2026-09-12"
category: "Python"
tags: ["Python", "Input", "While Loops", "Break", "Continue", "Lists", "Dictionaries"]
---

# Python Journey #6 - User Input and while Loops

Python provides `input()` for getting information from users and `while` loops for repeating code while a condition is `True`.

---



```python
## 1. The `input()` Function

`input()` allows a program to receive information from the user.

name = input("What is your name? ")

print(f"Hello, {name}!")

Example:

What is your name? Nithish
Hello, Nithish!

The text inside input() is called the prompt.











##2. input() Always Returns a String

This is very important.

age = input("How old are you? ")

print(type(age))

If the user enters 21, the result is still:

<class 'str'>

So Python stores:

"21"

not:

21









##3. Converting Input to an Integer

Use int() when you need an integer.

age = int(input("How old are you? "))

print(type(age))

Now the type is:

<class 'int'>

For decimal numbers, use float():

height = float(input("Enter your height: "))










##4. Using Input in Calculations

This causes an error:

age = input("How old are you? ")

print(age + 1)

because age is a string.

Correct:

age = int(input("How old are you? "))

print(age + 1)

If the user enters 21:

22












##5. The while Loop

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














##6. while Loop Structure

Basic syntax:

while condition:
    # code to execute

Example:

number = 1

while number <= 10:
    print(number)
    number += 1

Something inside the loop usually needs to change the condition.












##7. Infinite Loops

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











##8. while with User Input

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












##9. Using a Flag

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










##10. break

break immediately exits a loop.

while True:
    message = input("Enter something: ")

    if message == "quit":
        break

    print(message)

Even though:

while True:

would normally continue forever, break stops it.











##11. continue

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










##12. Processing a List with while

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











##13. Removing All Instances of a Value

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










##14. Using while with Dictionaries

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









##15. A Common Interactive Pattern

A very common Python pattern is:

while True:
    data = input("Enter something: ")

    if data == "quit":
        break

    # process data

This pattern is useful for interactive programs.










##16. for vs while
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









##17. input() Cheat Sheet

Get text:

name = input("Name: ")

Get an integer:

age = int(input("Age: "))

Get a decimal:

height = float(input("Height: "))

Remember:

input() → always returns a string












##18. while Cheat Sheet

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








##19. Important Differences
Concept	Purpose
input()	Get data from the user
int()	Convert to integer
float()	Convert to decimal
while	Repeat while condition is true
break	Exit loop immediately
continue	Skip current iteration
Flag	Boolean variable controlling a loop