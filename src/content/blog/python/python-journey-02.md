---
title: "Python Journey #2 - Lists"
description: "Learning Python lists, indexing, modifying lists, adding and removing elements, sorting, and common list operations."
date: "2026-09-11"
category: "Python"
tags: ["Python", "Lists", "Indexing", "List Methods"]
---

# Python Journey #2 - Lists


```python
A **list** is an ordered and changeable collection of values.

## 1. Creating a List

Lists use square brackets `[]`.

cars = ["BMW", "Audi", "Tesla"]

Each value is called an element.





## 2. List Indexing

Python uses zero-based indexing.

BMW    → 0
Audi   → 1
Tesla  → 2

Access an element using its index:

print(cars[0])

Output:

BMW







## 3. Changing Elements

Lists are mutable, meaning their elements can be changed.

cars = ["BMW", "Audi", "Tesla"]

cars[1] = "Mercedes"

print(cars)

Output:

['BMW', 'Mercedes', 'Tesla']








## 4. Adding Elements

4.1 append()

Adds an element to the end.

cars.append("Toyota")


4.2 insert()

Adds an element at a specific position.

cars.insert(1, "Honda")






## 5. Removing Elements


5.1 del

Removes an element using its index.

del cars[1]



5.2 pop()

Removes and returns an element.

car = cars.pop()

By default, it removes the last element.




5.3 remove()

Removes an element using its value.

cars.remove("Audi")
Remember
del      → remove by index
pop()    → remove and return
remove() → remove by value



## 6. Sorting Lists


6.1 sort()

Changes the original list.

cars.sort()

Reverse sorting:

cars.sort(reverse=True)



6.2 sorted()

Returns a sorted version without changing the original list.

sorted(cars)
Remember
sort()   → changes the original list
sorted() → returns a sorted version






## 7. Reversing a List

reverse() reverses the current order.

cars.reverse()

It does not mean alphabetical reverse sorting.




## 8. Finding the Length

Use len() to find the number of elements.

cars = ["BMW", "Audi", "Tesla"]

print(len(cars))

Output:

3






## 9. Negative Indexing

Python allows negative indexes to access elements from the end.

BMW    → -3
Audi   → -2
Tesla  → -1

For example:

print(cars[-1])

Output:

Tesla







## 10. Index Errors

If a list contains 3 elements, the valid indexes are:

0, 1, 2

This causes an error:

print(cars[3])

because index 3 does not exist.

The error is:

IndexError






## 11. Important List Methods
Method	Purpose
append()	Add to end
insert()	Add at a position
remove()	Remove by value
pop()	Remove and return
sort()	Sort the list
reverse()	Reverse the list