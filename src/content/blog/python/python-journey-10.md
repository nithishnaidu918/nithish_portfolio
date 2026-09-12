---
title: "Python Journey #10 - Testing Your Code"
description: "Learning how to test Python functions and classes using unittest, assertions, and setUp()."
date: 2026-09-12
category: "Python"
tags: ["Python", "Testing", "unittest", "Unit Testing"]
---

# Python Journey #10 - Testing Your Code



```text
Testing helps us verify that our code behaves as expected.

Instead of manually checking everything after every change, we can use **automated tests**.

Code
 ↓
Test
 ↓
PASS / FAIL








## 1. Why Testing?

Suppose we have:

def get_formatted_name(first, last):
    return f"{first} {last}"

We could manually test it:

print(get_formatted_name("Nithish", "Ch"))

But imagine having 100 functions.

Manually checking everything becomes inefficient.

Automated tests allow us to quickly verify that existing functionality still works after code changes.











## 2. unittest

Python provides a built-in testing framework called unittest.

import unittest

No additional installation is required.

A test class usually inherits from:

unittest.TestCase

Example:

class NamesTestCase(unittest.TestCase):
    ...











## 3. Testing a Function

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










## 4. Understanding a Test
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












## 5. assertEqual()

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











## 6. A Failing Test

Suppose we write:

self.assertEqual(result, "Nithish Kumar")

But the function returns:

Nithish Ch

The test fails.

This is useful because the test tells us that the actual behavior doesn't match our expectation.










## 7. Testing Multiple Cases

We can test different inputs:

class NamesTestCase(unittest.TestCase):

    def test_first_last_name(self):
        result = get_formatted_name("Nithish", "Ch")
        self.assertEqual(result, "Nithish Ch")

    def test_another_name(self):
        result = get_formatted_name("Rahul", "Kumar")
        self.assertEqual(result, "Rahul Kumar")

Now we have two independent tests.











## 8. Testing Variations

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











## 9. Testing a Class

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












## 10. Creating a Class Test
import unittest

from survey import AnonymousSurvey


class TestAnonymousSurvey(unittest.TestCase):

    def test_store_single_response(self):
        question = "What language did you first learn?"

        survey = AnonymousSurvey(question)

        survey.store_response("Python")

        self.assertIn("Python", survey.responses)

This checks whether "Python" was successfully stored.











## 11. assertIn()

assertIn() checks whether a value exists inside a collection.

self.assertIn("Python", survey.responses)

For example:

survey.responses = ["Python"]

Then:

"Python" ∈ survey.responses

is true, so the test passes.











## 12. Testing Multiple Responses

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










## 13. setUp()

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










## 14. How setUp() Works

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











## 15. Important Assertion Methods
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












## 16. Test Naming

Test names should clearly describe what they test.

Good:

def test_store_single_response(self):

Good:

def test_user_can_add_item(self):

Less useful:

def test1(self):

A good test name should tell us what behavior is being tested.











## 17. Why Testing Matters

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













## 18. Good Testing Principles

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










## 19. Complete Mental Model
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









## 20. Core Syntax
import unittest


class MyTestCase(unittest.TestCase):

    def setUp(self):
        # prepare test data
        pass

    def test_something(self):
        # run code
        # check result
        self.assertEqual(actual, expected)