---
layout: post
title: XAI - Attributions
---

One of the most complex topics in XAI is attributions--calculating how much of an effect specific inputs to a model have on its outputs. For example, we would expect that certain pixels were more important than others in a classification of an image. In the image below, it is likely that pixels from the puppy are more important than pixels from the background. Attributions is the precise calculation of this importance. 

Why is this important? Non-interpretable models like deep neural networks are blackboxes--we can see their predictions, but not the reasons why. ...

So how are attributions calculated? Let's start with an abstract example. Suppose that a teacher decides to give their students a group test. The classroom is divided into groups of four and a four-question test is given. Everyone in the same group recieves the same score. However, after the tests are collected, the teacher announces that four extra-credit points will be given to each group and that it is up to the students to decide how these points will be distributed within the group (eg: each student receives one point, the student who did the most will receive all four points, etc).

How can the points be fairly distributed?

Whoever contributed the most should get the most points, right? It seems fair that bonus points received should be proportional to individual contribution to the group score. Let's calculate some examples.

Suppose each student answered exactly one problem. Then it's easy, each student should receive one point:
| Student # | Problem 1 | Problem 2 | Problem 3 | Problem 4 | Bonus |
|-------|--------|---------|-------|-------|-------|
| Student A | O | - | - | - | 1 |
| Student B | - | O | - | - | 1 |
| Student C | - | - | O | - | 1 |
| Student D | - | - | - | O | 1 |


Suppose one student knew zero answers, one knew two answers, and two knew exactly one answer. Again, this is easy--students can receive points equal to the number of problems they answered:
| Student # | Problem 1 | Problem 2 | Problem 3 | Problem 4 | Bonus |
|-------|--------|---------|-------|-------|-------|
| Student A | O | - | - | - | 1 |
| Student B | - | O | - | - | 1 |
| Student C | - | - | - | - | 0 |
| Student D | - | - | O | O | 2 |



