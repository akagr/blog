---
toc: true
layout: post
description:
categories: []
title: Haskell Diary #2 - Algebra of Data Types
---
In every single Haskell guide I've been through, the term **Algebraic Data Types** is mentioned at least once. In most cases, the text simply moved on to how types are defined and used. A small minority actually tries to explain why Haskell's data types are called `Algebraic`. An even smaller minority succeeded in getting the point across. Here's me documenting what I've learned so far about them. Hopefully it'd be of some help for future Haskell learners.

### Some background first

`Bool` is one of the simplest data type in Haskell. Its defined as:

```haskell
data Bool = True | False
```   
So when we say an expression is of type `Bool`, what values can it possibly evaluate to? It'd be either `True` or `False`. Correct? 2 possible values.

Here's a quick home-made definition of a type `Weekday`:

```haskell
data Weekday = Monday    |
               Tuesday   |
               Wednesday |
               Thursday  |
               Friday    |
               Saturday  |
               Sunday
```
Now, if we say that an expression is of type `Weekday`, what are the possible values here. Any of the days we defined would be correct. That gives us 7 possible values whose type would be `Weekday`.

Let's associate (in our minds) types with the number of values possible. So `Bool` is a 2. And `Weekday` is a 7. Similarly, if we check the max and min bounds of `Int`, we can put `Int` to `2 ^ 64` (because that many values are possible with `Int` data type).

This is something to keep in mind going forward. A data type can be associated to a number based on how many values an expression of that type can possibly evaluate to. Also, it is totally legit for a type to have infinite possible values (think `String`).

### Sum
Now we can get to the **Algebra** part. Here's the all-too-familiar definition of `Maybe`.

```haskell
data Maybe a = Just a | Nothing
--   Maybe a =   a    +   1
```
Ignore the comment in the second line for now.

What are the possible values of `Maybe a`? That's an absurd question unless we know what `a ` is.

Let's do it again. If an expression has type, say, `Maybe Bool`, what values can it have? `Just True`, `Just False` and `Nothing`. 3 values in total. That's 2 boolean values plus 1 new `Nothing` value.

If a variable has type `Maybe Weekday`, the possible values are: 

* Just Monday
* Just Tuesday
* Just Wednesday
* Just Thursday
* Just Friday
* Just Saturday
* Just Sunday
* Nothing

So that's 7 `Weekday` values plus 1 `Nothing`. Now check out the comment in the code snippet above again and see if it makes some sense.

The `|` operator in type definitions is acting like `+`. Data constructors which do not take any types (like `Nothing`) act as constants and represent a value in themselves. Let's see one more example of this to cement this insight.

```haskell
data Either a b = Left a | Right b
--   Either a b =    a   +   b
```

The possible values corresponding to type `Either Bool Weekday` are:

* Left True
* Left False
* Right Monday
* Right Tuesday
* Right Wednesday
* Right Thursday
* Right Friday
* Right Saturday
* Right Sunday

That's 2 + 7 = 9 possible values in total.

### Product
Like the sum, data types can also be defined as products of other data types. Here's a quick example using a custom data type.

```haskell
data Tuple a b = Tuple a b
```

Let's apply some concrete type to both `a` and `b` and see what values are possible.

For `Tuple Bool Bool`, we have the following possible values:

* Tuple True False
* Tuple True True
* Tuple False False
* Tuple False True

That's 2 * 2 = 4 values.

For `Tuple Bool Weekday`, the possible values are:

* Tuple True Monday
* Tuple True Tuesday
* Tuple True Wednesday
* Tuple True Thursday
* Tuple True Friday
* Tuple True Saturday
* Tuple True Sunday
* Tuple False Monday
* Tuple False Tuesday
* Tuple False Wednesday
* Tuple False Thursday
* Tuple False Friday
* Tuple False Saturday
* Tuple False Sunday

That's 2 * 7 = 14 values.

While `|` acts like sum, specifying data types as composites is similarly equivalent to multiplication. `Tuple Weekday Weekday` will give 49 possible values. Because `Weekday * Weekday` or `7 * 7`.

### Data types follow algebraic laws

We can check this real quick by seeing how they follow distribution. For a refresher, distributive law is defined as

```
a * (b + c) == (a * b) + (a * c)
```

Let's see if we can prove that this law holds for data types. Left side of equation first. We need a data type which corresponds to `a * (b + c)`. We already know from above that `Either b c` is equivalent to `b + c`. To multiply it with `a`, we put that in a composite data type as follows.

```haskell
data DLeft a b c = DLeft a (Either b c)
--   DLeft a b c =       a * (b + c)
```

Next, we'll construct a data type which corresponds to right side of equation.

```haskell
data DRight a b c = DRLeft a b | DRRight a c
--   DRight a b c =   (a * b)  +  (a * c)
```

All that's left is to see if those two give the same number of possible values. Let's put `a = Bool`, `b = Bool` and `c = Weekday`.

`DLeft Bool Bool Weekday` will have the following values:

* DLeft True (Left True)
* DLeft True (Left False)
* DLeft True (Left Sunday)
* DLeft True (Right Monday)
* DLeft True (Right Tuesday)
* DLeft True (Right Wednesday)
* DLeft True (Right Thursday)
* DLeft True (Right Friday)
* DLeft True (Right Saturday)
* DLeft True (Right Sunday)
* DLeft False (Left True)
* DLeft False (Left False)
* DLeft False (Right Sunday)
* DLeft False (Right Monday)
* DLeft False (Right Tuesday)
* DLeft False (Right Wednesday)
* DLeft False (Right Thursday)
* DLeft False (Right Friday)
* DLeft False (Right Saturday)
* DLeft False (Right Sunday)

Which are be `Bool * (Bool + Weekday)` or `2 * (2 + 7)` = 18 values in total.

Let's see if the `DRight Bool Bool Weekday` offers the same number of values.

* DRLeft False False
* DRLeft False True
* DRLeft True False
* DRLeft True True
* DRRight False Monday
* DRRight False Tuesday
* DRRight False Wednesday
* DRRight False Thursday
* DRRight False Friday
* DRRight False Saturday
* DRRight False Sunday
* DRRight False Monday
* DRRight True Tuesday
* DRRight True Wednesday
* DRRight True Thursday
* DRRight True Friday
* DRRight True Saturday
* DRRight True Sunday

That's `(Bool * Bool) + (Bool * Weekday)` or `(2 * 2) + (2 * 7)` = 18 values.

Hence Proved! I always loved writing this :)

### Exponents

Sum and product aren't the only operations out there. One of the most common data types in Haskell is actually exponential. We know them as `(->)` or more generally as functions.
This is a lot bigger topic but I will quickly list out how to visualise them.

For our example, we'll use a simple function of type `Weekday -> Bool`. How many ways are possible to write a function of that type while being total (not skipping any Weekdays).

* myFunc Monday = True
* myFunc Tuesday = True
* myFunc Wednesday = True
* myFunc Thursday = True
* myFunc Friday = True
* myFunc Saturday = True
* myFunc Sunday = True

Note that above is just one possible definition of such a function, not 7. This function can now take a Weekday (any one of them) and return a Bool. It may have been better to write it in short like `myFunc _ = True`.
We're talking about how many ways a function can be defined, instead of the number of possible values in previous operations.
If we change any of the `True` in above definition to `False`, it's a whole new function. What if we change a couple of `True` above to `False`? That's yet another function. Writing all that will make this the longest post ever.
So I'm gonna believe in reader and leave them to write all the possible definitions. In the end, the number of ways a similarly trivial function from `Weekday -> Bool` can be defined is 128 which is `2 ^ 7` or `Bool ^ Weekday`.

To formalise it, a function `a -> b` corresponds to algebraic operation `b ^ a`.

### Conclusion

I wrote this post for a select species of programmers who like to know what they're playing with with usability of that knowledge being non-consequential. I think that in general course of things, all the text above is not super handy and will probably not help someone write that awesome web server or parser. So, sincere apologies if I wasted your time.

For a fuller discussion on the algebra of data types, including recursive types, there's a great series of posts [here](https://chris-taylor.github.io/blog/2013/02/10/the-algebra-of-algebraic-data-types/).

I'll appreciate any and all feedback. Use this [reddit thread](https://www.reddit.com/r/haskell/comments/4e5fxp/haskell_diary_2_algebra_of_data_types/). Thanks for reading!
