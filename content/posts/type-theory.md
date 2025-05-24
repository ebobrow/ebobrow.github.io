+++
date = '2025-05-23T19:08:38-07:00'
draft = false
title = 'What even is type theory?'
tags = ['PLs', 'type theory']
+++

The problem with being self-taught in PLs is that I sort of learned everything
at the same time, which means I did not get a good sense of what things fall in
what categories. In this post I seek to determine, once and for all, what is
type theory and what is not. To do that, we're going to back it way up and start
at the very beginning.

## Logic

It's easier to begin to answer this question with what is not type theory.
In other words, some of the things that I know about typing systems are just
true of logic in general.

A formal logic is a set of sentences together with some way of assigning
meaning to those sentences. A common way of assigning meaning to sentences is
through a deductive system: a set of axioms (true sentences) and rules (ways to
derive new theorems from existing true sentences). Another way of assigning
meaning is through a semantics, which provides a mapping from sentences in our
logical system to elements of some domain we already know how to reason about.
In other words, a deductive system allows you to reason about your logic from
within and a semantics allows you to jump out of the system.

### Axiomatic systems

When people talk about axiomatic systems like Peano Arithmetic or
Zermelo-Fraenkel set theory, there is an implicit ambient system. That is,
axioms alone are not enough to define a logic because axioms contain no
information about how to use them.

For example, you can axiomatize classical logic using only two symbols,
constructing the others from there. There are lots of ways to do this. One nice
one is Church's aximoatization from $\{\rightarrow, \bot\}$:
$$
\begin{align}
    &A \rightarrow (B \rightarrow A) \\
    &(A \rightarrow (B \rightarrow C)) \rightarrow ((A \rightarrow B) \rightarrow
    (A \rightarrow C) \\
    &((A \rightarrow \bot) \rightarrow \bot) \rightarrow A
\end{align}
$$
But remember, you have to play really stupid. What does any of this mean? On its
own, nothing. In order to do anything with this, you need at least one rule of
inference. Modus ponens works in this case.
$$
\frac{A, A \rightarrow B}{B}
$$
Now that we have an inference rule, we have a means of deriving new facts from
old ones, which is what logic is all about.

<!-- ### Consistency -->

<!-- A desirable property of a logical system is consistency, meaning that you cannot -->
<!-- prove both a fact and its negation. This would obviously be very bad, especially -->
<!-- if you are trying to construct a system as a foundation of mathematics. -->
<!-- Classical logic is not consistent because of, for example, the liar's paradox. -->
<!-- We define the proposition $P$ where -->
<!-- $$ -->
<!-- P\equiv \neg P, -->
<!-- $$ -->
<!-- so $P$ is true if it is not true. -->

## Type theory

Fine, I will be unoriginal and tell the barber story. Suppose there is a very
small town with only one barber. Suppose also that everyone in this town needs
to be shaved. Suppose also, crucially, that everyone in this town is dense as
bricks and takes everything literally. Great. Now you have a good picture of
academia.

Now, the barber shaves everyone in the town who does not shave themself. The
question is, does the barber shave himeslf? In classical set theory, this is a
paradox. You can prove both that the barber shaves himself and that he does not.
ZFC's solution is to prevent the construction of such a barber. But that
wouldn't make the townspeople very happy. We can instead avoid the paradox with
a notion of types: the barber is a different _type_ of thing than his clients.
As a human with the ability to think critically, that is probably how you
implicitly interpreted the original statement anyways.

So we're back to our original question. What is type theory? It's the study of
the class of logical systems that use types. Confusingly, each of these systems
is also called a type theory. The simply typed lambda calculus is a type theory.
Martin-Löf's system is also a type theory. It's nothing special, it's just the
first one to use dependent types, which makes it more expressive than STLC.

Typically, there are four judgements you can make in a type theory: that
something is a type, that two types are equal, that something has a certain
type, and that two elements of a type are equal. A fair follow-up question may
be, why do I care? Well, you need to be able to say when something is a type
because logicians are dense as bricks and overly pedantic. It's also clearly
helpful to be able to claim when an object inhabits a type because, under
propositions-as-types, this is the same as saying that the object is a proof of
the proposition represented by that type. Why do we care when two elements of a
type are equal? I mean sure, that seems important, but is it really on par with
the judgement that a proposition is true?

Well, from a purely logical standpoint, I don't think so. But what about when we
start thinking of elements of types as computations? Now it suddenly seems a lot
more interesting. Now we are saying that two elements yield the same result when
computed. They may not be the same program, but they behave the same in the ways
that matter to us. This has real-world applications!! It's called program
verification and it's pretty cool.

## So what's the deal with Curry-Howard?

There's this weird doubling that happens with type theory. Suppose you wish to
prove that the proposition $A$ is true. In other logical systems, you make a
judgement that $A$ is true and you prove it using inference rules. In type
theory, you make a judgement that $a\in A$ for some $a$. Now you have two
proofs. Using the inference rules of the type theory, you have proved that $a$
is of type $A$. But also, $a$ itself is a proof term for $A$. And, because
inference rules (introduction and elimination) are tied to syntax, the proof
that $a\in A$ is related to the structure of $a$. Thought of the other way
around, $a$, contains a record of the steps used to prove its type.

You can have other
proof terms for $A$. They may, using the equality judgement discussed above, be
equal to $a$. What if there is a way to see that two proof terms are equal but
one is "simpler" than the other? Then we have derived computation: you can
"compute" a proof term by simplifying it until it reaches its canonical form.

Maybe we say that one proof term is simpler than another if it represents a
simpler proof. We can formalize this using the property of local soundness,
which shows that you can never gain information by applying an elimination rule.
Local soundness is demonstrated by showing that a proof that uses an
introduction rule followed by an elimination rule can always be simplified. For
example, if we know $A$ and $B$ and we wish to prove $A$ (and we're feeling a
little showy), we can write the natural deduction proof
$$
\begin{prooftree}
    \AXC{}
    \UIC{$A$}
    \AXC{}
    \UIC{$B$}
    \RL{$\wedge I$}
    \BIC{$A\wedge B$}
    \RL{$\wedge E$}
    \UIC{$A$}
\end{prooftree}
$$
Of course, we could also have written
$$
\begin{prooftree}
    \AXC{}
    \UIC{$A$}
\end{prooftree}
$$
In a way, the first proof "reduces" to the second. The first proof corresponds
to the proof term $\text{fst}(a,b)$ and the second corresponds to $a$. This is
exactly the reduction we would want to define when defining the semantics for
pairs. In summary: when working in a type theory, you gain access to proof
terms which contain some record of the steps used to derive a theorem. These
proof terms act as computations if you treat proof simplifications as
reductions.

For more details on this view, see
[A Tutorial on the Curry-Howard Correspondence](https://web.archive.org/web/20190405114037/http://purelytheoretical.com/papers/ATCHC.pdf).
