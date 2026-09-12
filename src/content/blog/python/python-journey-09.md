---
title: "Python Journey #9 - Files, Exceptions & JSON"
description: "Learning how Python reads and writes files, handles exceptions, and stores persistent data using JSON."
date: 2026-09-12
category: "Python"
tags: ["Python", "Files", "Exceptions", "JSON", "pathlib"]
---

# Python Journey #9 - Files, Exceptions & JSON

This journey covers three important areas:

1. **Files** → Read and write data
2. **Exceptions** → Handle errors safely
3. **JSON** → Store structured data for later use

---



```python
## 1. Reading From a File

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










## 2. Using Path

pathlib provides a modern way to work with file paths.

from pathlib import Path

path = Path("pi_digits.txt")

path represents the location of the file.

contents = path.read_text()

reads the file contents.











## 3. Reading Line by Line

If we want to process each line separately:

lines = path.read_text().splitlines()

for line in lines:
    print(line)

splitlines() converts the file contents into a list of lines.












## 4. Working With File Contents

read_text() returns a string, so we can use normal string methods.

from pathlib import Path

path = Path("hello.txt")
contents = path.read_text()

print(contents.upper())

Output:

HELLO PYTHON












## 5. Removing Extra Whitespace

Files may contain a newline at the end.

contents = path.read_text().rstrip()

rstrip() removes whitespace from the right side.

Example:

"Hello Python\n"

becomes:

"Hello Python"










## 6. Writing to a File

Use write_text() to write data.

from pathlib import Path

path = Path("programming.txt")

path.write_text("I love Python!")

If the file doesn't exist, Python creates it.











## 7. Writing Multiple Lines

Use \n for a new line.

from pathlib import Path

contents = "Python\nJava\nC++\n"

path = Path("languages.txt")
path.write_text(contents)

The file contains:

Python
Java
C++












## 8. Important write_text() Warning

If the file already exists:

path.write_text("New content")

the previous contents are replaced.

So:

Python
Java
C++

becomes:

New content










## 9. Encoding

You can explicitly specify the encoding.

contents = path.read_text(encoding="utf-8")

and:

path.write_text("Hello", encoding="utf-8")

UTF-8 is a common choice for text files.












## 10. Exceptions

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












## 11. try and except

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












## 12. Handling ValueError

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












## 13. else With Exceptions

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











## 14. finally

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














## 15. Handling File Errors

Trying to read a missing file produces FileNotFoundError.

from pathlib import Path

path = Path("missing.txt")

try:
    contents = path.read_text()
except FileNotFoundError:
    print("The file doesn't exist.")

Now the program handles the error instead of crashing.















## 16. pass

Sometimes we intentionally want to ignore an error.

try:
    contents = path.read_text()
except FileNotFoundError:
    pass

pass means:

Do nothing.

Be careful with this because silently ignoring errors can make debugging harder.













## 17. Storing Data

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












## 18. JSON

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












## 19. Python Dictionary → JSON

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













## 20. Saving JSON to a File
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











## 21. Reading JSON Data
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














## 22. Important JSON Functions

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












## 23. Practical JSON Example

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














## 24. Combining Exceptions + JSON

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










## 25. File + Exception Cheat Sheet
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
26. Mental Model
                    FILES
                      │
             ┌────────┴────────┐
             ↓                 ↓
           READ              WRITE
             ↓                 ↓
       read_text()       write_text()
             │                 │
             └────────┬────────┘
                      ↓
                    DATA
                      │
                      ↓
                    JSON
                      │
             ┌────────┴────────┐
             ↓                 ↓
       Python → JSON      JSON → Python
       dumps / dump       loads / load


                    ERRORS
                      ↓
                     try
                      ↓
              ┌───────┴───────┐
              ↓               ↓
           success          failure
              ↓               ↓
            else            except
              └───────┬───────┘
                      ↓
                   finally