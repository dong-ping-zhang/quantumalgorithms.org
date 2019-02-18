--- 
id: 2
layout: page
chapter: 1 
comments: true 
tags: qml, tools
title: Swap test for distances and Nearest Centroids
description: Here we see how to use the swap test to calculate distances and inner products betewen vectors in our dataset. The same algorithm can use superposition to perform Nearest Centroid, a very simple classical machine learning algorithm. 
author:
- 'Alessandro “Scinawa” Luongo'
bibliography:
- 'sample.bib'
- 'Mendeley.bib'
permalink: swapdistances
---

Introduction to the swap test
-----------------------------

What is known as *swap test* is a simple but powerful circuit used to
measure the “proximity” of two quantum states (cosine distance in
machine learning). It consists in a controlled swap operation surrounded
by two Hadamard gates on the controlling qubit. Repeated measurements of
the ancilla qubit allows us to estimate the probability of reading $0$
or $1$, which in turn will allow us to estimate $\braket{\psi|\phi}$.
Let’s see the circuit:

![image](/assets/swap_distances/swap_test.png)

It is simple to check that the state at the end of the execution of the
circuit is the following:

$$
\Big[\ket{\psi} \ket{\phi} + \ket{\phi} \ket{\psi} \Big]\ket{0} +\Big[\ket{\psi} \ket{\phi} - \ket{\psi} \ket{\phi} \Big] \ket{1}
$$

Thus, the probability of reading a $0$ in the ancilla qubit is:
$$P (\ket{0}) = \left( \frac{1+|\braket{\psi|\phi}|^2}{2} \right)$$ And
the probability of reading a $1$ in the ancilla qubit is:
$$P (\ket{1}) = \left( \frac{1-|\braket{\psi|\phi}|^2}{2} \right)$$

This means that if the two states are completely orthogonal, we will
measure an equal number of zero and ones. On the other side, if
$\ket{\psi} = \ket{\phi}$, then the probability amplitude of reading
$\ket{1}$ in the ancilla qubit is $0$. Repeating this operation a
certain number of time, allows us to estimate the inner product between
$\ket{\psi},\ket{\phi}$. Unfortunately, at each measurement we
irrevocably destroy the states, and we need to recreate them in order to
perform again the swap test. We will see, that this is not much of a problem if we have
an efficient way of creating $\ket{\psi}$ and $\ket{\phi}$. But you might also ask how much of the two states can I recover after the swap test. The use of swap test can be better described with the following theorem:


##### Theorem: Swap test for inner products
Suppose you have access to unitary
$U_\psi$ and $U_\phi$ that allows you to create $\ket{\psi}$ and
$\ket{\phi}$, each of them requiring time $T(U_\psi)$ and $T(U_\phi)$.
Then, there is an algorithm that allows to estimate inner products between
two states $\ket{x},\ket{y}$ in $O(T(U_\psi)T(U_\phi)\varepsilon^{-2})$ time.


Let's recall some useful theorems before goin on with the proof: 
##### Theorem: Chernoff Bounds
Let $X = \sum_{i=0}^n X_i $, where $X_i = 1$ with probability $p_i$ and $X_i=0$ with probability $1-p_i$. All $X_i$ are independent. Let $\mu = E[X] = \sum_{i=0}^n p_i$. Then:
-   $P(X \geq (1+\delta)\mu) \leq e^{-\frac{\delta^2}{2+\delta}\mu} $
    for all $\delta > 0$
-   $P(X \leq (1-\delta)\mu) \leq e^{\frac{\delta^2}{2}}$

##### Theorem: Chebyshev inequality
_Let $X$ a random variable with $E[X] = \mu$ and $Var[X]=\sigma_2$. For all $t > 0$:_

$$P(|X - \mu| > t\sigma) \leq 1/t^2$$

_If we substitute $k/\sigma$ on $t$, we get the equivalent version that we use to bound the error:_
$$P(|X - \mu|) \geq k) \leq \frac{\sigma^2}{k^2}$$


_Proof: The correctnes of the circuit was sown before. This is the analysis of the running time. We recognize in the measurement on the ancilla qubit a random variable $X$ with Bernulli distribution with $p=(1+|\braket{\psi|\phi}|^2)/2$, and variance $p(1-p)$. The number of repetitions that are necessary to estimate the expected value $\bar{p}$ of $X$ with relative error $\epsilon$ is bounded by the Chebyshev inequality._

Swap test for distances between vector and center of a cluster
--------------------------------------------------------------

Now we are going to see how to use the swap test to calculate the
distance between two vectors. This section is entirely based on the work
of {% cite Lloyd2013a %}. There, they explain how to use
this subroutine to do cluster assignment and many other interesting
things in quantum machine learning. 

In the following section we will assume that we are
given access to two unitaries $U : \ket{i}\ket{0} \to \ket{i}\ket{v}$
and $V : \ket{i}\ket{0} \to \ket{i}\ket{|v_i|}  $.

Before starting, let’s recall the relation between inner product and distance of
$\vec{u}, \vec{v} \in \mathbb{R}^n$. The inner product between two
vector is $\braket{ v, u } = \sum_{i} v_i u_i $, and the norm of a
vector is $ |v|= \sqrt{\langle v, v \rangle} $. Therefore, the distance
can be rewritten as:

$$ ||u-v|| = \sqrt{ \langle u-v, u-v \rangle } = \sqrt{\sum_{i} (u_i - v_i )^2 } = \sqrt{|u|^2 + |v|^2 -2 \langle u, v \rangle } $$

By setting $$ Z=|u|^2 + |v|^2 $$
it follows that:
$ ||u-v||^2 = Z \left( 1 - \frac{ 2 \langle u, v \rangle } {Z} \right) $.

As you may have guessed, to find the distance $||v-u||$ we will repeat the
necessary number of times the swap circuit in order to estimate the probability of a certain outcome. The task is now is to find the right states to create to measure a distance with this circuit. 

We first start by creating
$$|\psi \rangle = \frac{1}{\sqrt{2}} \Big( \ket{0}\ket{u} + \ket{1}\ket{v} \Big)$$
querying the [QRAM](qram) in $O(log(N))$ time, where N is the dimension of the
Hilbert space (the length of the vector of the data). Then we proceed by creating
$$|\phi\rangle \frac{1}{\sqrt{Z}} \Big( |\vec{v}||0\rangle + |\vec{u}||1\rangle \Big) $$
and and estimate $Z=|\vec{u}|^2 +  |\vec{v_j}|^2$. Remember that for two
vectors, $Z$ is easy to calculate, while in the case of a distance
between a vector and the center of a cluster then
$Z=|\vec{u}|+\sum_{i \in V} |\vec{v_i}|^2$. In this case, calculating
$Z$ scales linearly with the number of elements in the cluster, and we
don’t want that.

To create $\ket{\phi}$ and estimate $Z$, we have to start with another,
simpler-to-build $\ket{\phi^-}$ and make it evolve to $\ket{\phi}$. To
do so, we perform an [Hamiltonian simulation](hamsim) to apply the following time dependent Hamiltonian to the initial state $\ket{\phi^-} = \ket{-}\ket{0}$, for a certain amount of time $t$ such that $t|\vec{v}|, t|\vec{u}| << 1$:
$$H = |\vec{u}|\ket{0}\bra{0}+|\vec{v}|\ket{1}\bra{1} \otimes \sigma_x$$


Formally, this means that we apply $e^{-iHt} \ket{\phi^-}$ for small $t$. This will give us the
following state:
$$\Big( \frac{cos(|\vec{u}|t)}{\sqrt{2}}\ket{0} - \frac{cos(|\vec{v}|t)}{\sqrt{2}}\ket{1} \Big) \ket{0} - \Big( \frac{i sin(|\vec{u}|t)}{\sqrt{2}}\ket{0} - \frac{i sin(|\vec{v}|t)}{\sqrt{2}}\ket{1} \Big) \ket{1}$$

Reading the ancilla qubit in the second register, we should read $1$
with the following probability, given by small angle approximation of
the $sin$ function:

$$P(1) =   \lvert - \frac{i sin(|\vec{u}|t)}{\sqrt{2}} \rvert^2 + \lvert \frac{i sin(|\vec{v}|t)}{\sqrt{2}} \lvert^2   \approx |\frac{|\vec{u}|t}{\sqrt{2}}|^2 + | \frac{|\vec{v}|t}{\sqrt{2}} |^2 =  \frac{1}{2} \Big( |\vec{u}|^2t^2 + |\vec{v}|^2t^2 \Big) = Z^2t^2/2$$

Now we are almost ready to use the swap circuit. Note that our two
quantum register have a different dimension, so we cannot swap them.
What we can do instead is to swap the index register of $\ket{\phi}$
with the whole state $\ket{\psi}$. The probability of reading $1$ is:

$$\begin{split}
p(1) = \frac{2||\vec{u}||^2 + 2||\vec{v}||^2 - 4(u,v)}{8Z}
\end{split}$$

The distance is normalized by $Z$, which we assume to know classically. So, after sampling we can rescale the probablity by Z.  We saw how to use a simple circuit to estimate things like inner product
and distance between two quantum vectors. We have assumed that we have
an efficient way of creating the states we are using, and we didn’t went
deep into explaining how. Thanks to the statistical estimators we have, given a $\epsilon > 0$, we can repeat the previous circuit $O(\epsilon^{-2})$ times to have the desired precision.

This circuit ca be easily extended to perform Nearest Centroid algorithm by using a query in superposition. As you may have thought it is very simple with a classical computer to calculate the distance between two vectors. By quering the QRAMs in superposition we can estimate how to calculate the distance between a vector and the center of a cluster. Classically, this would take linear time  in the number of elements, while with the quantum computer is exponentially faster. Note that there is also an exponential improvement in the number of qubits we use to store the state: classically we would use $O(dm)$ bits to store a vector $\in \mathbb{R}^d$ of length $d$ with $m$ bits of precisions. Quanumly, we can use only $\log d$ qubits with a [fidelity]() proportional to $m$.  Further on, we can use use amplitude estimation in order to reduce the dependency on error to $O(\epsilon^{-1})$.


Calculations
------------

It’s time now to prove that our claim is true and to show some
calculation. After all the previous passages, this is the initial state:

$$\ket{0}\Big(  \frac{1}{\sqrt{Z}} \left( |\vec{u}|\ket{0} + |\vec{v}|\ket{1} \right)     \otimes \frac{1}{\sqrt{2}} (\ket{0}\ket{u} + \ket{1}\ket{v} ) \Big)$$

We apply an Hadamard on the leftmost ancilla register:

$$\frac{1}{2\sqrt{Z}} \left[   \ket{0} \Big(   \left( |\vec{u}|\ket{0} + |\vec{v}|\ket{1} \right)     \otimes  (\ket{0}\ket{u} + \ket{1}\ket{v} ) \Big)   +
                        \ket{1} \Big(  \left( |\vec{u}\ket{0} + \vec{v}\ket{1} \right)     \otimes  (\ket{0}\ket{u} + \ket{1}\ket{v} ) \Big)  \right] =$$

$$\begin{split}
= \frac{1}{2\sqrt{Z}} \Big[   \ket{0} \Big(  |u|\ket{00u} + |u|\ket{01v} + |v|\ket{10u}  + |v|\ket{11v} \Big) \\
\ket{1} \Big( |u|\ket{00u} + |u|\ket{01v} + |v|\ket{10u} + |v| \ket{11v} 
\Big)  \Big]
\end{split}$$

Controlled on the ancilla being $1$, we swap the second and the third
register:

$$\begin{split}
= \frac{1}{2\sqrt{Z}} \Big[   \ket{0} \Big(  |u|\ket{00u} + |u|\ket{01v} + |v|\ket{10u}  + |v|\ket{11v} \Big) \\
\ket{1} \Big( |u|\ket{00u} + |u|\ket{10v} + |v|\ket{01u} + |v| \ket{11v} \Big)  \Big]
\end{split}$$

Now we apply the Hadamard on the leftmost ancilla qubit again:

$$\begin{split}
= \frac{1}{2^{3/2}\sqrt{Z}} \Big[ 
|u|\ket{000u} + |u|\ket{001v} + |v|\ket{010u} + |v|\ket{011v} \\
+|u|\ket{100u} + |u|\ket{101v} + |v|\ket{110u} + |v|\ket{111v}  \\
+|u|\ket{000u} + |u|\ket{010v} + |v|\ket{001u} + |v|\ket{011v} \\
-|u|\ket{100u} - |u|\ket{110v} - |v|\ket{101u} - |v|\ket{111v}
\Big]
\end{split}$$

And now we check the probability of reading $\ket{1}$.

$$\begin{split}
p(1) = \frac{1}{2^{3}Z} (u\bra{v10} + v\bra{u01} - u\bra{v01} - v\bra{u10})\\
(u\ket{01v} + v\ket{10u} - u\ket{10v} - v\ket{01u})
\end{split}$$

$$\begin{split}
p(1) = \frac{2||u||^2 + 2||v||^2 - 4(u,v)}{8Z}
\end{split}$$


### References 

{% bibliography --cited %}

