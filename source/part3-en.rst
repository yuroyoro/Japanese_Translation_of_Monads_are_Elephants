Monads are Elephants Part 3
========================================================================

In this series I've presented an alternative view on the old parable about the blind men and the elephant. In this view, listening to the limited explanations from each of the blind men eventually leads to a pretty good understanding of elephants.

So far we've been looking at the outside of monads in Scala. That's taken us a great distance, but it's time to look inside. What really makes an elephant an elephant is its DNA. Monads have a common DNA of their own in the form of the monadic laws.

This article is a lot to digest all at once. It probably makes sense to read it in chunks. It can also be useful to re-read by substituting a monad you already understand (like List) into the laws.

Equality for All
------------------------------------------------------------------------

Before I continue, I have to semi-formally explain what I mean when I use triple equals in these laws as in "f(x) ≡ g(x)." What I mean is what a mathematician might mean by "=" equality. I'm just avoiding single "=" to prevent confusion with assignment.

So I'm saying the expression on the left is "the same" as the expression on the right. That just leads to a question of what I mean by "the same."

First, I'm not talking about reference identity (Scala's eq method). Reference identity would satisfy my definition, but it's too strong a requirement. Second, I don't necessarily mean == equality either unless it happens to be implemented just right.

What I do mean by "the same" is that two objects are indistinguishable without directly or indirectly using primitive reference equality, reference based hash code, or isInstanceOf.

In particular it's possible for the expression on the left to lead to an object with some subtle internal differences from the object on the right and still be "the same." For example, one object might use an extra layer of indirection to achieve the same results as the one on the right. The important part is that, from the outside, both objects must behave the same.

One more note on "the same." All the laws I present implicitly assume that there are no side effects. I'll have more to say about side effects at the end of the article.

------------------------------------------------------------------------

Inevitably somebody will wonder "what happens if I break law x?" The complete answer depends on what laws are broken and how, but I want to approach it holistically first. Here's a reminder of some laws from another branch of mathematics. If a, b, and c are rational numbers then multiplication (*) obeys the following laws:

Certainly it would be easy to create a class called "RationalNumber" and implement a * operator. But if it didn't follow these laws the result would be confusing to say the least. Users of your class would try to plug it into formulas and would get the wrong answers. Frankly, it would be hard to break these laws and still end up with anything that even looks like multiplication of rational numbers.

Enough intro. To explain the monad laws, I'll start with another weird word: functor.

WTF - What The Functor?
------------------------------------------------------------------------

Usually articles that start with words like "monad" and "functor" quickly devolve into soup of Greek letters. That's because both are abstract concepts in a branch of mathematics called category theory and explaining them completely is a mathematical exercise. Fortunately, my task isn't to explain them completely but just to cover them in Scala.
