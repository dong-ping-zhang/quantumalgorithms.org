---
layout: page
title: How to QRAM your classical data in a quantum computer
comments: true 
status: done
tags: qml, tools
description: "The fastest way to load classical data in a quantum computer is the QRAM: an efficient state preparation tecnique that requires an almost linear time algorithm as preprocessing."
author:
- Alessandro Scinawa Luongo
permalink: kptrees.html
---
In this post we’re going to build some quantum circuits to load data in a quantum computer. These circuits are one of multiple solutions to the more general problem of state preparation: how to create a quantum state $\ket{\psi(\theta)}$ for an expressive-enough class of quantum states (eventually parameterized by $\theta$). This article focuses on QRAM. There are other solutions that we might cover in future posts.  


This post is organized as such: first we talk about the theory behind the QRAM, stating the theorem that you can use to build you own quantum algorithms. Then we will explain *in practice* how to build some circuits that give you deeper insight what the QRAM is. Finally, we will address some further issues and conclude.

/* dpz: The following paragraph needs clarification. TBD */ 

While talking to people at the conferences, we realized that QRAM is a word used to refer to two different concepts. Originally, it was used to describe an oracle that loads in memory $n$ different values as *digital* encoding of $m$ bits (i.e. you create a tensor of $m$ qubits to store a scalar value). Others also use the term for a circuits which loads the data with *amplitude encoding*. As András Gilyén puts it: "if you add a *q* in front of it, should be the quantum version of the classical thing". As the classical RAM works by returning a list of bits, I'd stick with the idea of using QRAM for the original meaning and call the other **KP-trees**, or **QRAM-amplitudes**. Further ambiguity is introduced by the fact that KP-trees internally uses the QRAM. So sometimes people use QRAM to describe both of the circuit.


You can think of KP-trees in two ways. First as a unitary that you can execute in a quantum computer, or as a particular data structure that you can use to store your data when your data is represented by a matrix. Imagine that your training data set is a list of $n$ vectors of dimension $d$, and you want to load them onto a quantum computer. The KP circuit, albeit being $O(nd)$ size (it is a faithful representation of the matrix), is of polylogarithmic depth in $n$ and $d$. This means a sublinear runtime when executed. Formally, for a given vector $x(i) \in \mathbb{R}^d$ and $i \in [n]$, we can use the KP-trees to build the following state:

$$ U_{KP}: \frac{1}{\sqrt{n}} \sum_{i\in [n]} \ket{i}\ket{0} \mapsto \frac{1}{\sqrt{\sum_{i \in [n]} \norm{x_i}^2}} \sum_{i=0}^{n} \norm{x_i}\ket{i}\ket{x(i)} $$

Let's take a look at this iconic quantum state. Mathematically, this state is just a very long vector of $nd$ length (advantageously padded to make $n$ and $d$ powers of two). You can imagine it by piling all the *normalized* rows of $X$ one after the other such that the final result has unitary norm. Then, each of the element in the tensor is weighted by the norm of the row, and the final state is again normalized to have unit $\ell_2$. In other words, each of the $\ket{x(i)}$ in the second register is a unitary vector, as $\ket{x(i)} = \norm{x(i)}^{-1}\vec{x}(i)$, and each of them is weighted by their norm ($\norm{x_i}$). You might wonder if you need the first register in the output of the KP-circuit, or if you can get rid of it. Well, it turns out it is actually very useful, as you will see in our other articles of quantum algorithms. 
In some sense, this state is not difficult to build. The challenge is that we want to built it with a circuit whose depth is polylogarithmic in $n$ and $d$.

Mathematically, the idea behind the KP-trees is rooted in an observation described in  {% cite Grover2002creating %}, which in just two pages gives an efficient algorithm for the *state preparation problem*: the problem of finding a circuit that builds a given quantum state. Precisely, for a state proportional to square integrable probability didstribution. Indeed, _by precomputing the partial amplitudes_, it is possible to perform only a logarithmic number of controlled operations. Later on we will use this intuition, since luckily, a data vector in $\mathbb{R}^d$ (i.e. like most of the classical data that we use in ML) can be thought as an efficient square integrable distribution.

Imagine we have received a set of classical data in form of a matrix of $n$ rows (samples) and $d$ columns (features). The KP-trees for a matrix $X$ is a collection of binary trees $B_i$ of depth $\log d$, where each tree has stored the information needed to create a state corresponding to each row of the matrix. Your quantum circuit will perform "parallel" controlled rotation on $\log d$ qubits efficiently. 

Practically, we want to build a circuit to perform two operations:

$$  \frac{1}{\sqrt{n}}\sum_{i=0}^n\ket{i} \to \frac{1}{\sqrt{\sum_{i}^n \norm{x(i)}^2}}\sum_{i=0}^n ||x_i||\ket{i} $$ 

and

$$  \frac{1}{\sqrt{n}}\sum_{i=0}^n\ket{i} \to \frac{1}{\sqrt{n}}\sum_{i=0}^n \ket{i}\ket{x(i)} $$ 

##### Theorem: KP-trees, or QRAM-amplitudes circuit for the matrices.

_Let_ $X \in \mathbb{R}^{N \times d}$. _There exists a data structure to store the rows of $X$ such that:_
- _The size of the data structure is_ $O(nd\\: log(nd))$
- _The time to store a row $x(i)$ is $O(d\\: log^2(nd))$, and the time to store the whole matrix $X$ is thus $O(nd\\: log^2(nd))$_
- _A quantum algorithm that has quantum access to the data structure can perform the mapping 
$U\_X: \ket{i}\ket{0} \to \ket{i}\ket{x(i)} $ and the mapping $V\_X : \ket{j}\ket{0} \to \ket{j}\ket{\widehat{X}}$ for $j \in [d]$ where $\widehat{X} \in \mathbb{R}^n$ has entries $\widehat{X}_i = \norm{x(i)}$ in time $polylog(nd)$_

The proof consist of building the following *binary* tree, which is performed as a pre-processing phase of the data before the execution. This can be done upon receiving the data, or while building the quantum circuit before the execution. Here, you will mostly find what is written in {% cite kerenidis2016recommendation %}.

I call $x(i)$ a row of the matrix $X$ and I address each of the elements as $x_{ij}$. For each row $x(i) \in X$ we build a  binary tree $B_i$ of depth $\lceil \log d\rceil$ with $d$ leaves. Each internal node $v$ of the tree stores the sum of the entries of all the leaves of the subtree rooted at $v$, that is, the sum of the 2 childrens. The leaf of each tree $B\_i$ stores the value $x_{ij}^2$, along with the sign of $x_{ij}$. Organized as such, you can easily verify that the root of the tree stores $\norm{x(i)}^2$. 

Each layer of the tree is stored in a bucket-brigade QRAM. To create this data structure you need to be able to address all the $nd$ elements of the matrix, which counts for $\log nd$ bits. 

We need to measure the time and the space needed to *build* and *store* this data structure. 
The *time* needed to build a binary tree for a vector of length $d$ is $O(d \log^2 nd)$ because for each of the $d$ component of the vector, there are $\lceil log d \rceil$ updates to the data structure (given by the creation of a new leave and the updates on the internal nodes of the tree), and each update requires $O(\log nd)$ operations to retrieve the address of the node to update (you can imagine you are using dictionary, which still needs to be able to read all of the $\log nd$ bits to address $nd$ values.).

On *space* complexity, given that we have to store $n$ vectors, the space used to store the matrix $X$ is therefore $O(nd\\: log^2 nd)$: there are $n$ vectors and each vector creates $O(d \log(d))$ nodes, and each node uses $O(\log nd)$ bits. 

The time needed to store the whole matrix is $O(nd\\: log^2(nd))$. 

Note also that in these calculations we factor out all the observation on the bits of precision used to store each value $x_{ij}$. This somehow is related to the precision we want to achieve, and the precision allowed by the technology, i.e. how precisely can we control quantum gates?

You might observe that the process of summing the amplitudes in a tree is similarly to the idea of Grover we mentioned before. Indeed we are just storing all the partial amplitudes in a tree.

Now we are going to show how to use quantum access of this data structure to load the data. Observe that for a tree $B_i$, at depth $t$ the data stored in a node
$k \in \\{0,1\\}^t$ is stored:

$$B_{i,k} := \sum_{j \in [n], j_{1:t}=k} x^2_{ij}$$

In this formula with $j_{1:t}=k$ we denote the first $t$ bits in the binary expansion of $j \in \left[ d \right]$. This value represent the probability of observing outcome $k$ by reading only the first $t$  qubits of $\ket{x(i)}$ in the standard basis. This observation will be used to perform a series of controlled rotations on the data register (i.e. the set of qubits storing the vector $x(i)$). Let's see how. 


Now we will show how to use quantum access to the various layer of the trees to build a state $\ket{x(i)}$, using only polylog($nd$) operations. This means that we are allowed to query the layers of each of the trees in superposition. Given an initially empty register $\ket{0}$ of $\lceil log d \rceil$ qubits, we perform a series of rotation such that the amplitudes of the state of the register match the probability stored in the KP-trees. Specifically, we apply the following map:

$$\ket{i}\ket{k}\ket{0} \to \ket{i}\ket{k}\frac{1}{\sqrt{B_{i,k}}} \Big(\sqrt{B_{i,k0}}\ket{0} + \sqrt{B_{i,k1}}\ket{1}  \Big)$$


It can be done by first querying a bucket-brigade like data structure for

$$\ket{i}\ket{k}\ket{0}\to\ket{i}\ket{k}\ket{\theta_{ik}}$$

Here, $\theta_{ik}$ is $arcsin(\frac{\sqrt{B\_{ik0}}}{\sqrt{B\_{ik}}})$, or equivalently $arccos(\frac{\sqrt{B\_{ik1}}}{\sqrt{B\_{ik}}})$. We want to use this information to perform a controlled rotation $R(\theta)$ on the $t$-th qubit of the data register. We can use each of the qubits of $\ket{\theta_m}$ as control register for the rotation on each of the qubit of the data register.  In quskit, this is exactly the gate cu1, albeit extended to control many qubits. In practice, it will be something along this line:

~~~
circ.cu1(math.pi theta_m/float(2 **m), q_reg[i], target_qubit)
~~~
{: .language-python}


For the last qubit, the controlled rotation takes into consideration the eventual sign:

$$\ket{i}\ket{k}\ket{0} \to \ket{i}\ket{k}\frac{1}{\sqrt{B_{i,j}}} \Big(sgn(a_{i,k0})\sqrt{B_{i,k0}}\ket{0} + sgn(a_{i,k0})\sqrt{B_{i,k1}}\ket{1}  \Big)$$

Obviously, the number of controlled rotation to apply is $\lceil log d \rceil$. The operation for the qubit $t+1$, is controlled on the index register $\ket{i}$ and on the previous $t$ bits of the register being on state $k$. For each controlled operation there are 2 query to the binary tree: once for each children.

Finally, we show how to build $V\_X$. This procedure is analogous to the creation of a binary trees $B\_i$, for the rows, except that instead of saving the square of the amplitude we just need to save the square of the norm of the vectors. This is a no-brainer for us, since the norm of the vector is stored in the root node of $B\_i$. The $i$-th component of the $\widehat{X}$ is 
$||x(i)||^2$. Thus, when creating $B\_i$ we also update the tree for the oracle $V_X$. Querying the oracle $V_X$ can be done in polylog($Nd$) time as well.

This concludes the proof of the previous theorem. What remains much more interesting is the actual way to build the circuit for the query to the nodes of the trees.

##  Some circuits for the QRAM and the oracles

Let's start our journey from the oracle employed in the Grover algorithm. There, you assume to have access to a function $f: \{0,1\}^n \to \{0,1\}$, which, as an oracle, you write as:

$$ U_f \ket{x}\ket{c} \mapsto \ket{x}\ket{c \oplus f(x)} $$

with the caveat that $f(x)$ is 1 for just a single value of $x$. You can think of this function as a list $[n]$ elements representing all the $f(x)$ for which just one is $1$ and the others are $0$. How would you build $U_f$ as a circuit? If you think about it, you might realize that you can build this oracle as just a single controlled operation! Indeed, controlled on the value of the index register (the leftmost) being $x$ we perform a X gate on the data register. Note that the depth of this oracle is $O(1)$ since we have just a single gate. (it's actually a little bit more than a single gate since we have to decompose the multi-controlled-not in the gate set supported by the QPU (quantum processing unit) but this is another story..). 

Building upon this idea, we do a slight generalization, and construct what goes under the name of a *multiplexer*. Imagine you have $n$ qubits in the index register and $m$ qubits in the target register. Controlling on the index register being one upon $2^n$ different states, we write the $m$ bit strings of integers, basically concatenating $nm$ controlled operations.

If we want to load the list $\[3,1,3,7\]$, we do need to address 4 elements (i.e. 2 qubits in the index register) ad we need 3 qubits in the data register (7 in decimal is 111 in binary).

Note that, since in qiskit you dont have the ability to control operation on qubits being $0$, we perform a $X$ gate before using a "normal" control (and we undo it after).  In this case the depth of the circuit is $O(n)$, so we would't use this circuit in practice, but is a very simple way to store date and prove you the existence of the cirucit. 

This is an example of the code. Note that this part of code uses the function `oracle_multiplexer`, which we have not pulished yet. 

~~~
# This represent the dataset
dataset = np.array([3,7,15])

n = int(np.ceil(np.log2(dataset.shape[0])))
d = len(bin(max(dataset))[2:])

i = 2
print("Retriving {}-th item from the oracle".format(dataset[i]))

idx = QuantumRegister(n, "index")
datareg = QuantumRegister(d, "data")
cdatareg = ClassicalRegister(d, "cdata")


qc = QuantumCircuit(idx,datareg,cdatareg)


#  I set the index register to i=2, because I want to query that value
initialize_index(qc, idx, i)
qc.barrier()

# I apply the circuit of oracle multiplexer
oracle_multiplexer(qc, dataset, idx, datareg)

qc.barrier()
qc.measure(datareg,cdatareg)

shotNum = 1024
job = execute(qc, Aer.get_backend('qasm_simulator'),shots=shotNum)
results = job.result()

            
key = bin(dataset[i])[2:].rjust(len(datareg),'0')
print("Dictionary of results (outcome, frequency):", results.get_counts())

print("We should have measured {} times {}, which in binary is {}".format(shotNum, dataset[i], key))
print("Check: {}".format(shotNum==results.get_counts()[key]))

qc.draw()
~~~
{: .language-python}


<img height="300" width="854  " src="/assets/images/qram/oraclemultiplexer.png"/>

Let's describe this image. The first NOT gate on the index_0 register is setting the circuit to query the second element of our dataset (2 in binary is 10, so we just filp one qubit). The rest is of the circuit is just the following: controlled in the index being 0, write on the data register the first element of the dataset. Note that since we cannot control on a set of qubit being 0, we have to "sandwitch" the controls with NOT gates. 




We can apply the same idea of the multiplexer to build a *multiplexer for the amplitude encoding*. The depth of the circuit will still be linear in the number of rows $n$ (i.e. we don't get any exponential speedup) but at least we are able to built the state we need. 
~~~
dataset = np.array([ [1,2,3,4], [2,3,4,5] ])

idx = QuantumRegister(n, "index")
datareg = QuantumRegister(d, "data")
cdatareg = ClassicalRegister(d, "cdata")

qc = QuantumCircuit(idx,datareg,cdatareg)

qram_multiplexer(qc,idx, datareg, dataset)

qc.draw()
~~~
{: .language-python}

Again, This is the resulting circuit. Note that it uses only one ancilla qubit for doing multi-controlled rotations on the data register.
<img height="854" width="854  " src="/assets/images/qram/kptreescircuit.png"/>




As before, the idea is the following. Instead of writing the values of the nodes at each query, we perform directly the controlled rotation on the ancilla qubit. Note that in this way you can create a binary tree $B_i$ with half the nodes of the original tree, since we just need the value of $\theta=arcsin(\frac{\sqrt{B\_{ik0}}}{\sqrt{B\_{ik}}}))$. The depth of the circuit is $O(nd)$, and this makes us sad. But we can amend to this, by using another architecture for performing the same task.


### Bucket-brigade QRAM and its application on KP-tree/QRAM-amplitude

This is where fun starts. The bucket-brigade BB QRAM is log-depth cirucuit to load the values of the layers of the tree at a given depth. 
A log-depth cricuti for querying the trees can therefore be used to solve the linear-depth problem of the circuit introduced before. The implementation, is thorougly described in {% cite giovannetti2008architectures %} but going into depth requires a post in its own. 

The useage of the BB within the context of the KP-trees is the following: first we use the BB to load in a quantum register the values of the nodes at depth $t$, and then, we use controlled rotations to rotate the $t$-th qubit in the data register coherently. 

More formally, controlled on the index register $\ket{i}$, we use a bucket-brigade architecture to load each of the $\log d$ layer of the $B_i$ trees in an ancillary register. Then, we can use other controlled rotation (controlled on the register we used to store the partial amplitudes loaded so far) to perform the rotation. In this way we can have achieve the goal we want: a log depth circuit to load matrix into memory, while using controlled rotations to encode the values as amplitudes.


#### Conclusions, criticisms, ideas...
In the original work, instead of building all the trees, the authors assumed to receave each of the component $x_{ij}$ of the matrix $X$ one after the other. Note that if we access to the whole matrix $X$, the creation of the trees $B_i$ can be easily paralelized.  

I hope now is clear why I consider the KP-trees/QRAM-amplitudes as a particular data format. The leaves of the trees $B_i$ stores the original matrix $X$ and therefore we can go back to our original encoding of $X$ as we want. 

There are some criticism that QRAM will be difficult to build, due to the complexity of the circuit (many CNOT and controlled orataions are involved). True, but after wall it will be as difficult as executing any other quantum cirucit with a lot of CNOTS. Moreover, we know from {% cite arunachalam2015robustness %} that circuit doing a polynomial number of queries to a QRAM might not need error correcting codes (they will just fail with a certain probability, and we can take majority of voting and bound exponentially the failure probability of the algorithm). 

Note that this model of "receiving and preprocessing" the data suits perfectly the machine learning paradigm. For instance, things like normalizing, removing the mean, and scaling data to unitary variance (all things which are often done when the training set is received, can also be done in $O(n)$ time during the preprocessing, thus the QRAM/KP-trees can be built right after the initial preprocessing. 

Let me conclude with some obsrvation I have made around this topic.

**Idea 1**. Note that you can imagine to store in the QRAM also a dataset after a pass of "Gaussian noise",  such that we anonymize the value of the feature of the matrix, but we might preserve the value of the thing that we can learn. Unfortunately, I do not know much about *privacy preserving machine learning*, but this is definitely worth studying more. Indeed, it would be crucial to see which techniques we can borrow from the classical world, and bring them in the quantum realm. We, as quantum information community, cannot pretend to ignore all the progress in these directions, since sooner or later, if we expect to apply QML in the real world we also need to be compliant to the legislation framework we developed.

**Idea 2**. Also note that if you give me the circuit for a QRAM/QRAM-amplitudes, I can recover very simply the original matrix X. What if we edit the circuit by randomly decomposion the controlled rotations in other gates, and by adding other random rewriting rules of the circuit? The hope is that, in this way, the only way to recover the dataset is doing tomography on a massively huge quantum state, by querying the qram for each value $\ket{i}$ of the index register. 


### References

{% bibliography --cited %}



