---
title: "Machine Learning Journey #1 - Understanding the Foundations"

description: "Learning the core ideas behind Machine Learning, including models, features, targets, training, inference, supervised learning, and generalization."

date: "2026-09-14"

category: "Machine Learning"

tags: ["Machine Learning", "ML", "AI", "Fundamentals"]
---

# Machine Learning Journey #1

Machine Learning is the foundation of modern AI.

In this journey, I focused on understanding **what Machine Learning actually is, how models learn from data, and how ML problems are classified.**

---

## 1. What is Machine Learning?

Machine Learning allows computers to learn patterns from data and use those patterns to make predictions or decisions.

Instead of writing every rule manually, we provide examples and allow an algorithm to learn from them.





## 2. AI vs Machine Learning vs Deep Learning

These three concepts are related but not the same.

Artificial Intelligence
        ↓
Machine Learning
        ↓
Deep Learning




Artificial Intelligence

AI is the broad field of building systems that perform tasks that appear intelligent.

Examples:

Reasoning
Planning
Perception
Language understanding
Decision making




Machine Learning

ML is a subset of AI where systems learn patterns from data.




Deep Learning

Deep Learning is a subset of ML that primarily uses multi-layer neural networks.

Examples:
Computer Vision
Speech Recognition
Large Language Models
Generative AI








## 3. Traditional Programming vs Machine Learning
Traditional Programming
Rules + Data
     ↓
  Program
     ↓
  Output

The programmer explicitly defines the rules.

Example:

if temperature > 30:
    print("Hot")
else:
    print("Not Hot")


Machine Learning
Data + Expected Output
          ↓
      ML Algorithm
          ↓
         Model

The algorithm learns patterns from examples.

This difference is one of the most important foundations of ML.









## 4. What is a Model?

A model is the learned relationship produced from training data.

The basic idea is:

Training Data
     ↓
Learning Algorithm
     ↓
Trained Model

For example, a simple linear regression model can be represented as:

y = wx + b

Here:

w = learned parameter
b = learned parameter
x = input
y = prediction

The parameters are learned during training.







## 5. Features and Target

Consider:

Area	Bedrooms	Age	Price
1000	2	10	₹40L
1500	3	5	₹65L
2000	3	3	₹90L


Features
The inputs used by the model:
Area
Bedrooms
Age

Usually represented as:
X


Target
The value we want to predict:
Price

Usually represented as:
y

Therefore:

X → y
Remember
Features = Inputs
Target   = What we want to predict










## 6. Training

Training is the process of finding model parameters that allow the model to perform well on the training data.

A simplified training loop looks like this:

Input
  ↓
Model
  ↓
Prediction
  ↓
Compare with Actual Value
  ↓
Loss
  ↓
Update Model
  ↓
Repeat

The model produces a prediction:
ŷ

The actual value is:
y

The difference between them is used to calculate a loss.










## 7. Training vs Inference

These are two different stages.



Training
The model learns from data.

Data
 ↓
Learning
 ↓
Model



Inference
The trained model makes predictions on new data.

New Data
   ↓
Trained Model
   ↓
Prediction


Simple distinction
Training  → Model learns

Inference → Model predicts









## 8. Types of Machine Learning

The major categories are:

Machine Learning
│
├── Supervised Learning
├── Unsupervised Learning
├── Semi-Supervised Learning
├── Self-Supervised Learning
└── Reinforcement Learning

For classical Machine Learning, the two categories I need to understand first are:

Supervised Learning
Unsupervised Learning









## 9. Supervised Learning

In supervised learning, we have:
Input Data + Correct Answers

Example:

Hours Studied	Result
2	Fail
4	Pass
6	Pass
8	Pass

The model learns the relationship:

X → y

where y is known during training.

Supervised learning mainly contains:

Supervised Learning
       │
       ├── Regression
       │
       └── Classification










## 10. Regression

Regression is used when we want to predict a continuous numerical value.

Examples:
House price
Salary
Temperature
Sales
Delivery time








## 11. Classification

Classification is used when we want to predict a category.

Examples:
Spam / Not Spam
Fraud / Not Fraud
Cat / Dog
Pass / Fail








## 12. Regression vs Classification

This distinction should become automatic.

Problem	                  Type
Predict house price   	Regression
Predict salary	        Regression
Predict temperature  	Regression
Detect spam	            Classification
Detect fraud        	Classification
Predict pass/fail   	Classification

The simplest rule:

Numerical value → Regression

Category/Class → Classification









## 13. Unsupervised Learning

In unsupervised learning, we don't have target labels.
The model tries to discover useful patterns or structure in the data.

Important unsupervised learning tasks include:


Clustering
Finding groups in data.

Examples:
K-Means
DBSCAN
Hierarchical Clustering
Dimensionality Reduction


Reducing the number of features while preserving useful information.

Examples:
PCA
t-SNE
UMAP
Anomaly Detection


Finding unusual observations.

Examples:
Fraud detection
Network attacks
Unusual transactions










## 14. Generalization

A model shouldn't simply memorize the training data.

The real goal is:

Learn useful patterns from training data and perform well on unseen data.

This is called generalization.

Generalization is one of the central goals of Machine Learning.








## 15. Overfitting

Overfitting happens when a model learns the training data too specifically.

For example:

Training Performance → Excellent
Test Performance     → Poor

The model may have memorized the training examples instead of learning patterns that generalize.









## 16. Underfitting

Underfitting happens when the model is too simple to capture the important patterns.

Training Performance → Poor
Test Performance     → Poor








## 17. How to Formulate an ML Problem

Before choosing an algorithm, ask:

Question 1

What am I trying to predict or discover?

Question 2

Do I have labels?

Labels?
 /    \
Yes    No
 ↓      ↓
Supervised
         Unsupervised

If supervised, ask:

Question 3

What type of target do I have?

Target
 /    \
Number Category
  ↓       ↓
Regression Classification
Example

Predict employee salary:

Features
experience
education
skills
location

        ↓

Target
salary

Salary is numerical.

Therefore:

Supervised
    ↓
Regression










## 18. The Complete ML Picture

This is the roadmap I want to keep in mind as I continue learning:

Problem
   ↓
Data
   ↓
Problem Formulation
   ↓
Data Preparation
   ↓
Feature Engineering
   ↓
Model Selection
   ↓
Training
   ↓
Validation
   ↓
Evaluation
   ↓
Hyperparameter Tuning
   ↓
Deployment
   ↓
Monitoring

