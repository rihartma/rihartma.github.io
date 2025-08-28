---
layout: post
title:  "Error mitigation technique"
date:   2025-08-28
mathjax: true
---

This post is follow-up of my previous post about fidelity in quantum kernels. I highly recommend to go through it before diving into this idea. This post is basically extension of the ideas I've discussed there. We will generalize the technique and get an interesting error mitigation technique for quantum computers!

We have derived the following two relations:

$$s_0 = p_0 (1-q)^n + p_1 q(1-q)^{n-1} + \dots,$$

$$s_1 = p_0 n q(1-q)^{n-1} + p_1 \bigl( (1-q)^n + (n-1) q^2 (1-q)^{n-2}\bigr) + \dots.$$

First, we will generalize these relations. Let $\mathbf{A}_{ij}$ be a probability of measuring string with Hamming weight $j$ eventhough it was supposed to have Hamming weight $i$.
How did this happen? Well, some of the "1"s in the string got flipped to "0"s and some of the "0"s got flipped to "1"s. We get 

$$\mathbf{A}_{ij} = \sum_{k=\max(0, j-n+i)}^{\min(i,j)} {i \choose k} q^{i-k} (1-q)^{k} {n-i \choose j-k} q^{j-k}(1-q)^{n-i-j+k}.$$

This complicated formula needs some clarification:
* ${i \choose k} q^{i-k} (1-q)^{k}$ is probability of flipping $i-k$ "1"s into "0"s and the rest of "1"s keeps unchanged.
* ${n-i \choose j-k} q^{j-k}(1-q)^{n-i-j+k}$ is probability of flipping $j-k$ "0"s into "1"s and the rest of $n-i-j+k$ "0"s keeps unchanged.
* The weird looking lower and upper indices in the summation ensure feasibility, we need $0\le k \le i$ and
$0 \le j-k \le n-i$.

<!-- 
$$\begin{aligned}
s_0 &= \sum_{i=0}^n p_i q^i (1-q)^{n-i} \\
s_1 &= p_0 n q (1-q)^{n-1} \\
    &+ p_1 \bigl( (1-q)^n + (n-1) q^2 (1-q)^{n-2}\bigr) \\
    &+ p_2 \bigl( {2 \choose 1}q(1-q)^{n-1}   +   ((n-2)q^3(1-q)^{n-3}) \bigr) \\
    &+ p_3 \bigl( {3 \choose 2}q^2(1-q)^{n-2} + (n-3)q^4(1-q)^{n-4} \bigr) \\
    &\enspace \vdots \\
    &= \sum_{i=0}^n p_i \bigl( i q^{i-1}(1-q)^{n-i+1} + (n-i)q^{i+1}(1-q)^{n-i-1} \bigr) \\
% \begin{equation}
s_j &= \sum_{i=0}^n p_i \bigl( {i \choose k} (1-q)^k q^{i-k} {n-i \choose j-k} q^{j-k} (1-q)^{n-i-j+k}
\bigr)
% \end{equation} 
\end{aligned}$$ -->

So, columns of matrix $\mathbf{A}_{:j}$ form coefficients for $s_j$. We seek a linear estimator

$$\hat{p}_0 = \sum_{j=0}^n w_j \hat{s}_j,$$
where the coefficients $\mathbf{w}$ are solution to the system of linear equations
$\mathbf{Aw} = \mathbf{e}_0$ with $\mathbf{e}_0 = \begin{pmatrix} 1 \\ 0 \\ \vdots \\ 0\end{pmatrix}.$ This choice  $\mathbf{e}_0$ ensures, that our estimator is unbiased. Notice that, if we want less complex model, we can limit ourselves easily just to first $K$ weights (in the previous blog post, $K=2$)
$$\mathbf{A_{0:K}w_{0:K}} = \mathbf{e}_0,$$
this introduces some small bias. We need to investigate a bit more, what effects does this have.

The motivation for such error mitigating technique was obtaining correct probability for measuring "all-zeros" strings (it is used in fidelity quantum kernels). But notice, that this method is quite universal, we can use it for any string, not just `0...0`. The only difference is that we wouldn't be interested in Hamming weight anymore but we would be interested in Hamming distance from the particular bit-string we are interested in.

I need to think this through a bit more, to see, how does it behave and if it actually is useable in real quantum computers. I think, it has got some promises. When I find more time, I will try to extend the analysis of this error mitigation technique.
