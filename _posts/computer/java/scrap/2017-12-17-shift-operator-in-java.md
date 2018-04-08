---
layout: post
title:  "shift operator in java"
categories: computer
tags:  computer article java copy stackoverflow
author: web
source: https://stackoverflow.com/questions/19058859/what-does-mean-in-java
---

* content
{:toc}


  

* * *
  

The `>>>` operator is the [unsigned right bit-shift operator in Java](http://docs.oracle.com/javase/tutorial/java/nutsandbolts/op3.html). It effectively divides the operand by `2` to the power of the right operand, or just `2` here.

The difference between `>>` and `>>>` would only show up when shifting negative numbers. The `>>` operator shifts a `1` bit into the most significant bit if it was a `1`, and the `>>>` shifts in a `0`regardless.

<!--more-->
**UPDATE:**

Let's average `1` and `2147483647` (`Integer.MAX_VALUE`). We can do the math easily:

    (1 + 2147483647) / 2 = 2147483648 / 2 = 1073741824

Now, with the code `(low + high) / 2`, these are the bits involved:

              1: 00000000 00000000 00000000 00000001
    +2147483647: 01111111 11111111 11111111 11111111
    ================================================
    -2147483648: 10000000 00000000 00000000 00000000  // Overflow
    /2
    ================================================
    -1073741824: 11000000 00000000 00000000 00000000  // Signed divide, same as >> 1.

Let's make the "shift" to `>>>`:

              1: 00000000 00000000 00000000 00000001
    +2147483647: 01111111 11111111 11111111 11111111
    ================================================
    -2147483648: 10000000 00000000 00000000 00000000  // Overflow
    >>> 1
    ================================================
    +1073741824: 01000000 00000000 00000000 00000000  // Unsigned shift right.


Bitwise and Bit Shift Operators
===============================

  

https://docs.oracle.com/javase/tutorial/java/nutsandbolts/op3.html  

* * *

  

The Java programming language also provides operators that perform bitwise and bit shift operations on integral types. The operators discussed in this section are less commonly used. Therefore, their coverage is brief; the intent is to simply make you aware that these operators exist.

The unary bitwise complement operator "`~`" inverts a bit pattern; it can be applied to any of the integral types, making every "0" a "1" and every "1" a "0". For example, a `byte` contains 8 bits; applying this operator to a value whose bit pattern is "00000000" would change its pattern to "11111111".

The signed left shift operator "`<<`" shifts a bit pattern to the left, and the signed right shift operator "`>>`" shifts a bit pattern to the right. The bit pattern is given by the left-hand operand, and the number of positions to shift by the right-hand operand. The unsigned right shift operator "`>>>`" shifts a zero into the leftmost position, while the leftmost position after `">>"` depends on sign extension.

The bitwise `&` operator performs a bitwise AND operation.

The bitwise `^` operator performs a bitwise exclusive OR operation.

The bitwise `|` operator performs a bitwise inclusive OR operation.

The following program, [`BitDemo`](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/examples/BitDemo.java), uses the bitwise AND operator to print the number "2" to standard output.

class BitDemo {
    public static void main(String\[\] args) {
        int bitmask = 0x000F;
        int val = 0x2222;
        // prints "2"
        System.out.println(val & bitmask);
    }
}
  

What do << or >>> in java mean
==============================

  

https://stackoverflow.com/questions/13387365/what-do-or-in-java-mean  

* * *

  

They are Bitwise Bit shift operators, they operate by shifting the number of bits being specified . Here is [tutorial](http://docs.oracle.com/javase/tutorial/java/nutsandbolts/op3.html) on how to use them.

> The signed left shift operator "<<" shifts a bit pattern to the left
> 
> The signed right shift operator ">>" shifts a bit pattern to the right.
> 
> The unsigned right shift operator ">>>" shifts a zero into the leftmost position
