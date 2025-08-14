---
layout: post
title:  "Fidelity in quantum kernels"
date:   2025-08-14
---

I've been recently playing with quantum kernels and quantum support vector machines. When it comes to fidelity quantum kernels, the central building block
is to compute fidelity between two data points (respectively between their embedding into the Hilbert space). The idea is pretty simple. Let $\Phi$ be a
mapping of a data vector $x$ into the Hilbert space (basically, it is the quantum circuit that changes the state vector in a way that depends on our data). 
When we have two data points $x_1$ and $x_2$, we need to calculate fidelity between $\Phi(x_1)$ and $\Phi(x_2)$. Suppose, that our quantum state is initialized
into all zeros $|0\dots 0\rangle$. We first apply $\Phi(x_1)$ and than inversed operation to $\Phi(x_2)$. If the data points are the same or really similar, than $\Phi(x_1)$ and $\Phi(x_2)$ should be pretty close to each other. Hence, if we applied inversely the second operation, we should get back to the initial all zeros state $|0\dots 0\rangle$. When we perform measurement, how many times out of all the shots did we obtain bitstring `0...0`? That gives is the estimate of probability, which is the fidelity we aim for.

So where is the problem? Well, the Hilbert space is huge and the mapping $\Phi$ is a complex quantum circuit. The noise is really non-negligible, hence we don't
get the measurement statistics we seek. Image really oversimplified scenario. The computation is error-less but we have small probability $q$ for a read-out 
 error. (That is, we should measure $|0\rangle$ (or $|1\rangle$) but sometimes the results get flipped with probability $q$.) The errors are independent
across different qubits (again, unrealistic oversimplification). Than, with high probability, when we should measure all zeros, at least one of the positions gets a bit-flip, and we are screwed:/ We obtained measurement statistics with significantly reduced counts for all zeros bitstring `0...0`.
Specifically, the probability of correctly getting bitstring `0...0` is just $$(1-q)^n.$$

Well, I was thinking how to at least a little bit fix this issue. My thought process goes as follows. I wouldn't just count how many times I obtained bitstring `0...0` but also bitstrings with Hamming distance $1$ from `0...0` (Let's call these bitstrings $S_1$.) Then I would artificially inflate the `0...0` count by
some fraction of $S_1$ count, which could "reverse" the leakage, that occured because of the read-out errors. Well, let's try to calculate it (we will get a
little bit different outcome but the logic stays the same).

Let:
* $p_0$ be the true probability of measuring `0...0` in a error-less scenario
* $p_1$ be the true probability of measuring string from $S_1$ in a error-less scenario
* I will also use $p_2$, $p_3$, $\dots$, which is a natural extension of the notation above

Next, let:
* $\hat{p}_0$ be the estimate of $p_0$ we obtained from the measurement statistics (here, the estimates are corrupted by the read-out error)
* $\hat{p}_1$ be the estimate of $p_1$, etc...

The following expression gives us a probability of measuring bitstring $s$ eventhough it should have been bistring $o$:

$$q^H(s, o)(1-q)^n-H(s, o),$$ where $H(\dot, \dot)$ is a Hamming distance and $n$ is num. of qubits (lenght of the bitstring).









Next


