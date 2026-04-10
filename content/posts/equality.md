---
title: "Equality in Type Theory"
date: 2026-04-08T09:54:44-07:00
slug: 2026-04-08-equality
type: posts
draft: false
tags:
  - PLs
  - Type Theory
---

As math gets more advanced, the things you prove get dumber and dumber. In
Calculus, you prove really cool things like "This sum of infinitely many real
numbers equals a finite number!" Then you take Analysis where you have to
explain "What is infinity?" and "What is a real number?" and "What is equals?"

The answer to the last question, as it turns out, is, "Nobody knows."

## Word to the Wise

The secret to PL theory is that we take a collection of very common symbols,
strip them of their meaning entirely, and then build it back from the ground up.
For the rest of this post, to protect your sanity, kindly forget everything you
think you know about $\equiv$ and $=$. We may as well use 🐒 and 🪕.

## Syntactic Equality

The simplest (and strictest) form of equality is syntactic equality. Two
expressions are syntactically equal if they were literally constructed the same
way. Therefore, $a+b$ is syntactically equal to $a+b$ but it is not
syntactically equal to $b+a$. The benefit of syntactic equality is that it is
straightforward to implement and it is definitely decidable. However, it misses
some equalities that you might expect, such as
$$
\lambda a. a \neq_S \lambda b. b
$$
$$
(\lambda a. a) \, x \neq_S x
$$

## Judgmental/Definitional Equality

We can get a little more clever with our equality if we introduce it as a new
type of judgment in our type theory. Just as you can have $\Gamma \vdash
A\textsf{ type}$ for "$A$ is a type" and $\Gamma \vdash a:A$ for "the term $a$
has type $A$," you can also have $\Gamma \vdash A \equiv B$ and $\Gamma \vdash a
\equiv b : A$ for equality of types and terms, respectively. This approach
involves the construction of new rules, like
$$
\frac{\Gamma \vdash A \equiv A' \qquad \Gamma \vdash B \equiv B'}
     {\Gamma \vdash A \rightarrow B \equiv A' \rightarrow B'}
$$
An implementation like this is also referred to as *definitional equality*
because it takes into account the definitions of our system. Definitional
equality essentially allows us to substitute one term anywhere we see another
because by definition, they are equal. A standard set of equality rules will
also take into account things like $\beta$-reduction, so that in this system we
have
$$
\vdash \lambda a. a \equiv \lambda b. b : A \rightarrow A
$$
$$
\vdash (\lambda a. a) \, x \equiv x : A
$$
Judgmental equality is still decidable, but a drawback is that we can only use
it in metatheoretical statements. Inside the type theory, we are totally blind
to the notion of equality. In order to use equality within our system, we need
to get a little wild.

## Typal/Propositional Equality

By the Curry-Howard isomorphism, we should be able to express a type that, when
inhabited, proves that $a=b$. This logical statement involves the use of
quantified variables, so it can only be expressed in predicate logic. Likewise,
the type $a=b$ would have to depend on terms, so it can only be expressed in a
dependent type theory.

From now on, we will deal in Martin-Löf type theory. This is the simply typed
lambda calculus extended with dependent products (analogous to universal
quantification in logic) and dependent sums (analogous to existential
quantification). Product types are inhabited by functions, whose return type may
depend on their input, and sum types are inhabited by pairs, where the first
element can be thought of as the witness and the second element a proof of the
desired property.

In this type theory, we can define identity types. Intuitively, these represent
the smallest reflexive closure---every term equals itself and nothing else.
Formally, we define the type $a=_Ab$ where $a$ and $b$ are terms and $A$ is a
type. Its introduction form is the term $\textsf{refl}_A(a)$, with the rule
$$
\frac{\Gamma \vdash a : A}
     {\Gamma \vdash \textsf{refl}_A(a) : a =_A a}
$$
This might feel a bit silly, but what it allows us to do is to reason about
variables. Because we are defining equality to be the *smallest* reflexive
closure, we know that if we want to do something with a proof that $x$ is equal
to something else, we need only consider the case where $x$ equals itself. We
define an elimination form for the identity type as a sort of structural
induction. The term is called $J$ for reasons beyond my understanding, and the
rule is
$$
\frac{\Gamma,a:A,b:A,p:a=_Ab \vdash C\textsf{ type} \qquad \Gamma,c:A \vdash t :
C[c/a, c/b, \textsf{refl}_A(c)/p]}
     {\Gamma,a:A,b:A,p:a=_Ab \vdash J_A(t,a,b,p) : C}
$$
Oh boy. Let's break this down. In our context, we have terms $a$ and $b$ and a
proof that they are equal. With this proof, we wish to prove something new. The
typing rule for $J$ essentially states that, if we can prove this new thing
about some arbitrary term $c$ using a proof that $c=_Ac$, then it must hold in
every case. Because that is the *only* possible case. We pass a proof term $t$
proving this for the specialized case, and what we get is a proof in the general
case.

As a simple example, suppose we wish to prove that $a=_Ab \rightarrow b=_Aa$.
After some proof work, we end up with $a:A,b:A,a=_Ab$ in our context. Now, we
need to pass the right arguments to $J$ so that its type $C$ is $b=_Aa$. The
left premise is satisfied because $b$ and $a$ are valid terms, so the identity
type is a valid type. Next, we look at the right premise. After the
substitutions, we get the type $c=_Ac$. We need some term $t$ with this type.
But that's easy: it's the $\textsf{refl}$ term. Therefore, we derive that
$$
a:A,b:A,p:a=_Ab \vdash J_A(\textsf{refl}_A(a), a, b, p) : b =_A a
$$
This proves that equality is commutative.

Typal equality is looser than judgmental equality, in the sense that any two
terms which are judgmentally equal must also be typally equal. Suppose $\Gamma
\vdash x \equiv y : A$. Then, the rules of judgmental equality give us that
$$
\begin{prooftree}
    \AXC{}
    \UIC{$\Gamma \vdash x \equiv y : A$}
    \UIC{$\Gamma \vdash (x =_A y) \equiv (x =_A x)$}
    \AXC{}
    \UIC{$\Gamma \vdash \textsf{refl}_A(x) : x =_A x$}
    \BIC{$\Gamma \vdash \textsf{refl}_A(x) : x =_A y$}
\end{prooftree}
$$

Now we see that all judgmentally terms are typally equal, but is the converse
true? It turns out that the answer is no.

## Extensional vs Intensional Systems

We might be content to call it a day here. With identity types, we now have a
way to reason about equality within our type theory. However, this is not the
end of the story. We are still somewhat limited in our expressivity. A common
example is that of functional extensionality: It might make sense to have a rule
that, for some functions $f,g:A \rightarrow B$, as long as
$$
\forall x:A. f(a) = g(a)
$$
then $f=g$. In other words, we don't care about the function definitions
themselves as long as their behavior is the same on every input. For example, to
return to our old addition function, we may want to be able to prove
$$
\vdash (\lambda x. x + 1) =_\textsf{nat} (\lambda x. 1 + x)
$$
This is not possible with our identity types because these are not, strictly
speaking, the same function, nor can they reduce to the same function. One
solution to this problem is to introduce extensional type theory, which is just
our existing theory with an additional rule:
$$
\frac{\Gamma \vdash p : a =_A b}
     {\Gamma \vdash a \equiv b : A}
$$
We've already seen that judgmental equality implies typal equality, so by adding
the reverse direction we've now removed the distinction between judgmental
equality and typal equality.

This gives us some nice properties like functional extensionality.[^1] Suppose
$r : \prod_{x:A} f(x) =_{B(y)} g(x)$. That is, $r$ is a proof that $f$ and $g$
agree on all inputs. This means that
$$
y : A \vdash r(y) : f(y) =_{B(y)} g(y),
$$
and applying our new equality reflection rule, we get
$$
y:A \vdash f(y) \equiv g(y) : B(y).
$$
This allows us to derive $(\lambda y:A. f(y)) \equiv (\lambda y:A. g(y)) : \prod_{x:A} B(x)$
and therefore $f \equiv g : \prod_{x:A} B(x)$. Finally, we can substitute $g$
for $f$ in the type of $\textsf{refl}(f)$ to get
$$
\vdash \textsf{refl}(f) : f = g.
$$
Therefore, if $f$ and $g$ agree on all inputs, we can derive their equality.

The basic idea of functional extensionality is that you can look into the
definition of a function to check for equality. A similar principle applies to
dependent pairs.

So this is great! Now our equality is smart, and it has a lot of nice
properties. But the downside of an extensional type theory is that equality is
no longer decidable. This makes sense: Now that we have functional
extensionality, is $f(x):=0$ equal to
$$
g(x) := \begin{cases}
0 & \text{if the Riemann hypothesis holds} \\
1 & \text{otherwise?}
\end{cases}
$$
We can't know. For this reason, a lot of people avoid extensional type theory in
favor of decidable systems, like Homotopy Type Theory. Someday I will understand
Homotopy Type Theory, but is the me writing this post equal to the me that
understands HoTT? That's decidable: No.

## Sources

1. [nLab extensional type theory](https://ncatlab.org/nlab/show/extensional+type+theory)
2. [Stanford Encyclopedia of Philosophy Intuitionistic Type Theory](https://plato.stanford.edu/entries/type-theory-intuitionistic)
3. [Steve Awodey's Notes on Type Theory](https://awodey.github.io/typetheory/notes/typetheory.pdf)
4. [The HoTT Book](https://homotopytypetheory.org/wp-content/uploads/2013/03/hott-online-323-g28e4374.pdf)
5. *Extensional Constructs in Intensional Type Theory,* Martin Hoffman

[^1]: Proof from [here](https://cstheory.stackexchange.com/questions/46331/extensional-type-theory-and-function-extensionality).
