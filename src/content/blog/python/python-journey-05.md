---
title: "Python Journey #5 - Dictionaries"
description: "Learning Python dictionaries, key-value pairs, modifying data, looping, membership checks, and nested data structures."
date: "2026-09-12"
category: "Python"
tags: ["Python", "Dictionaries", "Key-Value", "Nesting"]
---

# Python Journey #5 - Dictionaries

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