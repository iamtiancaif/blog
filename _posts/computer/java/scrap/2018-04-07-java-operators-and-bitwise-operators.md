---
layout: post
title:  "java operators and bitwise operators"
categories: computer
tags:  computer java copy 
author: web
source: https://docs.oracle.com/javase/tutorial/java/nutsandbolts/operators.html
source: https://www.tutorialspoint.com/java/java_basic_operators.htm
---

* content
{:toc}


Operators
=================


As we explore the operators of the Java programming language, it may be helpful for you to know ahead of time which operators have the highest precedence. The operators in the following table are listed according to precedence order. The closer to the top of the table an operator appears, the higher its precedence. Operators with higher precedence are evaluated before operators with relatively lower precedence. Operators on the same line have equal precedence. When operators of equal precedence appear in the same expression, a rule must govern which is evaluated first. All binary operators except for the assignment operators are evaluated from left to right; assignment operators are evaluated right to left.


precedence
-----------------

|Operators|Precedence|
|--- |--- |
|postfix|expr++ expr--|
|unary|++expr --expr +expr -expr ~ !|
|multiplicative|* / %|
|additive|+ -|
|shift|<< >> >>>|
|relational|< > <= >= instanceof|
|equality|== !=|
|bitwise AND|&|
|bitwise exclusive OR|^|
|bitwise inclusive OR|||
|logical AND|&&|
|logical OR||||
|ternary|? :|
|assignment|= += -= *= /= %= &= ^= |= <<= >>= >>>=|



The Bitwise Operators
---------------------

Java defines several bitwise operators, which can be applied to the integer types, long, int, short, char, and byte.

Bitwise operator works on bits and performs bit-by-bit operation. Assume if a = 60 and b = 13; now in binary format they will be as follows −

a = 0011 1100  
b = 0000 1101

\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-

a&b = 0000 1100  
a|b = 0011 1101  
a^b = 0011 0001  
~a  = 1100 0011  
The following table lists the bitwise operators −
<!--more-->

Assume integer variable A holds 60 and variable B holds 13 then −

[Show Examples](https://www.tutorialspoint.com/java/java_bitwise_operators_examples.htm)

|Operator|Description|Example|
|--- |--- |--- |
|& (bitwise and)|Binary AND Operator copies a bit to the result if it exists in both operands.|(A & B) will give 12 which is 0000 1100|
|| (bitwise or)|Binary OR Operator copies a bit if it exists in either operand.|(A | B) will give 61 which is 0011 1101|
|^ (bitwise XOR)|Binary XOR Operator copies the bit if it is set in one operand but not both.|(A ^ B) will give 49 which is 0011 0001|
|~ (bitwise compliment)|Binary Ones Complement Operator is unary and has the effect of 'flipping' bits.|(~A ) will give -61 which is 1100 0011 in 2's complement form due to a signed binary number.|
|<< (left shift)|Binary Left Shift Operator. The left operands value is moved left by the number of bits specified by the right operand.|A << 2 will give 240 which is 1111 0000|
|>> (right shift)|Binary Right Shift Operator. The left operands value is moved right by the number of bits specified by the right operand.|A >> 2 will give 15 which is 1111|
|>>> (zero fill right shift)|Shift right zero fill operator. The left operands value is moved right by the number of bits specified by the right operand and shifted values are filled up with zeros.|A >>>2 will give 15 which is 0000 1111|





