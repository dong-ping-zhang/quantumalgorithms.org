---

title: How to evaluate a classifier
comments: true 
status: done
tags: qml, intro
description: When performing a real data analysis is important to understand how good is our algorithm in solving our problem. Here is a short recap of the most used metrics for classifications.
author:
- 'Alessandro “Scinawa” Luongo'
- 'Dong Ping Zhang'

---


Practitioners in quantum machine learning domain do not only benefit from building their
skills in quantum algorithms, but also from having basic knowledge of statistics and data science. In this article, we talk about various methods on how to evaluate a classifier. What does it means in practice? Imagine that we have a medical test that is able to tell if a patient is sick or
not. We might want to consider the behaviour of our classier with
respect to the following parameters: the cost of identifying a sick
patient as healthy, and the cost of identifying a healthy
patient as sick. For example, if the patient is a zombie and it
contaminates all the rest of the humanity we want to minimize the
occurrences of falsely identifying the sick patient as healthy, while if the cure for “zombiness” is
lethal for a human patient, we want to minimize the occurrences of falsely identifying the healthy person as sick. With P and N we count the number of patients tested
Positively or Negatively. This is formalized in the following
definitions:

-   **TP True positive (statistical power)** : are those labeled as
    sick that are actually sick.

-   **FP False positive (type I error)**: are those labeled as sick but
    that actually are healthy

-   **FN False negative (type II error)** : are those labeled as
    healthy but that are actually sick.

-   **TN True negative**: are those labeled as healthy that are healthy.

Given this simple intuition, we can take a binary classifier and perform an experiment over a data set. We can measure:

-   **True Positive Rate (TPR) = Recall = Sensitivity**: is the ratio of
    correctly identified instances as sick among all the instances that are sick. It answers the question: “how good are we at detecting sick
    people accurately?”.
    $$\frac{  TP }{ TP +  FN}  + \frac{TP }{P} \simeq  P(test=1|sick=1)$$
    This is an estimator of the probability of a positive test given a
    sick individual.

-   **True Negative Rate (TNR) = Specificity** is a ratio that shows how many are labeled as healthy but that are actually sick.
    $$\frac{ TN }{ TN +  FP}  = p(test = 0 |  sick =0)$$ How many
    healthy patients will test negatively to the test? How good are we at avoiding false alarms?

-   **False Positive Rate = Fallout**
    $$FPR = \frac{  FP }{ FP +  TN }  = 1 - TNR$$

-   **False Negative Rate = Miss Rate**
    $$FNR = \frac{  FN }{ FN +  TP } = 1 - TPR$$

-   **Precision, Positive Predictive Value (PPV)**:
    $$\frac{ TP }{ TP + FP} \simeq p(sick=1 | positive=1)$$ How many instances tested positive are actually sick?

-   **$$F_1$$ score** is a more compressed index to measure the performance of a binary classifier. It simply calculates the harmonic mean of Precision and Sensitivity:
    $$F_1 = 2\frac{Precision \times Sensitivity }{Precision + Sensitivity }$$

-   **Receiver Operating Characteristic (ROC)** evaluates the TRP and FPR
    at all the scores returned by a classifier by changing a parameter.
    It is a plot of the true positive rate against the false positive
    rate for the different possible value (cutpoints) of a test or
    experiment.

-   The **confusion matrix** generalize these 4 combination of (TP TN FP
    FN) to multiple classes: is a $$l \times l$$ where at row $$i$$ and
    column $$j$$ you have the number of elements from the class $$i$$ that
    have been classified as elements of class $$j$$.

Confession: we wrote this post because we forget about these terms occasionally and we were not able to find them described in a concise way with the same formalism
without googling which accumulatively has taken more time than what was spent writing this post. Further reference can be found here:
[here](https://uberpython.wordpress.com/2012/01/01/precision-recall-sensitivity-and-specificity/)
