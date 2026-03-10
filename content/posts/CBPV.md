---
title: "Why is it called Call-By-Push-Value?"
date: 2026-03-08T10:07:53-07:00
slug: 2026-03-08-cbpv
type: posts
draft: false
tags:
  - PLs
---

Call-by-push-value (CBPV) is a programming language that subsumes call-by-name
(CBN) and call-by-value (CBV) semantics, making it useful to reason about
effectful computations. But why is it called that?

## Call-by-name and call-by-value

First, we should motivate CBPV by explaining what CBN and CBV are. To start out
with, we will only consider the basic lambda calculus with no effects. The
important rule here when expressing the lambda calculus's semantics is
$\beta$-reduction: How do we apply a function?

The big idea is that we wish to perform a substitution. The notation $e[e'/x]$
(also written a million different ways, depending where you look) expresses the
operation replacing all free occurrences of the variable $x$ in $e$ with the
term $e'$. Suppose we had some language expressing arithmetic, and we defined
the function $\lambda x. x + x$. We might expect an application of this function
to behave like
$$
(\lambda x. x + x) \, 1 \longrightarrow 1 + 1.
$$
This idea is also expressed by $(x+x)[1/x]$. So surely, we should have a rule in
our semantics that looks like
$$
\frac{e_1[e_2/x] \longrightarrow e_1'}
     {(\lambda x. e_1) \, e_2 \longrightarrow e_1'}
$$
In other words, when we encounter an application (the bottom of the rule), we
should evaluate the function body after substituting the argument (the top of
the rule) and take that to be the result.

But this alone is not enough. What if we had the term
$((\lambda x. \lambda y. x + y) \, 1) \, 2$? This does not match our rule,
because the outer application (in the head position) does not have a lambda
abstraction on the left. It has another application. Then we might also need the
rule
$$
\frac{e_1 \longrightarrow e_1'}{e_1\,e_2 \longrightarrow e_1'\,e_2}
$$
Now, our system will start by reducing the lefthand argument of the application
as far as it can, until it gets a function definition. Then, it will use
our first rule to apply the function to its argument. In a move that
miraculously sheds no light on the situation at all, this system is called
call-by-name.

One drawback to call-by-name is that terms can get really unwieldy. Suppose we
have a function that uses its argument a lot of different times, and we apply it
to a really large term. Now, after $\beta$-reduction, we have duplicated our
large term and we need to reduce it a lot of different times. We could have
saved work by reducing the argument before applying the $\beta$-reduction rule.
We can add the rule
$$
\frac{e_2 \longrightarrow e_2'}
     {e_1 \, e_2 \longrightarrow e_1 \, e_2'}
$$
But now, we've introduced _nondeterminism_ to our system. Given an application,
we don't know whether we should reduce the lefthand side or the righthand side.
It would make sense to fully reduce one side, then fully reduce the other, and
then apply $\beta$-reduction.

To do this, we need some notion of what it means to be fully reduced. We define
a _value_, which is a term that cannot step any further. In the basic lambda
calculus, this is very simple:
$$
v := \lambda x. t
$$
The only terms that can't reduce any further are function definitions. We can
now alter the two rules from CBN to get
$$
\frac{e_1 \longrightarrow e_1'}{e_1 \, v \longrightarrow e_1' \, v}
\qquad
\frac{e[v/x] \longrightarrow e'}{(\lambda x. e) \, v \longrightarrow e'}
$$
Altogether, this system is called call-by-value. These names are slightly weird,
but they make more sense when you think of the distinction as the difference in
what an identifier may be bound to. In call-by-value, identifiers may only be
bound to values, but in call-by-name they may be bound to anything. Therefore,
in some sense an identifier is just the "name" of some arbitrary step in your
program.

## Effects

In the regular lambda calculus, the distinction between CBN and CBV is
irrelevant. Whichever evaluation strategy we pick, terms will always reduce to
the same value. This is because the lambda calculus is "a language of being"
and not of "doing," so the order in which we "do" things doesn't matter.[^1]

What constitutes a "language of doing," then? This is something we refer to as
effects. Often, I hear effects described as anything that requires something of
the outside world. This isn't a bad definition, but I think it's more helpful to
think of them in terms of "doing." With effects, it matters when they happen,
because they _do_ something. A common example of an effect is input/output. A
user knows when something is printed to the terminal. There is nothing in the
terminal and then there is something: something has been _done_. Other examples
of effects are exceptions, random choice, or keeping state. All of these happen
at a discrete point during the execution of a program.

To see why this changes things, let's consider a language that adds random
choice to our lambda calculus with arithmetic. Now, we can express the program
$(\lambda x. x + x) \, (\textsf{random}())$. In call-by-value, this might
evaluate to
$$
(\lambda x. x + x) \, (\textsf{random}()) \longrightarrow_{CBV}
(\lambda x. x + x) \, 4 \longrightarrow_{CBV}
4 + 4 \longrightarrow_{CBV} 8.
$$
In call-by-name, on the other hand, we might get
$$
(\lambda x. x + x) \, (\textsf{random}()) \longrightarrow_{CBN}
\textsf{random}() + \textsf{random}() \longrightarrow_{CBN}
4 + 1 \longrightarrow_{CBN} 5.
$$
It's clear that these now exhibit different behavior. In CBN, we get two
different random numbers. There is a world before and after the
$\textsf{random}()$ function is evaluated, and those worlds are different.
Therefore, evaluating it twice is observably different from evaluating it once.

This is a problem. This means that systems with this tiny difference in their
evaluation strategies are totally different, and we need to prove things about
them separately. Enter call-by-push-value.

## Call-by-push-value

The first thing to note about call-by-push-value is that it has two classes of
terms: values and computations. In the style used above, we say that values
_are_ and computations _do_. Corresponding with these two classes, there are
also two classes of types:
$$
\begin{align*}
&\text{value types } &A &:= U\underline{B} \mid \sum_{i \in I} A_i \mid 1 \mid A \times A \\
&\text{computation types } &\underline{B} &:= F A \mid \prod_{i \in I} \underline{B}_i \mid A \rightarrow \underline{B}
\end{align*}
$$
A value of type $U\underline{B}$ is a "thunk:" a suspended computation of type
$\underline{B}$. A computation of type $F A$ produces a value of type $A$ (after
some amount of doing).

Under this classification, each function "knows" whether it behaves like CBN or
CBV. A "call-by-value"-style function might have a type like $1 \rightarrow
\underline{B}$, where the argument is a value type. On the other hand, a
"call-by-name"-style function will have a type like $U\underline{B} \rightarrow
\underline{B}$, where its argument is a thunk. This function will have to force
the thunk before it can do anything with its result, which is like a
call-by-name execution style where the argument is evaluated after it is bound
inside the function body.

One interesting thing to note is that there are two different product types. The
value type $A \times A$ is a pattern-match product, so its elimination form is
in some sense "immediate." The computation type $\prod_{i \in
I}\underline{B}_i$, on the other hand, is a projection product, so its
elimination form requires a step of computation.

To apply a function $M$ to a value $V$, CBPV uses the notation $V'M$. This will
make sense momentarily.

### CK-Machine

There are many intersting ways to represent the semantics of CBPV, but one very
natural form is the CK-machine. Each step of evaluation updates the CK-machine's
configuration, which has the form $M,K$ where $M$ (the inside) is the term we
are evaluating and $K$ (the outside) is a stack. Using this format, function
application requires two rules. The first sees an application and pushes the
argument onto the stack:
$$
\begin{align*}
&V'M \qquad &K \\
\rightsquigarrow \, &M &V :: K
\end{align*}
$$
The second rule sees a function definition and pops its argument off the stack:
$$
\begin{align*}
&\lambda x. M \qquad &V :: K \\
\rightsquigarrow \, &M[V/x] &K
\end{align*}
$$
Now, we see that we can read $'$ as "push onto the stack" and $\lambda$ as "pop
from the stack."

So instead of binding variables to identifiers, like CBN and CBV, we can think
of CBPV's evaluation style as purely pushing and popping from the stack. We can
only push values onto the stack, but this can simulate CBN-style computation by
pushing a suspended computation in the form of a thunk.

And that's my whirlwind tour of CBPV! There's a lot I didn't cover, which I
might write a second post about. My main takeaway is that it is a language that
represents effects naturally. Effects can be added to languages with CBN or CBV
evaluation strategies, but it feels in some sense like you lose control of the
order of operations. By allowing you to suspend an effectful computation, CBPV
puts you in full control of when things happen. That's why we say it subsumed
CBN and CBV, but I think that's not the most important thing going on.

[^1]: Credit Paul Blain Levy in his PLMW talk at POPL '26. I think this is a very
nice way to think of things, and I cannot take credit for it myself.
