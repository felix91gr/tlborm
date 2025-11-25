# Turing Completeness

Rust's Declarative Macros are Turing-Complete in their expansion. Here we will prove this fact.

## Tag Systems

A [Tag System](https://en.wikipedia.org/wiki/Tag_system) is a model of computation that we will use in our proof.

### Definition

> **Note:** the definition below is taken verbatim from Wikipedia.

Briefly, a Tag System is a triplet \\( (m, A, P) \\) where:
\\(  \\)
* \\( m \\) is a positive integer, called the deletion number.
* \\( A \\) is a finite alphabet of symbols, one of which can be a special *halting symbol* \\( H \\). All finite (possibly empty) strings on \\( A \\) are called **words**.
* \\( P \\) is a set of production rules, assigning a word \\( P(x) \\) (called a production) to each symbol \\( x \\) in \\( A \\). The production (say \\( P(H) \\)) assigned to the halting symbol is seen below to play no role in computations, but for convenience is taken to be \\( P(H) = H \\).

A **halting word** is a word that either:
* Begins with the halting symbol \\( H \\), 
* or whose length is less than \\( m \\).

A **transformation** \\( t \\) (called the tag operation) is defined on the set of non-halting words, such that if \\( x \\) denotes the leftmost symbol of a word \\( S \\), then \\( t(S) \\) is the result of deleting the leftmost \\( m \\) symbols of \\( S \\) and appending the word \\( P(x) \\) on the right. Thus, the system processes the m-symbol head into a tail of variable length, but the generated tail depends solely on the first symbol of the head.

A **computation** by a tag system is a finite sequence of words produced by iterating the transformation \\( t \\), starting with an initially given word and halting when a halting word is produced.

### Computational Power

When its parameter \\( m \\) is greater than 1, a Tag System is capable of representing an arbitrary Turing Machine. We will use this fact in our proof.

## Proof

The proof consists of the following steps:

1. We will take an **arbitrary** Tag System \\( T \\) of \\( m > 1 \\).
2. We will construct a `macro_rules!` definition `my_tag_system` that simulates this Tag System, such that:
   1. Every halting string in the Tag System halts in our macro.
   2. Every production rule in the Tag System is reproduced exactly in our macro.

This will prove that we can fully simulate any Tag System of \\( m > 1 \\) with a `macro_rules!` definition's expansion. And, since simulating a Tag System inherits its halting properties, our macro's expansion will halt **if and only if the Tag System halts**.

### An Arbitrary Tag System

Let's specify an arbitrary Tag System \\( T = (m, A, P) \\):

* \\( m \in \mathbb{N}, m > 1 \\)
* \\( A \\) is our set of symbols.
  * \\( A = \\{ a, b, c, \ldots \\} \\)
  * \\( A \\) is finite
  * The empty string is considered a string on \\( A \\).
  * \\( A \\) may contain a special character for halting: \\( H \\).
* \\( P \\) is a set of production rules \\( \\{ p_a, p_b, p_c, \ldots \\} \\), which can be equivalently seen as a function from \\( A \\) to the strings on \\( A \\):

\\[ P: A \to A^\star \\]

### Constructing a Macro Definition for our Tag System

We will now construct a `macro_rules!` definition that fully simulates `T`.

#### Declaration

We begin with the declaration. Nothing special here.

```rust,ignore
macro_rules! my_tag_system {
```

#### Halting Input

First we define what inputs will halt. There are two cases.

A string beginning with the special character \\( H \\) will halt.

```rust,ignore
(H $($tail:tt)*) => {};
```

A string of length \\( [0 \ldots (m - 1)] \\) will halt. 

```rust,ignore
() => {};

($ident_1:ident) => {};

($ident_1:ident $ident_2:ident) => {};

/* ... */

($ident_1:ident $ident_2:ident /* ... */ $ident_m_minus_2:ident) => {};

($ident_1:ident $ident_2:ident /* ... */ $ident_m_minus_2:ident $ident_m_minus_1:ident) => {};
```

#### Non-Halting Input

> **Note**: given two strings \\( a, b \\), the concatenation of both is written as \\( a | b \\) 

For each symbol in \\( A \\), we have a production rule. To recap:

A string \\( S = x | head | tail \\) with \\( \texttt{len}(head) = m - 1 \\) will be transformed into \\( t(S) \\) in the following way:

\\[ t(S) \\]

\\[ = t(x | head | tail) \\]

\\[ = tail | P(x) \\]

Thus, we write each of the rules in `P` like this:

```rust,ignore
// P(x)
(x $d_1:tt $d_2:tt /* ... */ $d_m_minus_1:tt $($tail:tt)*) => {
    my_tag_system!($($tail)* p_x)
};
```

Where we replace `p_x` with the expansion of \\( P(x) \\).

#### All Together

Putting it all together, it looks like this:

```rust
macro_rules! my_tag_system {

    // Halting Input
    (H $($tail:tt)*) => {};

    () => {};

    ($ident_1:ident) => {};

    ($ident_1:ident $ident_2:ident) => {};

    /* ... */

    ($ident_1:ident $ident_2:ident /* ... */ $ident_m_minus_2:ident) => {};

    ($ident_1:ident $ident_2:ident /* ... */ $ident_m_minus_2:ident $ident_m_minus_1:ident) => {};

    // Non-Halting Input

    // P(a)
    (a $d_1:tt $d_2:tt /* ... */ $d_m_minus_1:tt $($tail:tt)*) => {
        my_tag_system!($($tail)* p_a)
    };

    // P(b)
    (b $d_1:tt $d_2:tt /* ... */ $d_m_minus_1:tt $($tail:tt)*) => {
        my_tag_system!($($tail)* p_b)
    };

    // ...
}
#
# fn main() {}
```

### Analysis of our Macro Definition

When does our macro's expansion halt?

Well, as we can see from our definition:
* We can represent all strings on \\( A \\).
* Our macro's definition halts with all strings for which \\( T \\) halts.
* Our macro's definition reproduces all production rules present in \\( T \\). No more, and no less.

Therefore, the macro expansion process will quite literally simulate the Tag System we've encoded in our definition.

Our macro's expansion will halt **if and only if \\(  T \\) halts**.

#### Caveat: Recursion Limit

The Rust Compiler has, very reasonably, a `recursion_limit` parameter for macro expansion. This allows the compiler to bail itself out in case a macro expansion is taking an unreasonably long time.

Such a parameter does technically mean that our macros will always halt. But for the purposes of formal analysis, that's just an implementation detail.

> **Note**: it's the same as noting that a computer's RAM is finite, and thus doesn't represent a real Turing Machine.

Besides, this limit can be bypassed by setting the `recursion_limit` attribute to a very large number, for example: `#![recursion_limit = "999999999999999999"]`

### Relation to Turing-Completeness

* A Tag System with \\( m > 1 \\) is Turing-Complete.
  * This means that we can take an arbitrary Turing Machine \\( M \\) and convert it into a Tag System \\( T \\) with \\( m > 1 \\) that halts if and only if \\( M \\) halts.
* We have just proven that we can take an arbitrary Tag System \\( T \\) and convert it into a `macro_rules!` definition `my_tag_system` that halts in its expansion if and only if \\( T \\) halts.

Therefore, we are able to:
1. Take an arbitrary Turing Machine \\( M \\), 
2. Convert it into a Tag System \\( T \\) that halts if and only if \\( M \\) halts, 
3. And then convert it into a `macro_rules!` definition `my_tag_system` that halts in its expansion if and only if \\( T \\) halts.

And since the "if and only if" relation is transitive, we've just proven that `macro_rules!` expansion is Turing-Complete.

## Consequences of Turing-Completeness

The fact that Rust's Declarative Macros are Turing Complete imposes very important consequences on tooling:

* Will a macro always finish expanding? Undecidable.
* By [Rice's Theorem](https://en.wikipedia.org/wiki/Rice%27s_theorem), most properties of a Turing-Complete system are Undecidable.
  * Are two macros equivalent? Who knows! It's actually very likely that this is impossible to tell in general.
  * Is a macro's definition arm redundant? Might be impossible to tell in general.

All of this is to say: as far as the author of this fragment (Félix Fischer) is concerned, it's **To Be Determined** whether or not any of these questions and many more like them are even answerable.
