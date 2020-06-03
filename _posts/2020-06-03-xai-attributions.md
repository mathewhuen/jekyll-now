---
layout: post
title: XAI - Attributions
---

One of the most complex topics in XAI is attributions--calculating how much of an effect specific inputs to a model have on its outputs. For example, we would expect that certain pixels were more important than others in a classification of an image. In the image below, it is likely that pixels from the puppy are more important than pixels from the background. Attributions is the precise calculation of this importance. 

### Why?
Why is this important? Non-interpretable models like deep neural networks are blackboxes--we can see their predictions, but not the reasons why. ...

### Motivation

So how are attributions calculated? Let's start with an abstract example. Suppose that a teacher decides to give their students a group test. The classroom is divided into groups of four and a four-question test is given. Everyone in the same group recieves the same score. However, after the tests are collected, the teacher announces that four extra-credit points will be given to each group and that it is up to the students to decide how these points will be distributed within the group (eg: each student receives one point, the student who did the most will receive all four points, etc).

How can the points be fairly distributed?

Whoever contributed the most should get the most points, right? It seems fair that bonus points received should be proportional to individual contribution to the group score. Let's calculate some examples.

1 Suppose each student answered exactly one problem. Then it's easy, each student should receive one point:

| Student\Problem | P1 | P2 | P3 | P4 | Bonus |
|:-------|:---------:|:-------:|:-------:|:-------:|:---------:|
| Student A | O | - | - | - | 1 |
| Student B | - | O | - | - | 1 |
| Student C | - | - | O | - | 1 |
| Student D | - | - | - | O | 1 |


2 Suppose one student knew zero answers, one knew two answers, and two knew exactly one answer. Again, this is easy--students can receive points equal to the number of problems they answered:

| Student\Problem | P1 | P2 | P3 | P4 | Bonus |
|:-------|:---------:|:-------:|:-------:|:-------:|:----------:|
| Student A | O | - | - | - | 1 |
| Student B | - | O | - | - | 1 |
| Student C | - | - | - | - | 0 |
| Student D | - | - | O | O | 2 |

So far, not so hard. But let's formalize this logic. For each student, we look at the number of correct answers the group could get with that student missing and calculate how many more questions they could answer when we add that student in. This seems to give the marginal contribution of that student. If f(x) is the score for group x, then the marginal value of student s in group x is M(x) = f(x)-f(x/s). For example 2 above, M(D) = f({A,B,C,D}) - f({A,B,C}) = 4 - 2 = 2. Thus, Student D should receive 2 points.

However, things get a little more tricky if multiple students can answer the same question. Suppose student A can answer question 1, student B doesn't know any answers, and both students C and D can answer problems 2, 3, and 4:

| Student\Problem | P1 | P2 | P3 | P4 | Bonus |
|:-------|:---------:|:-------:|:-------:|:-------:|:----------:|
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

M(A) = (f({A,B,C,D}) - f({B,C,D}) + f({A})) / 2 = (4 - 3 + 1) / 2 = 2/2 = 1

M(B) = (f({A,B,C,D}) - f({A,C,D}) + f({B})) / 2 = (4 - 4 + 0) / 2 = 0/2 = 0

M(C) = (f({A,B,C,D}) - f({A,B,D}) + f({C})) / 2 = (4 - 4 + 3) / 2 = 3/2 = 1.5

M(D) = (f({A,B,C,D}) - f({A,B,C}) + f({D})) / 2 = (4 - 4 + 3) / 2 = 3/2 = 1.5

Great! Now the bonus points seem better distributed. However, this is just hinting at the mathematically "fair" solution (indeed, if answer overlaps are more complicated, the above method will quickly fall short).

Given student s in group x, our current formulation for M(s) is

M(s) = (f(x) - f(x/s)  +  f({s})) / 2

This is equal to 

M(s) = (f(x) - f(x/s)  +  f({s}) - f({})) / 2 = (f(x) - f(x/s)  +  f({s}) - f({s}/s)) / 2

since we can say the score assigned to a group of no students is zero. Do you see a pattern? We are looking at two subgroups (x/s and the empty subgroup) of the original group x that are missing s, and calculating the average score difference when s is added to those subgroups. The complete answer is to calculate this average over all subgroups missing s (the power set of x/s).

M(s) = 1/n * Sigma({f(y) : y in power(x/s)}), where n is the size of the power set of x/s

This formulation is called the Shapley Value! It was originally from game theory and was used to calculate "fair" contributions of individuals to a group score. However, treating model features as individual players (such as pixels in an image), we can calculate contributions of each feature to the output of a model for individual data samples. 

### Shapley Implementations

That's a pretty powerful statement. However, actually applying it to a machine learning model can be problematic. 

The first problem is that the power set is huge. If your model has 5 features, then for a given data sample, we can calculate the feature-wise attribution. This means generating (5*2^4) = 80 sets and calculating scores for each. Not terrible, but large models can have more than a hundred or thousand inputs. An 224 x 224 rgb image has 150,528 pixels. That's just way too many sets to even consider. Fortunately, there are several sampling and cross-feature set sharing methods we can use to reduce this to a manageable size.

The second problem is that there is usually no easy way to calculate model outputs for variable input sizes. Calculating f(x) for the entire input x is fine, but how do we calculate f(x/s) for the input with one feature removed? The most common solution is to sample from a training dataset for the feature s, average all the scores together, and use that average as a representative of f(x/s). If each feature is linearly independent, this is a mathematically sound process. However, feature independence is one of the most commonly abused assumptions in machine learning (right up there with assumptions about data being normal distributed). If features are not independent, then this sampling can create unrealistic feature combinations that shouldn't be considered in an attribution calculation. Furthermore, you may sample from an untrained part of the feature space and the model may behave erratically (though some may argue this is not actually a drawback as we are seeking to explain the whole model, regardless of how well it is trained--regardless, unrealistic data points are a problem). 

How can we fix this? There are several sampling techniques I've used to get around the second problem above. Unfortunately, I do not have ownership of those techniques, but if you are interested in using Shapley, I suggest being creative about data generation or strict about feature independence.

For an implementation reference (and a bunch of great explanations of various xai concepts), please see Christoph Molnar's online book on interpretable machine learning https://christophm.github.io/interpretable-ml-book/cnn-features.html  This was my first exposure to XAI and a great resource.

Though, the above problems can be reduced with sampling, shapley is still slow and can't be vectorized for faster batch calculations. As an alternative, there are many attribution methods available. Some popular ones are LIME, DeepLIFT, Integrated Gradients, and GradCAM/GradCAM+ (see below for a more exhaustive list). LIME is model agnostic and works by fitting an explanative model (like SVM/tree-based models) locally around data points to be explained. There are many other model agnostic, surrogate models (eg, bayesian logistic models), but LIME seems to be the most common (see also Skater which incorporates several other techniques). DeepLIFT, Integrated Gradients, and GradCAM are all gradient based models and work great with neural networks. GradCAM is specifically designed for CNNs, however. DeepLIFT and Integrated Gradients are unique in that they are approximations to the above shapley value. And though I personally love DeepLIFT, it is not implementation agnostic and it deviates from shapley when recurrent units are utilized. So my prefered method for neural networks (or any method that allows backpropogation of gradients) is Integrated Gradients.

### Integrated Gradients

(more to come) 
