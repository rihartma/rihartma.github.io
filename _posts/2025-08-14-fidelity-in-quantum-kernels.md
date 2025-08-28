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
Specifically, the probability of correctly getting bitstring `0...0` is just $(1-q)^n.$

Well, I was thinking how to tackle this issue. My thought process goes as follows. I wouldn't just count how many times I obtained bitstring `0...0` but also bitstrings with Hamming distance $1$ from `0...0` (Let's call these bitstrings $S_1$.) Then I would artificially inflate the `0...0` count by
some fraction of $S_1$ count, which could "reverse" the leakage, that occured because of the read-out errors. Well, let's try to calculate it (we will get a
slightly different outcome than I expected).

Let:
* $p_0$ be the true probability of measuring `0...0` in a error-less scenario
* $p_1$ be the true probability of measuring string from $S_1$ in a error-less scenario
* I will also use $p_2$, $p_3$, $\dots$, which is a natural extension of the notation above

Next, let:
* $s_0$ be the noisy probability of measuring `0...0`. That is the probability we would obtain, if we performed infinitely many shots. Similarly, define $s_1$, $\dots$.
* $\hat{s}_0$ be the estimate of $s_0$ where we performed $N$ shots. Again, define the same way $\hat{s}_1$, $\dots$

The following expression gives us a probability of measuring bitstring $s$ even though it should have been bitstring $o$:
$$q^{H(s, o)}(1-q)^{n-H(s, o)},$$
where $H(\cdot, \cdot)$ is a Hamming distance and $n$ is the num. of qubits (length of the bitstring).

It is not hard to derive the relation between $s_?$ and $p_?$

$$s_0 = p_0 (1-q)^n + p_1 q(1-q)^{n-1} + \dots,$$

$$s_1 = p_0 n q(1-q)^{n-1} + p_1 \bigl( (1-q)^n + (n-1) q^2 (1-q)^{n-2}\bigr) + \dots.$$

To simplify the calculations, we keep only the first and the second term in the equations, so we get $s_0 = p_0 (1-q)^n + p_1 q(1-q)^{n-1}$ and $s_1 = p_0 n q(1-q)^{n-1} + p_1 \bigl( (1-q)^n + (n-1) q^2 (1-q)^{n-2}\bigr)$.

An explanation of these four terms:
* $p_0 (1-q)^n$ is the probability of measuring all-zeros without any unwanted bit-flip.
* $p_1 q(1-q)^{n-1}$ is the probability of measuring all-zeros eventhough it should have been string with Hamming weight 1. The only "1" in the string got flipped to "0" and the rest remained unchanged.
* $p_0 n q (1-q)^{n-1}$ is the probability of measuring string with Hamming weigth 1 eventhoug it should have been all-zeros string. That is, one of the zeros got bit-flipped.
* $p_1 \bigl( (1-q)^n + (n-1) q^2 (1-q)^{n-2}\bigr)$ is the probability of two disjoint events
    * either we measured string with Hamming weight 1 without any bit-flip
    * or we measured string with Hamming weight 1, but two bit-flips occured. The "1" got flipped to "0", and one "0" got flipped to "1".
* Notice that for simplicity, all the terms with Hamming weight 2, 3, etc. are omitted from the calculation.

We want a good estimate of $p_0$. We get it by combining $s_0$ and $s_1$ as $\hat{p}_0 = a s_0 + b s_1.$ But, of course, we don't know what $s_0$ and $s_1$ are, so we have to use its estimates

$$\hat{p}_0 = a \hat{s}_0 + b \hat{s}_1.$$

To compute $a$ and $b$, we want $\hat{p}_0$ to be
unbiased. To achieve that, we solve the system of 2 linear equations so that the coefficient with $p_0$ is $1$ and the coefficient with $p_1$ is $0$. This system of lin. equations looks like:

$$
\begin{pmatrix}
(1-q)^n         & n q (1-q)^{n-1} \\
q(1-q)^{n-1}    & (1-q)^n + (n-1) q^2 (1-q)^{n-2}
\end{pmatrix}
\begin{pmatrix}
a \\
b
\end{pmatrix}
=
\begin{pmatrix}
1 \\
0
\end{pmatrix}
$$

Nice, once we compute $a$ and $b$, we get an unbiased estimator. Let's see how the coefficients look like if we plug in some numbers:

```python
>>> q = 0.005
>>> n = 100
>>> a = -((n-1)*q**2 + (1-q)**2) / ((1-q)**n * (2*q - 1))
>>> b =  q / ((1-q)**(n-1) * (2*q - 1))
>>> a, b
(1.6549590276028556, -0.00829563845070449)
```

Notice, that it is not the case that we would increase the count of `0...0` by some fraction of $S_1$ (as I've initially though in the motivation). We just increase the count of `0...0` by some coefficient and decrease it with some fraction
of $S_1$. Huh, not what I expected in the first place. Hence, eventhough this seems like a good estimator, if we don't get $\hat{s}_0>0$ (it really often happens that $\hat{s}_0=0$, because we don't have resources to make sufficient amount of shots... exponentially many...),
we get even negative $\hat{p}_0$. How to get out of it? Well, I got something in my mind but it needs to settle down little bit:D

What next? There are several problems. For example, it could be insufficient to work only with $s_0$ and $s_1$. We need to extend the model to also consider bitstring with higher Hamming distance from `0...0`.
With the logic I have described, it is straightforward to extend this model to 
$$\hat{p}_0 = a \hat{s}_0 + b \hat{s}_1 + c \hat{s}_2, $$
or even further!. (The computation would be conceptually easy but grueling so I omit it for now...)

There is much to investigate and I think it would be enough even for a paper, but I don't have time right
now, I hope that I will find few days to dive in a bit more :-)

---

(2025-08-15)
Let me extend the post by going with the analysis further. First, let's calculate variance of 
${\hat{p}_0 = a \hat{s}_0 + b \hat{s}_1}$.

$$\operatorname{Var}(\hat{p}_0) = a^2 \operatorname{Var}(\hat{s}_0) + b^2 \operatorname{Var}(\hat{s}_1) + 2ab \operatorname{Cov}(\hat{s}_0, \hat{s}_1)$$

Let $I_i$ be an indicator r.v. that indicates whether we got `0...0` on i-th shot. It holds that
$P(I_i=1)=s_0$
and
${\hat{s}}_0 = \frac{1}{N}\sum_{i=1}^N I_i$
.

Now, we calculate $\operatorname{Var}(\hat{s}_0)$.

$$\operatorname{Var}(\hat{s}_0) = \frac{1}{N^2} N \operatorname{Var}(I_1) = \frac{s_0 (1-s_0)}{N}$$

Similarly, we get

$$\operatorname{Var}(\hat{s}_1) = \frac{s_1 (1-s_1)}{N}.$$

It remains to calculate the covariance (we use again indicator r.v's.)

$$\operatorname{Cov}(\hat{s}_0, \hat{s}_1) = \mathbb{E}[\hat{s}_0 \hat{s}_1] - s_0 s_1 = \dots = -\frac{1}{N}s_0s_1$$

So, we have $$\operatorname{Var}(\hat{p}_0) = a^2 \operatorname{Var}(\hat{s}_0) + b^2 \operatorname{Var}(\hat{s}_1) + 2ab \operatorname{Cov}(\hat{s}_0, \hat{s}_1) = \frac{1}{N} \bigl( a^2 s_0 (1-s_0) + b^2 s_1 (1-s_1) -2 a b s_0 s_1 \bigr).$$

Well, it seems that we increased (with realistic values of variables) the variance, hence we shouldn't use the model where the bias reduction is overshadowed by the increased in variance. Aaa, bias-variance tradeoff...

I plan to extend this a little bit later in some other post.



 