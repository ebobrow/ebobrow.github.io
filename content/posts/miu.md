+++
date = '2025-05-21T08:20:28-07:00'
draft = false
title = 'MU Puzzle in Automated Theorem Provers'
tags = ["PLs", "ATPs"]
+++

I recently began re-reading *Gödel, Escher, Bach* and was intrigued by the MU
puzzle, which is Hofstadter's introduction to formal systems. The puzzle is
this: the MIU language consists of the "axiom" _MI_ as well as all strings
constructed according to the following four rules.

1. If you have the string _xI_, then you can construct the string _xIU_ (where
   _x_ is any string of any length).
2. If you have _Mx_, then you can construct _Mxx_.
3. If _III_ appears anywhere in a string, you can replace it with _U_.
4. If _UU_ appears anywhere in a string, you can remove it.

Can you ever construct the string _MI_?

Hofstadter's point is that the puzzle cannot be solved from "within" the system
-- you can't exhaustively construct every string to determine whether you can
get _MI_. However, it is not too hard to solve from "outside" the system. Note
that there are two rules that can alter the number of *I*s in a string. Rule 2
doubles the number and rule 3 decreases it by three. Since we start with _MI_,
which has one _I_, we can never have a string whose number of *I*s is divisible
by three. Doubling a number that is not divisible by three produces a number
that is not divisible by three, and subtracting three from a number that is not
divisible three produces another number that is not divisible by three. Since
_MU_ contains 0 *I*s, a number which is divisible by 3, is cannot be an MIU
string.

## Coq

I was curious, though, how far you could get by working within the system. I
started a naive formulation of the puzzle in Coq[^1]. First, I defined an MIU
string[^2] as a list of elements of the set MIU.
```coq
Require Import Coq.Lists.List.
Import ListNotations.

Inductive MIU : Set := M | I | U.

Definition MIUString : Set := list MIU.
```
Then I defined an inductive predicate to determine whether an MIU string is
an element of the MIU language. To avoid confusion, from now on I say a string
is "valid" rather than a member of the language.
```coq
Inductive validMIUString : MIUString -> Prop :=
  | Ax : validMIUString [M; I]

  | R1 : forall x, validMIUString (x ++ [I]) ->
                   validMIUString (x ++ [I; U])

  | R2 : forall x, validMIUString ([M] ++ x) ->
                   validMIUString ([M] ++ x ++ x)

  | R3 : forall h t, validMIUString (h ++ [I; I; I] ++ t) ->
                     validMIUString (h ++ [U] ++ t)

  | R4 : forall h t, validMIUString (h ++ [U; U] ++ t) ->
                     validMIUString (h ++ t).

Hint Constructors validMIUString : core.
```
Now, the moment of truth. Can Coq's `eauto` tactic determine whether simple
strings are valid? I started with what I thought would be a very easy lemma:
```coq
Lemma simple : validMIUString [M; I; U].
```
Coq could not do this on its own. Let's help it along a little. Here is the
fully manual proof of `simple`.
```coq
Proof. apply R1 with (x := [M]). apply Ax. Qed.
```
The issue comes when we try to get Coq to guess the variable _x_. If we instead
try to call `eapply R1`, we get the error
```
Unable to unify "validMIUString (?M742 ++ [I; U])" with
 "validMIUString [M; I; U]".
```
This seems like it should be doable, but I guess lists are hard for the
unification algorithm. You could probably write an Ltac script to do a
breadth-first search like this, and maybe I will some other time, but that's not
really my main focus right now.

## Isabelle

I next tried to do the same thing in Isabelle. My definitions were about the
same.
```
datatype miu = M | I | U

inductive valid_miu_string :: "miu list ⇒ bool" where
ax: "valid_miu_string [M, I]" |
r1: "valid_miu_string (x @ [I]) ⟹ 
     valid_miu_string (x @ [I, U])" |
r2: "valid_miu_string (M # x) ⟹ 
     valid_miu_string (M # x @ x)" |
r3: "valid_miu_string (h @ [I, I, I] @ t) ⟹ 
     valid_miu_string (h @ U # t)" |
r4: "valid_miu_string (h @ [U, U] @ t) ⟹ 
     valid_miu_string (h @ t)"
```
Isabelle can do a better job of proving that strings _are_ valid. Using
Sledgehammer, I found a tactic that seems to work every time.
```
lemma "valid_miu_string [M, U, I, I, U]"
  by (metis append_Cons append_Nil valid_miu_string.simps)
```
However, it is just as clueless in proving when things are not valid strings.
We can prove a theorem like
```
lemma starts_with_M: "valid_miu_string x ⟹ hd x = M"
proof (induction rule: valid_miu_string.induct)
  case ax
  then show ?case by simp
next
  case (r1 x)
  then show ?case
    by (metis append_Nil hd_append2 list.sel(1))
next
  case (r2 x)
  then show ?case by simp
next
  case (r3 h t)
  then show ?case
    by (metis hd_append list.discI list.sel(1) miu.distinct(2))
next
  case (r4 h t)
  then show ?case
    by (metis hd_append list.sel(1) miu.distinct(4))
qed
```
which helps us prove when things are not valid strings. This is akin to
"stepping outside the system." But we, the humans, are still responsible for
having that kind of insight.

## Conclusion

So what did we learn? Not a lot. Welcome to my blog.

[^1]: I know it's supposed to be Roqc now but I FINALLY got to a point where I
    could be mature about the name Coq and I don't want to give that up.

[^2]: Word to the wise: if you're curious whether Coq supports strings natively,
    I would not recommend looking up "Coq string."
