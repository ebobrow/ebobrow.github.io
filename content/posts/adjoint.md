---
title: "Products, Exponentials, and Adjoint Functors"
date: 2026-04-25T12:40:59-07:00
slug: 2026-04-25-adjoint
type: posts
draft: false
tags:
  - Type Theory
  - Category Theory
---

In category theory, you often think something is a certain type of thing but
actually it turns out to also be a different type of thing. In a first
introduction, it appears that functors are somehow one level higher than
morphisms, because functors map morphisms to morphisms. Similarly, natural
transformations should be one level higher than functors because they map
functors to functors. This is true if you want to be closed-minded and sad. But
it ignores a beautiful thing about category theory: it is so abstract and
general that it can do category theory about itself.

A functor is only a morphism in the category of categories, and a natural
transformation is only a morphism in a functor category. We can just keep going
around and around, and nothing means anything anymore. We can also go in the
opposite direction! If morphisms connect objects, are they one level higher than
objects? Not at all! Certain categories contain exponentials, which are objects
that somehow encapsulate the hom-set between two other objects.

## Exponentials

To gain a little more intuition for exponentials, consider the category
$\textsf{Pos}$ of posets. For posets $A$ and $B$, the set $\textsf{Hom}(A,B)$ is
the set of all monotone functions between them. Interestingly, we can define a
partial ordering on these functions by comparing them pointwise. So
$\textsf{Hom}(A,B)$ is a poset, which means it must be an object in
$\textsf{Hom}$. This is an exponential object, which we denote $B^A$.

A brief aside on notation: I think it is very confusing. It helps me to think
about the category $\textsf{Set}$, where $B^A$ denotes the set of functions from
$A$ to $B$. The cardinality of this set is given by $|B^A|=|B|^{|A|}$ because
each function picks an element of $B$ for each element of $A$. I think that's
why this notation is used, but that doesn't mean I like it.

### Direct Definition

Before we talk about adjunctions, let's look at a slightly more straightforward
definition of an exponential. We define an exponential as $(B^A, \epsilon)$
where $B^A$ is the exponential object and $\epsilon : B^A \times A \rightarrow
B$ is called the evaluation morphism. The requirement is that, for every $f : C
\times A \rightarrow B$, there exists a unique morphism $\stackrel{\sim}{f} : C
\rightarrow B^A$ such that the following commutes:
{{< cd src="https://q.uiver.app/#q=WzAsMyxbMCwwLCJCXkEgXFx0aW1lcyBBIixbMCwwLDEwMCwxXV0sWzAsMiwiQyBcXHRpbWVzIEEiLFswLDAsMTAwLDFdXSxbMiwwLCJCIixbMCwwLDEwMCwxXV0sWzEsMCwiXFxzdGFja3JlbHtcXHNpbX17Zn1cXHRpbWVzIDFfQSIsMCx7ImNvbG91ciI6WzAsMCwxMDBdfSxbMCwwLDEwMCwxXV0sWzAsMiwiXFxlcHNpbG9uIiwwLHsiY29sb3VyIjpbMCwwLDEwMF19LFswLDAsMTAwLDFdXSxbMSwyLCJmIiwyLHsiY29sb3VyIjpbMCwwLDEwMF19LFswLDAsMTAwLDFdXV0=&embed" >}}
The main thing that I think is confusing about this definition is the sudden
appearance of $C$. This is a random third element that has nothing to do with
the exponential of interest. The reason that we need it is because we want to
deal with morphisms. If we turn a morphism $g:A\rightarrow B$ into an object, it
is no longer a morphism. But by including $C$, we are able to retain the origin
point of a morphism. With that in mind, we can think of $f$ as a two-argument
function and $\stackrel{\sim}{f}$ in a vague sense as the curried version of
that function. Then, all the diagram is saying is that we can always partially
apply $f$ to its argument in $C$, bringing us to the exponential object, and
then use our evaluation function to apply it to its argument in $A$. This is the
same as just applying it to both arguments.

### Adjoint Functors

We can also think of exponentiation as the right adjoint to the cartesian
product: for some fixed object $A$, $(- \times A) \dashv (-)^A$. This means that
they kind of basically undo each other. Because they are adjoint functors, they
must have a counit $\epsilon$, which is a natural transformation from $(-)^A
\times A$ to the identity functor. In other words, given any object $B$, we can
find a morphism $B^A \times A \rightarrow B$. But this is exactly the evaluation
morphism at $B$.

## Dependent Sum and Product Types

This raises an interesting question. In dependent type theory, function types
$B^A$ get lifted to product types $\Pi_{x\in A}B(x)$ and pairs $A \times B$ get
lifted to sum types $\sum_{x \in A}B(x)$. When thinking of these as
propositions, we can think of them as universal and existential quantification,
respectively. These are duals to each other in logic. Is this duality an
extension of the adjunction thing?

Well, kind of. We get the relationship $\sum \dashv \iota^* \dashv \prod$.
The reason we don't get a direct adjunct relationship here is because in
dependent type world, these functors bind variables. In other words, their
source category is not the same as their target category, so they can't be
considered as opposites.

The new functor $\iota^*$ is a weakening functor. Given a substitution $\iota :
(\Gamma,x:A) \rightarrow \Gamma$, we get an induced map $\iota^* :
\mathsf{Type}(\Gamma) \rightarrow \mathsf{Type}(\Gamma,x:A)$. We can think of
this as binding another term in the context. Therefore, the adjoints have type
$$
\Pi,\Sigma : \mathsf{Type}(\Gamma,x:A) \rightarrow \mathsf{Type}(\Gamma)
$$
where the input is the type $B(x)$ from above, since it includes an additional
binding for $x$.

<!-- There's a middle man now, in the form of the base change functor. -->
<!-- That is, dependent sums are the left adjoint of the base change functor and -->
<!-- dependent products are the right adjoint. So first, what is a base change -->
<!-- functor? -->

<!-- ### Base Change Functor -->

<!-- To talk about a base change functor, we need to understand what bases we're -->
<!-- changing between. Sometimes, it's helpful to think about a category focalized -->
<!-- over a single element. For a category $C$ and object $A \in C$, the slice -->
<!-- category $C/A$ is the category whose objects are morphisms in $C$ whose target -->
<!-- are $A$, and whose morphisms are composable maps between these objects. For -->
<!-- example, the following commuting diagram shows two objects $f$ and $f'$ with a -->
<!-- morphism $g : f \rightarrow f'$. -->
<!-- {{< cd src="https://q.uiver.app/#q=WzAsMyxbMSwxLCJBIixbMCwwLDEwMCwxXV0sWzAsMCwiWCIsWzAsMCwxMDAsMV1dLFsyLDAsIlgnIixbMCwwLDEwMCwxXV0sWzEsMCwiZiIsMix7ImNvbG91ciI6WzAsMCwxMDBdfSxbMCwwLDEwMCwxXV0sWzIsMCwiZiciLDAseyJjb2xvdXIiOlswLDAsMTAwXX0sWzAsMCwxMDAsMV1dLFsxLDIsImciLDAseyJjb2xvdXIiOlswLDAsMTAwXX0sWzAsMCwxMDAsMV1dXQ==&embed" >}} -->
<!-- This is called a slice category because we can think about slicing up $X$ and -->
<!-- $X'$ over $A$. If $C$ were the category $\mathsf{Set}$, these slices would be -->
<!-- the fibers of $f$ and $f'$ over each element of $A$. -->

<!-- Now, given two slice categories $C/A$ and $C/B$, along with a morphism $f : A -->
<!-- \rightarrow B$, we define the base change functor $f^* : C/B \rightarrow C/A$. -->
<!-- This gives us the pullback -->
<!-- {{< cd src="https://q.uiver.app/#q=WzAsNCxbMCwwLCJmXipFIixbMCwwLDEwMCwxXV0sWzEsMCwiRSIsWzAsMCwxMDAsMV1dLFswLDEsIkEiLFswLDAsMTAwLDFdXSxbMSwxLCJCIixbMCwwLDEwMCwxXV0sWzAsMiwiZl4qcCIsMCx7ImNvbG91ciI6WzAsMCwxMDBdfSxbMCwwLDEwMCwxXV0sWzEsMywicCIsMCx7ImNvbG91ciI6WzAsMCwxMDBdfSxbMCwwLDEwMCwxXV0sWzIsMywiZiIsMSx7ImNvbG91ciI6WzAsMCwxMDBdfSxbMCwwLDEwMCwxXV0sWzAsMSwiZyIsMSx7ImNvbG91ciI6WzAsMCwxMDBdfSxbMCwwLDEwMCwxXV0sWzAsMywiIiwxLHsiY29sb3VyIjpbMCwwLDEwMF0sInN0eWxlIjp7Im5hbWUiOiJjb3JuZXIifX1dXQ==&embed" >}} -->
<!-- We won't go into all the details, but basically $f^*E$ should capture all fibers -->
<!-- over $A$ that correspond to fibers over $B$. -->

<!-- ### Dependent Sums -->

So this adjoint-ness is kind of different. It makes sense, because the
relationship between the two is clearly different. They contain more information
in the form of their $B(x)$ types, so it no longer makes sense to think about
them "undoing each other." They are just duals, meaning that they interact with
binding in similar but opposite ways.

## Sources

1. [Steve Awodey's Notes on Type Theory](https://awodey.github.io/typetheory/notes/typetheory.pdf)
2. [Dependent sum/product and the base-change functor adjunctions on MathOverflow](https://mathoverflow.net/questions/446516/dependent-sum-product-and-the-base-change-functor-adjunctions)
3. [Andrej Bauer's Notes on Realizability](https://github.com/andrejbauer/notes-on-realizability/tree/master)
4. The Dao of Functional Programming by Bartosz Milewski
