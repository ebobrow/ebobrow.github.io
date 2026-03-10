---
title: "Semantics"
date: 2025-07-26T16:58:31+02:00
slug: 2025-07-26-semantics
type: posts
draft: false
tags:
  - PLs
---

There are two important aspects in the study of programming languages as formal
mathematical objects: syntax (or statics) and semantics (or dynamics). The syntax
specification includes an (often inductive) definition of well-formed terms in
the language, as well as, in typed languages, a specification of the typing
judgment. There is plenty of interesting stuff to study in syntax, but this blog
post is about semantics. Semantics is about assigning actual meaning to terms in
the language. This is important for reasoning about, for example, correctness,
equivalence, or termination. Depending on your language and use case, there are
many different ways to specify the semantics. This post is a survey of what I
have experienced to be the most common ones.

<!-- In this post I will explain the common methods, using the simply typed lambda -->
<!-- calculus as an example throghout. Briefly, the statics are given below: -->

<!-- $$ -->
<!-- \begin{align*} -->
<!--     t &:= () \mid (t_1,t_2) \mid \pi_1(t) \mid \pi_2(t) \mid \lambda x.t \mid t_1 \, t_2 \\ -->
<!--     \tau &:= \text{unit} \mid \tau_1 \times \tau_2 \mid \tau_1 \rightarrow \tau_2 -->
<!-- \end{align*} -->
<!-- $$ -->

<!-- $$ -->
<!--     \frac{}{\Gamma \vdash ():\text{unit}} -->
<!--     \quad -->
<!--     \frac{}{\Gamma,x:\tau \vdash x:\tau} -->
<!-- $$ -->

<!-- $$ -->
<!--     \frac{\Gamma \vdash t_1 : \tau_1 \quad \Gamma \vdash t_2 : \tau_2} -->
<!--          {\Gamma \vdash (t_1,t_2) : \tau_1 \times \tau_1} -->
<!--     \quad -->
<!--     \frac{\Gamma \vdash t : \tau_1 \times \tau_2}{\Gamma \vdash \pi_1(t) : \tau_1} -->
<!--     \quad -->
<!--     \frac{\Gamma \vdash t : \tau_1 \times \tau_2}{\Gamma \vdash \pi_2(t) : \tau_2} -->
<!-- $$ -->

<!-- $$ -->
<!--     \frac{\Gamma \vdash t_1 : \tau_1 \rightarrow \tau_2 \quad \Gamma \vdash t_2 : \tau_1} -->
<!--          {\Gamma \vdash t_1 \, t_2 : \tau_2} -->
<!--     \quad -->
<!--     \frac{\Gamma, x:\tau_1 \vdash t:\tau_2}{\Gamma \vdash \lambda x.t : \tau_1 \rightarrow \tau_2} -->
<!-- $$ -->

<!-- This variant of the simply typed lambda calculus is equipped with products and -->
<!-- the unit type (the nullary product). -->

## Operational

One common method is operational semantics, which specifies a transition system
for a language. There are two flavors of operational semantics: small-step (or
structural) and big-step (or natural). In small-step semantics, we define a
relation $\rightarrow$ specifying the behavior of individual parts of a program.
For example, a language with pairs may contain the rules
$$
\frac{e_1 \rightarrow e_1'}{(e_1,e_2) \rightarrow (e_1', e_2)}
\qquad
\frac{e_2 \rightarrow e_2'}{(v_1,e_2) \rightarrow (v_1, e_2')}
$$
where $v_1$ denotes a value (a term which cannot be reduced any further).
Essentially, these rules state that to evaluate a pair, you must "reach in" to
the subterms of the pair and evaluate those. The additional requirement that the
first subterm be a value in the rule on the right is optional and only to ensure
that the operational semantics are deterministic.

One important rule in any lambda-calculus system is function application, or
$\beta$-reduction. In small-step semantics, it is expressed as
$$
\frac{e_1[e_2/x] \rightarrow e_1'}{(\lambda x. e_1) \, e_2 \rightarrow e_1'}
$$
where $e_1[e_2/x]$ represents the substitution of $e_2$ for $x$ in $e_1$. If we
are using _call-by-value_ semantics, we would first require that $e_2$ reduce to
a value before applying $\beta$-reduction. On the other hand, in _call-by-name_
semantics, we substitute the argument into the function body before reducing it
at all.

With small-step operational semantics, we can then define the relation
$\rightarrow^* $ as the reflexive, transitive closure of $\rightarrow$. The
judgment $e \rightarrow^* v$ expresses that $e$ eventually reduces to a value by
some chain of reductions $e \rightarrow e_1 \rightarrow \cdots \rightarrow e_n
\rightarrow v$.

The other flavor is big-step operational semantics. This defines the relation
$\Downarrow$ which essentially serves the same role as $\rightarrow^* $, by
reducing a term all at once. The analogous big-step rule for pairs would be:
$$
\frac{e_1 \Downarrow v_1 \qquad e_2 \Downarrow v_2}{(e_1,e_2) \Downarrow (v_1,v_2)}
$$
With a single rule application, we are guaranteed to have reduced the term to
its final value (though of course the derivation tree may be arbitrarily tall).
Because of this property, the judgment need not be defined to take terms as its
right-hand side. Since we will not be chaining this judgment, we can output
whatever we want as the final value of a term. For example, in a language with
arithmetic, we may wish to output natural numbers. We can define the judgment $e
\Downarrow n$, where the rule for addition may be:
$$
\frac{e_1 \Downarrow n_1 \qquad e_2 \Downarrow n_2}{e_1+e_2 \Downarrow n_1+n_2}
$$
The subtlety here is that the $+$ on the left-hand side of the conclusion is a
term-level addition (meaning it is only a symbol with no inherent meaning)
whereas the $+$ on the right-hand side is addition in the natural numbers, which
does have a meaning.

However, we run into issues if we try to define $\beta$-reduction in
call-by-value semantics. We want to write
$$
\frac{e_2 \Downarrow n_2 \qquad e_2[n_2/x] \Downarrow n_1}
     {(\lambda x. e_1)\,e_2 \Downarrow n_1}
$$
but the substitution is malformed: How can we substitute a natural number for a
term? The solution is to introduce a _state_ that maps variables to values. In
this case, the signature of the state is $S : Var \rightarrow \mathbb{N}$. Now,
our big-step judgment is of the form $(e,S) \Downarrow n$ and our
$\beta$-reduction rule is:
$$
\frac{(e_2,S) \Downarrow n_2 \qquad (e_1, S + \{x \mapsto n_2\}) \Downarrow n_1}
     {((\lambda x. e_1)\,e_2, S) \Downarrow n_1}
$$
Here, the state plays the same role as the typing context in typing judgments:
it can "remember" the argument in a rule premise that decomposes the application
term.

## Equational

Very similar to operational semantics is the notion of equational semantics.
Now, our judgments are of the form $\Gamma \vdash e_1 = e_2 : \tau$. The
difference between this and small-step operational semantics is that there is no
"directionality." Instead of looking at chains of evaluations, we can instead
look at equivalence classes of terms, allowing us to replace any term with its
fully evaluated form.

I don't have much to say about equational semantics because I don't know a lot
about them but I'm sure they're interesting.

## Denotational

In the above two methods, we never actually assign meaning to terms. We give
them behavior based on their syntactic structure, which in a way gives them
meaning. But another method is denotational semantics, which seeks to actually
map terms into some domain that we already know about. We define an
interpretation mapping $[\![ - ]\!]$ which takes terms to elements of our domain
of interest.

Usually the interpretation is defined recursively. Let's return to our little
arithmetic language. I think this is worth going into a little more detail, so
let's fully define the language.
$$
\begin{align*}
    t &:= x \mid t_1 \, t_2 \mid (\lambda x:\tau. t) \mid \textbf{S}(t) \mid \textbf{0} \mid \textbf{ifz}(t_1,t_2,t_3) \\
    \tau &:= \texttt{nat} \mid \tau_1 \rightarrow \tau_2
\end{align*}
$$
Okay I lied when I said "fully" because I'm not actually going to give the
typing judgment. It's what you think it would be. Now we can define our
interpretation. For types, we have $[\![\texttt{nat}]\!] = \mathbb{N}$ and
$[\![\tau_1 \rightarrow \tau_2]\!]=[\![\tau_1]\!] \rightarrow [\![\tau_2]\!]$,
that is, the set of functions from $[\![\tau_1]\!]$ to $[\![\tau_2]\!]$. For
contexts, we have $[\![x_1:\tau_1,\dots,x_n:\tau_n]\!] =
[\![\tau_1]\!]\times\cdots\times[\![\tau_n]\!]$. Finally, for terms, we have
$$
\begin{align*}
[\![t_1 \, t_2]\!] &= [\![t_1]\!] ([\![t_2]\!]) \\
[\![\textbf{0}]\!] &= 0 \\
[\![\textbf{S}(t)]\!] &= [\![t]\!] + 1 \\
[\![\textbf{ifz}(t_1,t_2,t_3)]\!] &= \begin{cases}
    [\![t_2]\!] & \text{if }[\![t_1]\!] = 0 \\
    [\![t_3]\!] & \text{otherwise}
\end{cases}
\end{align*}
$$
Finally, we can define the mapping for lambda abstractions:
$$
[\![\lambda x: \tau. t]\!] = f
$$
where
$$
f(t') = [\![t[t'/x]]\!].
$$
In other words, we first substitute $t'$ for $x$ and then find the
denotation of this new term. Notice that there's no defined denotation for
variables, because in well-scoped terms we should never have to find the
denotation of a variable. Things get more complicated when we attempt to define
recursive functions, but that is an interesting enough topic for its own blog
post.

## Categorical

TODO: this should be its own post. Some day.
