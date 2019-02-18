---
title: "How to estimate distances and inner products: the final algorithm"
comments: true 
status: done
tags: qml, tools
description: "There are many ways to find distances between vectors using quantum algorithms. Here we explain an optimized and simple-to-use Theorem for distance calculation."
author:
- Alessandro Scinawa Luongo
permalink: distances.html

---

So here we want to talk about a useful Theorem to add in the toolbox of the quantum machine learning practitioner: a subroutine to find distances between vectors! 


Obviously, techniques like these are far from being new in the literature. To my knowledge, the first one was proposed in Seth Lloyd and others in {% cite Lloyd2013a %}. Their algo is a clever application of the [swap test](https://arxiv.org/pdf/1310.2035.pdf): a small circuit created by Ronald de Wolf and Ashley Montanaro in the context of property testing. Note that the algorithnm based soley on the swap test and it didn't have a good enough success probability for a serious complex use in machine learning. Lloyd and others used it to solve the Nearest Centroid problem, a simple machine learning algorithm. Then, Nathan Wiebe and others in {% cite wiebe2014quantum %} proposed other two subroutines to evaluate distances between vectors, but they didn't use the QRAM as state preparation procedure (in order to give guarantees for the time needed to load the data to process, they relied on sparsity assumption). There, they introduced a "median evaluation circuit" to boost the success probability of the test. Then, in {% cite KL18 %} we basically used a QRAM and an Hadamard to devise the QFDC, a sqare-distances estimation subroutine to estimate sqared distances between vectors. Finally, in {% cite kerenidis2018qmeans %} with Jonas, Anupam, and Iordanis, we took all these ideas and we merged them together, formalizing everything with a nice Theorem. Our circuit has high probability of succeeding - wihtout impacting significantly the running time. Also, contrary to the swap test, it maintain the information of the sign of the inner product. It basically works with any efficient state preparation procedure, but the most general uses a [QRAM](qram.html) based data structure for amplitude encoding. 

Mathematically, what we are going to see, is how to build the following mapping $$ \ket{i}\ket{j}\mapsto\ket{i}\ket{j}\ket{d(v_i, c_j)}$$, where$$v_i$$ and $$c_j$$ are some vectors in a dataset, which you can think of rows of a matrix $$V$$ and $$C$$. 

( For the curious, I have wrote something around the swap test (with the calculation missing in the original Seth's paper in [here](swapdistances.html), [here](failed.html), and [here](rewriting.html) ). The QFDC is explained on this website [here](qfdc.html).)


Formally, the Theorem is the following:
#### Theorem: Distances and inner products estimation
_Assume that for two matrices $$V \in \mathbb{R}^{n \times d}$$ and $$C \in \mathbb{R}^{k \times d}$$ the following unitaries_ $$\ket{i}\ket{0} \mapsto \ket{i}\ket{v_i}$$, _and_ $$\ket{j}\ket{0} \mapsto \ket{j}\ket{c\_j}$$ _can be performed in time$$T$$and the norms of the vectors are known. For any $$\Delta > 0$$ and $$\epsilon>0$$, there exists a quantum algorithm that  computes:_
$$\ket{i}\ket{j}\ket{0} \mapsto \ket{i}\ket{j}\ket{\overline{d^2(v\_i,c\_j)}}$$, where 
$$|\overline{d^{2}(v\_i,c\_j)}-d^{2}(v\_i,c\_j)| \leqslant  \epsilon$$ with probability at least $$ 1-2\Delta$$,
or 
$$\ket{i}\ket{j}\ket{0} \mapsto  \ket{i}\ket{j}\ket{\overline{(v\_i,c\_j)}},$$ where
$$|\overline{(v_i,c_j)}-(v_i,c_j)| \leqslant  \epsilon$$ _with probability at least_ $$ 1-2\Delta $$
_in time_ $$\widetilde{O}\left(\frac{ \norm{v_i}\norm{c_j} T \log(1/\Delta)}{ \epsilon}\right)$$.

### Preface and comments..
Let me spend a couple of words and introduce some Lemmas. First, you have to know that we usually perform the mappings required in the hypotesis of the Theorem using a [QRAM](qram), but any other state preparation procedure will work. We report here the "Median Lemma" used in the {% cite wiebe2014quantum %}, which you I'm sure you are going to re-use *many* times in your algorithms.. 


#### Theorem: "Median Lemma" {% cite wiebe2014quantum %}
_Let$$\mathcal{U}$$be a unitary operation that maps_
$\mathcal{U}:\ket{0^{\otimes n}}\mapsto \sqrt{a}\ket{x,1}+\sqrt{1-a} \ket{G,0}$$for some$$1/2 < a \le 1$$in time$$T$. 
_Then there exists a quantum algorithm that, for any$$\Delta>0$$and for any$$1/2<a\_0 \le a$, produces a state$$\ket{\Psi}$$such that$$\|\ket{\Psi}-\ket{0^{\otimes nL}}\ket{x}\|\le \sqrt{2\Delta}$$for some integer$$L$, in time_:
$$
2T\left\lceil\frac{\ln(1/\Delta)}{2\left(|a_0|-\frac{1}{2} \right)^2}\right\rceil.
$$

Morally, this lemma tells you that you can pump-up the success probability of a unitary (that is already "good", meaning that the success probability should be always bigger than 1/2), to$$1-\delta$. The price for this boost is just a log factor in the cost of producing the state you want. 


### Proof of "Distances and inner products estimation" Theorem.

We know show you how to compute a simple unitary to store in the amplitude of a quantum state some information needed to recover the distance. Then we use amplitude estimation to store coherently this information in a register and further process it to extract the distance. 


We start with the initial state:
$$
\ket{\phi_{ij}} := \ket{i} \ket{j} \frac{1}{\sqrt{2}}(\ket{0}+	\ket{1})\ket{0}
$$$

Then, we query the state preparation oracle (like a [QRAM](qram)) controlled on the third register to perform the mappings 
$\ket{i}\ket{j}\ket{0}\ket{0} \mapsto \ket{i}\ket{j}\ket{0}\ket{v\_i}$$and$$\ket{i}\ket{j}\ket{1}\ket{0} \mapsto \ket{i}\ket{j}\ket{1}\ket{c\_j}$. 
The state becomes:

$$
\frac{1}{\sqrt{2}}\left( \ket{i}\ket{j}\ket{0}\ket{v_i} + \ket{i}\ket{j}\ket{1}\ket{c_j}\right)
$$

We apply an Hadamard gate on the the third register to obtain: 

$$\ket{i}\ket{j}\left(\frac{1}{2}\ket{0}\left(\ket{v_i} + \ket{c_j}\right) + \frac{1}{2}\ket{1}\left(\ket{v_i} - \ket{c_j}\right)\right)$$

Just note that, for a given $$i$$ and $$j$$, the probability of reading$$\ket{1}$$when the third register is measured is,
$$
p_{ij}(1) =  \frac{1}{4}(2 - 2\braket{v_i,c_j}) =  \frac{1}{4} d^2(\ket{v_i}, \ket{c_j}) =  \frac{1 - \langle v_i | c_j\rangle}{2}
$$
which is proportional to the square distance between the two normalised vectors. In the following, instead of measuring the register, we perform$$L$$times in parallel amplitude amplification. Note that we can rewrite$$\ket{1}\left(\ket{v_i} - \ket{c_j}\right)$$as$$\ket{y_{ij},1}$$(by swapping the registers), and hence we have the final mapping


$$A: \ket{i}\ket{j} \ket{0} \mapsto \ket{i}\ket{j}(\sqrt{p_{ij}}\ket{y_{ij},1}+\sqrt{1-p_{ij}}\ket{G_{ij},0}) $$


where the probability$$p_{ij}$$is proportional to the square distance between the normalised vectors and$$G_{ij}$$is a garbage state. Note that the running time of$$A$$is$$T_A=\tilde{O}(T)$$. 


Now that we know how to perform$$A$$we need to encode this probability in a quantum register with high probability. For this, we use use amplitude estimation {% cite brassard2002quantum %} with the unitary$$A$, thus, creating a coherent unitary that maps:


$$
\mathcal{U}: \ket{i}\ket{j}  \ket{0} \mapsto \ket{i}\ket{j} \left( \sqrt{\alpha}  \ket{ \overline{p_{ij}}, G, 1} + \sqrt{ (1-\alpha ) }\ket{G', 0}  \right) 
$$$

where$$G, G'$$ are garbage registers,$$|\overline{p_{ij}} - p_{ij}  |  \leq \epsilon$$and$$\alpha \geq 8/\pi^2$$. 
The unitary $$\mathcal{U}$$ requires $$P$$ iterations of$$A$$with$$P=O(1/\epsilon)$$. Amplitude estimation thus takes time$$T_{\mathcal{U}} = \widetilde{O}(T/\epsilon)$$  . 


Finally, we are ready to apply the Median Lemma, which we have learned from {% cite wiebe2014quantum %}. This Lemma allows us to pump up the success probability of a subroutine that fails with constant probability (though I think it can be extended). The idea is simple: we repeat a certain number of time the same experiment and we select the best outcome by taking the median of the results. Then, we undo everything so to clean the register and we have computed almost surely the result we want. The number of time we have to repeat the experiment is logarithmic in the success probability, so can be arbitrarily small. 

In our case, we apply it for the unitary$$\mathcal{U}$$to obtain a quantum state$$\ket{\Psi_{ij}}$$such that, 

$$\|\ket{\Psi_{ij}}-\ket{0}^{\otimes L}\ket{\overline{p_{ij}}, G}\|_2\le \sqrt{2\Delta}$$

Concluding, the running time of the procedure is$$O( T_{\mathcal{U}} \ln(1/\Delta)) = \widetilde{O}( \frac{T }{\epsilon}\log (1/\Delta)  )$$. 

Note that we can easily multiply the value$$\overline{p_{ij}}$$by 4 in order to have the estimator of the square distance of the normalised vectors or compute$$1-2\overline{p_{ij}}$$for the normalized inner product. 

The running time of the procedure is thus:

$O( T_{\mathcal{U}} \ln(1/\Delta)) = O( \frac{T }{\epsilon}\log (1/\Delta)  )$. 

We can estimate the distance or the inner product of the *unnormalized* vectors as well. Since we know the norms of the vectors, we can simply multiply the estimator we got in the previous step with the product of the two norms to get an estimate for the inner product of the unnormalised vectors and a similar calculation works for the distance. Note that the absolute error$$\epsilon$$now becomes$$\epsilon \norm{v_i}\norm{c_j}$$and hence if we want to have in the end an absolute error$$\epsilon$$this will incur a factor of$$\norm{v_i}\norm{c_j}$$in the running time. 


This finish the proof :) 

### References

{% bibliography --cited %}


