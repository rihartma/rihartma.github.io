---
layout: post
title:  "Fidelity in quantum kernels"
date:   2025-08-14
mathjax: true
---

I've been recently playing with quantum kernels and quantum support vector machines. When it comes to fidelity quantum kernels, the central building block
is to compute fidelity between two data points (respectively between their embedding into the Hilbert space). The idea is pretty simple. Let $\Phi$ be a
mapping of a data vector $x$ into the Hilbert space (basically, it is the quantum circuit that changes the state vector in a way that depends on our data). 
When we have two data points $x_1$ and $x_2$, we need to calculate fidelity between $\Phi(x_1)$ and $\Phi(x_2)$. Suppose, that our quantum state is initialized
into all zeros $|0\dots 0\rangle$. We first apply $\Phi(x_1)$ and then inversed operation to $\Phi(x_2)$. If the data points are the same or really similar, then $\Phi(x_1)$ and $\Phi(x_2)$ should be pretty close to each other. Hence, if we applied inversely the second operation, we should get back to the initial all zeros state $|0\dots 0\rangle$. When we perform measurement, how many times out of all the shots did we obtain bitstring `0...0`? That gives us the estimate of probability, which is the fidelity we aim for.

So where is the problem? Well, the Hilbert space is huge and the mapping $\Phi$ is a complex quantum circuit. The noise is really non-negligible, hence we don't
get the measurement statistics we seek. Imagine a really oversimplified scenario. The computation is error-less (no decoherence, no gate errors, no cross-talk etc...) but we have
small probability $q$ for a read-out (measurement) error. (That is, we should measure $|0\rangle$ (or $|1\rangle$) but sometimes the results get flipped with probability $q$.)
Additionally, assume that the errors are independent
across different qubits (again, unrealistic oversimplification). Then, with high probability, when we should measure all zeros, at least one of the positions gets a bit-flip and we are screwed:/
We obtained measurement statistics with significantly reduced counts for all zeros bitstring `0...0`.
Specifically, the probability of correctly getting bitstring `0...0` is just $$(1-q)^n.$$

Well, I was thinking how to tackle this issue. My thought process goes as follows. I wouldn't just count how many times I obtained bitstring `0...0` but also bitstrings with Hamming distance $1$ from `0...0` (Let's call these bitstrings $S_1$.) Then I would artificially inflate the `0...0` count by
some fraction of $S_1$ count, which could "reverse" the leakage, that occured because of the read-out errors. Well, let's try to calculate it (we will get a
slightly different outcome than I expected).

Let:
* $p_0$ be the true probability of measuring `0...0` in a error-less scenario
* $p_1$ be the true probability of measuring string from $S_1$ in a error-less scenario
* I will also use $p_2$, $p_3$, $\dots$, which is a natural extension of the notation above

Next, let:
* $s_0$ be the estimate of $p_0$ we obtained from the measurement statistics (here, the estimates are corrupted by the read-out error)
* $s_1$ be the estimate of $p_1$, etc...

The following expression gives us a probability of measuring bitstring $s$ even though it should have been bitstring $o$:
$$q^{H(s, o)}(1-q)^{n-H(s, o)},$$ where $H(\cdot, \cdot)$ is a Hamming distance and $n$ is the num. of qubits (length of the bitstring).

It is not hard to derive the relation between $s_?$ and $p_?$

$$s_0 = p_0 (1-q)^n + p_1 q(1-q)^{n-1} + \sum_{i=2}^n p_i q^i (1-q)^{n-i},$$

$$s_1 = p_0 n q(1-q)^{n-1| + p_1 (1-q)^n + \sum_{i=2}^n p_i q^{i-1} (1-q)^{n-i+1}.$$

To simplify the calculations, we omit the third terms in the equations, so we get $s_0 = p_0 (1-q)^n + p_1 q(1-q)^{n-1}$ and $s_1 = p_0 n q(1-q)^(n-1) + p_1 (1-q)^n$.

We want a good estimate of $p_0$. We get it by combining $s_0$ and $s_1$ as $$\hat{p}_0 = a s_0 + b s_1.$$ To compute $a$ and $b$, we want $\hat{p}_0$ to be
unbiased. To achieve that, we solve the system of 2 linear equations so that the coefficient with $p_0$ is $1$ and the coefficient with $p_1$ is $0$. We get

$$a = \frac{(1-q)^{2-n}}{(1-q)^2-n q^2},$$

$$b = -\frac{q(1-q)^{1-n}}{(1-q)^2-nq^2}.$$

Nice, we got an unbiased estimator, but if we plug in some sample numbers, we see that the result is a bit different from the motivation I was thinking about in the beginning of this post.

```python
>>> q = 0.005
>>> n = 100
>>> a = ((1-q)**(2-n)) / ((1-q)**2-n*q**2)
>>> b = -(q*(1-q)**(1-n)) / ((1-q)**2-n*q**2)
>>> a, b
(1.6549694753786401, -0.00831642952451578)
```

Notice, that it is not the case that we would increase the count of `0...0` by some fraction of $S_1$. We just increase the count of `0...0` by some coefficient and decrease it with some fraction
of $S_1$. Huh, not what I expected in the first place. Hence, eventhough this seems like a good estimator, if we don't get $s_0>0$ (it really often happens that $s_0=0$, because we don't have resources to make
sufficient amount of shots... exponentially many...), we get even negative $\hat{p}_0$. How to get out of it? Well, I got something in my mind but it needs to settle down little bit:D

What next? There are several problems and some of them are easy to fix. For example, it is impossible in real computations that the readout errors are independent. There will be some correlations etc..., so it will be insufficient to work only with $s_0$ and $s_1$. We need to extend the model to also consider bitstring with higher Hamming distance from `0...0`.
With the logic I have described, it is straightforward to extend this model to 
$$\hat{p}_0 = a s_0 + b s_1 + c s_2, $$ or even further!. (The computation would be conceptually easy but grueling...)

Next question is about the variance of our estimate. Did we decrease it? There is much to ask and it would be enough even for a paper, but I don't have time right
now, I hope that I will find few days to dive in a bit more :-)
