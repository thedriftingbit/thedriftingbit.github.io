---
layout: post
title: "Addition Chains, Part One"
author: <author_id> 
categories: article
math: true
tags: [cryptography,cybersecurity,math]
---

Many cryptographic protocols require that numbers be raised to very large powers. The computers that these protocols are implemented on must be able to calculate these powers quickly and efficiently. This is the first of a series of articles based off of an undergrad paper I wrote about addition chains in mathematics and their role in cryptography. 

## Addition Chains

We will begin with the basic mathematical method behind exponentiation on a computer - the addition chain. An $\textit{addition chain}$ for an integer $\textit{n}$ is a sequence of integers: 

$$
1 = a_{0},a_{1},a_{2}, \dots ,a_{r}=n
$$

with the property that

$$
a_{i}=a_{j} + a_{k}
$$

for some $k \leq j < i$ for all $i = 1,2, \dots ,r.$

This article covers the **binary method**, which is an algorithm for exponentiating. The binary method is one of the most basic computational methods for finding an addition chain for a given number.

## Binary

To start, we will define the binary number system. Binary is a base-2 numbering system that is composed of two potential values, 1 or 0. In computing, these single values are referred to as bits. These bits can be used to represent any given decimal (base-10) number. Just as in decimal, binary has “places” for each 0 or 1 that are defined by powers of 2. In decimal, the “places” are the ones, tens, hundreds, etc., whereas the binary equivalent will be $2^0$, $2^1$, $2^2$, etc.

$$
			\begin{array}{||c | c||} 
            \hline
				\text{Decimal} & \text{Binary}\\
                \hline
				1 & 0000 \\
                \hline
				2 & 0010  \\
                \hline
				3 & 0011 \\
                \hline
				4 & 0100 \\
                \hline
				5 & 0101 \\
                \hline
				6 & 0110 \\
                \hline
				7 & 0111 \\
                \hline
				8 & 1000 \\
                \hline
				9 & 1001 \\
                \hline
				10 & 1010 \\
                \hline
			\end{array}
    $$

$\textbf{Table 1}$: $\text{Decimal numbers from 0 through 10 and their binary representation.}$
{: style="text-align: center"}

In **Table 1**, we have the decimal numbers $1$ through $10$, along with their binary representations. If we look at the binary representation for 0, we see 0000. These binary representations are determined by successive powers of $2$. The right-most binary digit stands for the value $2^0 = 1$, the next is $2^1 = 2$, the next is $2^2 = 4$, and so on. By placing a $1$ in one of these locations, we “turn on” that bit, meaning that the place of the bit is now active. We can add those numbers together to obtain the binary representation of a decimal number. For example, the binary representation of $6$ is $0110$, because we have a $1$ in the $2^1 = 2$ space and a 1 in the $2^2 = 4$ space. The sum of these two numbers is $6$.

### Hamming Weight

The **Hamming weight** of a number is the number of non-zero elements from which the number is composed. In binary, this means the number of times a $1$ occurs in the binary number.

Suppose we want to find the Hamming weight of the binary representation of the number $13$. First, we must find the binary representation of 13. We know that $2^3 = 8$, so we start with the binary string $1000$. Next, we know that $2^2 = 4$, and $8+4 = 12$, so we can add a second $1$ to our binary string to obtain $1100$. Finally, we need to add $2^0 = 1$ to obtain 13, making our final binary string $1101$. Counting the number of times $1$ occurs in the string tells us that the Hamming weight of $1101$ is $3$.

## Binary Method

Before we discuss how the binary method of exponentiating works, we have to define the floor function:

Let $\textit{x}$ be a real number and let $\textit{m}$ be an integer. Then the $\textit{floor function}$ of $\textit{x}$, denoted $\lfloor x \rfloor$, is the 
$$
max\{m \in \mathbb{Z} \mid m \leq x\}.
$$
{: style="text-align: center"}

With that in mind, we can show the right-to-left binary method algorithm for evaluating $x^n$, where $n \in \mathbb Z_{+}$ and $\textit{x}$ is a member of an algebraic system with identity element $1$ and associative multiplication:

&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;$\textbf{A1}$&ensp;&ensp; $Set$ $N = n$, $Y = 1$, $and$ $Z = x$.

&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;$\textbf{A2}$&ensp;&ensp; $($At this point, $x^n = Y\cdot Z^N$.$)$ $Set$ $N = \lfloor N/2 \rfloor$, and at the same time determine 
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;whether $N$ was even or odd. If $N$ was even, skip to step $\textbf{A5}$.

&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;$\textbf{A3}$&ensp;&ensp; $Set$ $Y = Z\cdot Y$.

&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;$\textbf{A4}$&ensp;&ensp; $If$ $N = 0$, the algorithm terminates with $Y$ as the answer.

&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;$\textbf{A5}$&ensp;&ensp; $Set$ $Z = Z\cdot Z$ and return to step $\textbf{A2}$.

As an example, the table below shows how to evaluate $x^{27}$ using the binary method algorithm.

$$
				\begin{array}{||c | c | c | c||} 
                \hline
				 & N & Y & Z\\\hline\hline
				\text{After step A1}& 27 & 1 & x \\ 
				\hline
				\text{After step A2} & 13 & 1 & x \\
				\hline
				\text{After step A3} & 13 & x & x\\
				\hline
				\text{After step A4} & 13 & x & x\\
				\hline
				\text{After step A5} & 13 & x & x^2\\
				\hline
				\text{After step A2} & 6 & x & x^2\\
				\hline
				\text{After step A3} & 6 & x^3 &  x^2 \\
				\hline
				\text{After step A4} & 6 & x^3 & x^2\\
				\hline
				\text{After step A5} & 6 & x^3 & x^4\\
				\hline
				\text{After step A2} & 3 & x^3 & x^4\\
				\hline
				\text{After step A5} & 3 & x^3 & x^8\\
				\hline
				\text{After step A2} & 1 & x^3 & x^8\\
				\hline
				\text{After step A3} & 1 & x^{11} & x^8\\
				\hline
				\text{After step A4} & 1 & x^{11} & x^4\\
				\hline
				\text{After step A5} & 1 & x^{27} & x^{16}\\
				\hline
				\text{After step A2} & 0 & x^{27} & x^{16}\\
				\hline
				\text{After step A3} & 0 & x^{27} & x^{16}\\
				\hline
				\text{After step A4} & 0 & x^{27} & x^{16}\\
				\hline 
                \end{array}
$$  

$\textbf{Table 2}$: $\text{Evaluation of $ x^{27} $ using the binary method algorithm.}$
{: style="text-align: center"}

The next article in this series will explain another part of the exponentiation puzzle - addition chains.