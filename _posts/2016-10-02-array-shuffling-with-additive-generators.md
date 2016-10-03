---
author: yifanlu
comments: true
math: true
date: 2016-10-02
layout: post
slug: array-shuffling-with-additive-generators
title: Array Shuffling with Additive Generators
categories:
- Technical
tags:
- math
- number theory
- latex
- algorithm
---

I was working on unit tests for a project and I wanted a fast and easy way to create random permutations of a range of numbers. That reminded me of some things I've learned in elementary number theory that I thought I might share with you. There is nothing new or non-trivial in this post, but I am always excited about sharing a concrete application for abstract mathematics.

Let's start with the code first.

```c
/**
 * @brief      Creates a random permutation of integers 0..limit-2
 *
 *             `limit` MUST BE PRIME! `ordering` is an array of size limit-1.
 *
 * @param[in]  limit     The limit (MUST BE PRIME). Technically another 
 *                       constraint is limit > 0 but 0 is not prime ;)
 * @param[out] ordering  An array of permutated indexes uniformly distributed
 */
static inline void permute_index(int limit, int ordering[limit-1]) {
  ordering[0] = rand() % (limit-1));
  for (int i = 1; i < limit-1; i++) {
    ordering[i] = (ordering[i-1] + ordering[0] + 1) % limit;
  }
}
```

This function picks, uniformly at random, an array that is a permutation of $$[0, 1, \dots, limit-1]$$. Additionally, it has the following nice properties:

  * The algorithm is optimal in both time and space complexity. Any algorithm that permutes $$n$$ elements must write out all the element. That means we take $$\Theta(n)$$ time and $$\Theta(1)$$ scratch space. We can also modify this into a streaming algorithm. For example, if we are using Python, we can use `yield` to permute an extremely large data set without having to store "seen" elements.
  * The marginal distribution is uniform (assuming `rand()` is uniform, which isn't technically true). This will be proven in the end. An important note: the permutation distribution is _not_ uniform. This will not generate all possible permutations.
  * The code is small and simple enough to copy-paste. You can easily modify it to permute any array of items (not just sequential numbers).

The runtime proof is trivial and will be omitted (it's a single `for` loop). The rest of this post will be a proof of correctness and assumes no previous knowledge of group theory.

First, some preliminaries. A _group_ is simply a set of elements and an operation that works on those elements that satisfies some basic properties. For example, classic addition over real numbers would be considered a group. That is because when you add any two real numbers, you get another real number (this is called the _closure_ property). Multiplication over real numbers is also a group. Addition over only integers is also a group. There are three other properties that are necessary to make an operation and a set a group. They are _associativity_ (ex: $$(1+2)+3=1+(2+3)$$), existence of an _identity_ (ex: $$5+0=5$$ makes $$0$$ an identity), and _invertibility_ (ex: $$5$$ and $$-5$$ are inverses in the additive group of integers because $$5+(-5)=0$$ where $$0$$ is the identity). As another example, let's look at the group of real numbers under multiplication. We have closure because any two numbers will multiply to another number. Associativity of multiplication is something you learned in grade school. The identity of real number under multiplication is $$1$$ because $$1*x=x$$. Finally, for any real number, the inverse is also a real number. For example $$4*0.25=1$$, which is the identity. A classic non-group is the integers over multiplication because most integers do not have an integral inverse.

All of this may seem basic and you might be asking "what is the point?" Well, the idea is that we intuitively "know" how addition and multiplication works. We "know" how to add numbers and we "know" that the number we add up to is still a number. However, by formalizing our intuitions into something more solid, we are able to build up ideas that may not be as obvious. There are many neat and bizarre groups that mathematicians study, but this post will focus on the next most basic group: modular arithmetic. If you are a programmer, chances are that you have worked with modular arithmetic. You might have written code like `int i = (x + y) % 5; // choose one of 5 slots`. That is modular arithmetic. In math, we typically do not use the percent symbol but instead write $$x + y \pmod 5$$ to denote addition modulo 5. The easy way to think about modular arithmetic is in terms of remainders. $$3+4 \pmod 5 \equiv 2 \pmod 5$$ because $$3+4=7$$ and $$7/5$$ is 1 with a remainder of 2.

Here's the key observation: for any integer, $$q$$, we can create a group $$\mathcal{G}_q=(\{0,\dots,q-1\},+)$$ for the modular arithmetic over $q$ (you can easily confirm the four properties to yourself). In fact this group is called a _cyclic group_ because you only need one element (in this case, the number $$1$$) to get every other element by repeatedly applying the group operation (addition). As an example, consider the group $$\mathcal{G}_6$$. To get every element, we have $$1+1=2 \pmod 6$$,$$2+1=3 \pmod 6$$,$$5+1=0 \pmod 6$$ and so on. We say that $$1$$ _generates_ $$\mathcal{G}_6$$.

This applies to any integer, but now let's just consider prime numbers, $$p$$. From above, we know that $$\mathcal{G}_p$$ is cyclic. That means it can be generated by $$1$$. However, it can be generated by $$2$$ as well. Here's an example for $$\mathcal{G}_5$$: $$2+2=4 \pmod 5$$, $$4+2=1 \pmod 5$$, $$1+2=3 \pmod 5$$, $$3+2 = 0 \pmod 5$$, $$0+2 = 2 \pmod 5$$. But, it's not just $$2$$. $$3$$ and $$4$$ also generate $$\mathcal{G}_5$$! You should try it out yourself. This leads us to our first theorem.

**Theorem 1** For any prime number, $$p$$, any integer $$x$$ where $$1 \leq x < p$$ will generate $$\mathcal{G}_p$$.

Before proving that, we will prove the following lemma that will be helpful in the theorem.

**Lemma 1** For any prime number, $$p$$ and any integer $$x \ne 0 \pmod p$$, there exists a $$r \pmod p$$ where $$x \ne r \pmod p$$ and can be produced only by adding $$x$$ to itself.

_Proof._ We showed above that this works $$x=1$$ so we will focus on the case of $$x \ne 1$$. Take $$x \pmod p$$ and add it $$y$$ times until the first time we go over $$p$$. That means $$x \cdot (y-1) \leq p$$ and $$x \cdot y > p$$. Since $$p$$ is prime, it must be that $$x \cdot (y-1) \ne p$$ also and therefore $$x \cdot (y-1) < p$$. Let $$r \equiv x \cdot y \pmod p$$. (I'm being sloppy with the math here by assuming $$x \cdot y$$ as the shorthand for adding $$x$$ to itself for $$y$$ times). This should be the first time we see $$r$$. Why is this? Well, let's look at the division/remainder definition of modular arithmetic again. We see that $$x \cdot y$$ is $$q \cdot p + r$$. Since $$y$$ the smallest number of times we add $$x$$ before reaching a number larger than $$p$$, we know that $$q = 1$$ (otherwise $$x > p$$ which contradicts our construction of $$x$$). So we have

$$
\begin{align*}
x \cdot y &= p + r \\
x \cdot y - r &= p \\
\end{align*}
$$

This is just a linear equation with known variables $$x$$, $$y$$, and $$p$$. Which means there exists at most one solution for $$r$$ and $$r \ne x$$ since $$p$$ is prime. $$\blacksquare$$

What's the upshot? We now have two unique elements! Only $$p-2$$ more to go. We can build the elements inductively.

**Lemma 2** Given unique elements $$S=\{r_1 \pmod p, r_2 \pmod p, \dots, r_{n-1} \pmod p\}$$ with $n < p$, we can find another unique element $$r_n \pmod p \not \in S$$ using only the group operation on elements in $$S$$.

_Proof._ Note that Lemma 1 is the base case with $$r_1=x$$. Now we prove the inductive case. Consider some fixed $$r_i \ne 0 \pmod p \in S$$. If, by contradiction, for all $$r_j \in S$$ we have $$r_i + r_j \pmod p \in S$$ then it must be the case that

$$
\begin{align*}
\sum_{r_j \in S} r_i + r_j \equiv \sum_{r_k \in S} r_k \pmod p
\end{align*}
$$

which comes from summing together all the relations. However, this means

$$
\begin{align*}
r_i |S| \equiv 0 \pmod p
\end{align*}
$$

contradicting our assumption that $$p$$ is prime. So there must be some $$r_i + r_j \not \in S$$. $$\blacksquare$$

Theorem 1 directly follows from this. An exercise to the reader is to show how this reduces to the algorithm presented at the beginning. What's left is to show that by picking the generator uniformly, we can get a random permutation of the elements.

**Theorem 2** Let $$S=\{1,\dots,p-1\}$$ be the elements for group $$\mathcal{G}_p=(S \cup \{0\},+)$$. If we draw $$X \sim S$$ uniformly and let $$\sigma(x)=X \cdot x \pmod p$$, then for some random $$Y \pmod p$$

$$
\begin{align*}
\Pr_X[ Y=\sigma(x) ] = \frac{1}{|S|}
\end{align*}
$$

_Proof._ Since $$X$$ generates $$\mathcal{G}_p$$, $$\sigma(x)$$ returns a unique value for each unique $$x$$. $$\blacksquare$$

I hope that you caught a glimpse of the wonderful world of number theory and how it might help you with coding. As an exercise to the reader, extend the algorithm and proof to work with any number (not just prime). You might even be able to apply the famous Chinese Remainder Theorem! Don't feel shy to point out the inevitable mistakes in this post. I am also curious if anyone has a simpler (and still elementary) proof. As always, you can put MathJax in comments with something like `$$a^2+b^2=c^2$$`.
