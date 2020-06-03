---
layout: post
title: XAI - Attributions
---

One of the most complex topics in XAI is attributions--calculating how much of an effect specific inputs to a model have on its outputs. For example, we would expect that certain pixels were more important than others in a classification of an image. In the image below, it is likely that pixels from the puppy are more important than pixels from the background. Attributions is the precise calculation of this importance. 

Why is this important? Non-interpretable models like deep neural networks are blackboxes--we can see their predictions, but not the reasons why. ...

So how are attributions calculated? Let's start with an abstract example. Suppose that a teacher decides to give their students a group test. The classroom is divided into groups of four and a four-question test is given. Everyone in the same group recieves the same score. However, after the tests are collected, the teacher announces that four extra-credit points will be given to each group and that it is up to the students to decide how these points will be distributed within the group (eg: each student receives one point, the student who did the most will receive all four points, etc).

How can the points be fairly distributed?

Whoever contributed the most should get the most points, right? It seems fair that bonus points received should be proportional to individual contribution to the group score. Let's calculate some examples.

1 Suppose each student answered exactly one problem. Then it's easy, each student should receive one point:
| Student # | Problem 1 | Problem 2 | Problem 3 | Problem 4 | Bonus |
|-------|--------|---------|-------|-------|-------|
| Student A | O | - | - | - | 1 |
| Student B | - | O | - | - | 1 |
| Student C | - | - | O | - | 1 |
| Student D | - | - | - | O | 1 |


2 Suppose one student knew zero answers, one knew two answers, and two knew exactly one answer. Again, this is easy--students can receive points equal to the number of problems they answered:
| Student # | Problem 1 | Problem 2 | Problem 3 | Problem 4 | Bonus |
|-------|--------|---------|-------|-------|-------|
| Student A | O | - | - | - | 1 |
| Student B | - | O | - | - | 1 |
| Student C | - | - | - | - | 0 |
| Student D | - | - | O | O | 2 |

So far, not so hard. But let's formalize this logic. For each student, we look at the number of correct answers the group could get with that student missing and calculate how many more questions they could answer when we add that student in. This seems to give the marginal contribution of that student. If f(x) is the score for group x, then the marginal value of student s in group x is M(x) = f(x)-f(x/s). For example 2 above, M(D) = f({A,B,C,D}) - f({A,B,C}) = 4 - 2 = 2. Thus, Student D should receive 2 points.

However, things get a little more tricky if multiple students can answer the same question. Suppose student A can answer question 1, student B doesn't know any answers, and both students C and D can answer problems 2, 3, and 4:
| Student # | Problem 1 | Problem 2 | Problem 3 | Problem 4 | Bonus |
|-------|--------|---------|-------|-------|-------|
| Student A | O | - | - | - | ? |
| Student B | - | - | - | - | ? |
| Student C | - | O | O | O | ? |
| Student D | - | O | O | O | ? |
Using the above logic, 
M(A) = f({A,B,C,D}) - f({B,C,D}) = 4 - 3 = 1
M(B) = f({A,B,C,D}) - f({A,C,D}) = 4 - 4 = 0
M(C) = f({A,B,C,D}) - f({A,B,D}) = 4 - 4 = 0
M(D) = f({A,B,C,D}) - f({A,B,C}) = 4 - 4 = 0
Well this seems wrong for two reasons:
* The two most knowledgable students both get zeros bonus points
* Intuitively, it would be nice if the sum of all points added up to 4 (this property is called completeness and we will explain more about it later)

The obvious fix would be to look at individual scores as well--since student C and D both can answer the same three questions by themselves, they should both share credit. This is just averaging the above marginal score M with individual scores:
M(C) = (f({A,B,C,D}) - f({B,C,D}) + f({A})) / 2 = (4 - 3 + 1) / 2 = 2/2 = 1
M(C) = (f({A,B,C,D}) - f({A,C,D}) + f({B})) / 2 = (4 - 4 + 0) / 2 = 0/2 = 0
M(C) = (f({A,B,C,D}) - f({A,B,D}) + f({C})) / 2 = (4 - 4 + 3) / 2 = 3/2 = 1.5
M(C) = (f({A,B,C,D}) - f({A,B,C}) + f({D})) / 2 = (4 - 4 + 3) / 2 = 3/2 = 1.5
Great! Now the bonus points seem more fair.



