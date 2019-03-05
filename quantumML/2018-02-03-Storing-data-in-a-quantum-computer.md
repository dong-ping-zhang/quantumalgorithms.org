--- 
title: Storing data in a quantum computer
comments: true 
status: done
tags: qml, intro 
description: Here we see how we can encode classical data in a quantum computer, and why it is important in quantum machine learning.
author:
- 'Alessandro “Scinawa” Luongo'
- 'Dong Ping Zhang'
permalink: storing.html
---

In this article, we talk about: what does it mean to represent or store data in a quantum computer. This is very important, because knowing what is the best way of encoding data in a quantum computer might pave the way for the intuition in solving new problems using a quantum computer. On the contrary, using the wrong encoding might prevent you from reasoning about the algorithm design and obtaining the desired advantages (e.g. performance gain) in the implementation of your algorithm. In {% cite schuld2015introduction %}, Maria Schuld and others wrote: _In order to use the strengths of quantum mechanics without being confined by classical ideas of data encoding, finding “genuinely quantum” ways of representing and extracting information could become vital for the future of quantum machine learning_. 

Here, we focus on accessing purely classical data. In this context, we usually assume to have access to a unitary operator $U$ that performs a query to a data structure that does the following: 
$$U\ket{i}\ket{0}\to \ket{i}\ket{x_i}$$

As a convention, we often use Greek letters inside kets to represent generically quantum states, and use Latin letters to represent quantum registers holding classical data. 
The first is called the index register, and the second is the target register, which after the query to a certain oracle is storing the information you want. 
Fundamentally, there are just two ways of encoding the information: the amplitude encoding and the binary encoding. In amplitude encoding you store your data in the amplitudes of a quantum state, therefore you can encode $n$ real values (up to some digits of precisions) using $\log n$ qubits. In the binary or digital encoding you store a bit in each qubit. You'll need to chose the appropriate strategy to process the data for each of chosen encoding. 


Scalars: $\mathbb{Z}$, $\mathbb{Q}$, $\mathbb{R}$
-------------------------------------------------------
In general, these values are encoded using the binary encoding. 
Let’s start with the most simple scalar: an integer. Let
$m \in \mathbb{N}$. We consider the binary expansion of $m$ encoded in $n$ qubits.  As example,
if your number’s binary expansion is $0100\cdots0111$ we can create the
state:
$\ket{x} =  \ket{0}\otimes \ket{1} \otimes \ket{0} \otimes \ket{0} \cdots \ket{0} \otimes \ket{1} \otimes \ket{1} \otimes \ket{1}$.
Formally, given $m$:

$$\ket{m} = \bigotimes_{i=0}^{n} \ket{m_i}$$

Using superposition of states like these we might create things like
$\frac{1}{\sqrt{2}} (\ket{5}+\ket{9})$ or more involved convex
combination of states. If we need to store negative integers, we could use one extra qubit for the sign, and invent some other tricks.

When programming quantum machine learning algorithms, it is common to use subroutines to perform arithmetics on real numbers. The numbers might come from other quantum procedures (like phase estimation or amplitude amplification) or might come directly from the memory (i.e.  like the norm of a vector). These numbers are simply encoded as m-bit binary expansion.

This encoding can be used to get speedup in the number of query to
an oracle, like in {% cite kapoor2016quantum %}. There, the authors encoded a real value representing each of the feature of a machine learning dataset with this encoding. In general, you can easily use this encoding if you aim at getting a speedup in oracle complexity using amplitude amplification and similar techniques, or in an intermediate step of your algorithm where you want to perform arithmetics on some values. Oracle complexity is a measure of the complexity of an algorithm that counts the number of times a given oracle $O$ (a unitary in the quantum computer) is called. 


Binary vectors: $\\{0,1\\}^n$
---------------------------

Let $\vec{b} \in \\{0,1\\}^n$.

We have three options here. As for the encoding the integers, we can use $n$ qubits with a digital encoding:

$$\ket{b} = \bigotimes_{i=0}^{n} \ket{b_i}$$

As an example, suppose you want to encode the vector
$[1,0,1,0,1,0] \in \\{0,1\\}^6$, which is $42$ in decimal. This will
correspond to the $42$-th base of the Hilbert space where our qubits
will live. Note that, in some sense, we are not "fully" using the $C^{2^{n}}$
Hilbert space: we are mapping our list of binary vector in vectors of the canonical base.
base. As a consequence, Euclidean distances between points in the new space are
different. 

Our second encoding is the following. For instance we can map a $0$ into
$1$ and $1$ as $-1$, and build:

$$\ket{v} = \frac{1} {\sqrt{2^n}} \sum_{i \in \{0,1\}^n} (-1)^{b_i} \ket{i}$$.

Or analogously: 

$$\ket{b} = \frac{1}{\sqrt{\sum_{i=0}^n}1} \sum_{i=0}^n b_i\ket{i} $$


Vectors and Matrices: $\mathbb{R}^n$, $\mathbb{R}^{n \times d}$
-----------------------------------------------------------------

Here is an example of amplitude encoding, a powerful approach in quantum machine learning algorithms for encoding the dataset. For a vector
$\vec{x} \in \mathbb{R}^{2^n}$, we can build:

$$\ket{x} = \frac{1}{{\left \lVert x \right \rVert}}\sum_{i=0}^{N}\vec{x}_i\ket{i} = |\vec{x}|^{-1}\vec{x}$$

Note that to span a space of dimension $N=2^n$, you just need $n$
qubits: we encode each component of the classical vector in the
amplitudes of a state vector. Being able to build this family of states enable us to load matrices in memory.

Let
$X \in \mathbb{R}^{n \times d}$, a matrix of $n$ vectors of $d$
components. We will encode them using $log(d)+log(n)$ qubits as the
states:

$$\frac{1}{\sqrt{\sum_{i=0}^n {\left \lVert x(i) \right \rVert}^2 }} \sum_{i=0}^n {\left \lVert x(i) \right \rVert}\ket{i}\ket{x(i)}$$

Or, put it another way:

$$\frac{1}{\sqrt{\sum_{i,j=0}^n,d {\left \lVert X_{ij} \right \rVert}^2}} \sum_{i,j} X_{ij}\ket{i}\ket{j}$$


This state is built using a [QRAM](qram.html). A QRAM gives us quantum access to two things: the norm of
the rows of a matrix and the rows itself. Formally, we assume to have access to $\ket{i}\mapsto \ket{i}\ket{x(i)} and $\ket{i} \mapsto \ket{i}\ket{\norm{x(i)}}$. Using these unitaries, we can do the following mapping:

$$\sum_{i=0}^{n} \ket{i} \ket{0} \to  \sum_{i=0}^n {\left \lVert  x(i)  \right \rVert}\ket{i}\ket{x(i)}$$

Many example of this encoding can be found in literature, like [QSFA](qsfa.html) and [many](https://arxiv.org/abs/1811.03975) others.

Graphs
======

For some kind of problems we can even change the computational model (i.e.
we can switch from the gate model to something else). For instance, a graph $G=(V,E)$ be encoded as a quantum state $\ket{G}$ such that:
$$K_G^V\ket{G} = \ket{G} \forall v \in V$$ where
$K_G^v = X_v\prod_{u \in N(v)}Z_u $, and $X_u$ and $Z_u$ are the Pauli
operators on $u$. Here is the way of picturing this encoding: Take as many
qubits in state $\ket{+}$ as nodes in the graph, and apply controlled
$Z$ rotation between qubits representing adjacent nodes. There are some
algorithms that use this state as input. For a more detailed description, look at {% cite zhao2016fast %}, which also gives very nice algorithms that might be useful in some graph problem. 

Conclusions
===========

The precision that we can use for specifying the amplitude of a quantum state might be limited in practice by the precision of our quantum computer in manipulating quantum states (i.e. development in techniques
in quantum metrology and sensing). Techniques that use a certain precision in the amplitude of a state might suffer of initial technical limitations of the hardware. The precision in the manipulation can be measured by the fidelity. At an inutitive level, you can think of the precision in manipulating quantum state as the 16, 32 or 64 bits of precision in our classical CPUs. 


### References

{% bibliography --cited %}

