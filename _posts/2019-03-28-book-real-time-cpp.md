---
layout: post
title:  "Book review: Real-Time C++"
categories: book-review c++ embedded
---

Recently I picked up a copy of [Real-Time C++](https://www.springer.com/la/book/9783662567173) by Christopher Kormanyos to determine its  suitability for teaching a few new colleagues embedded software development with C++. This post is my review of the book.

<!--more-->
---

I selected [Real-Time C++](https://www.springer.com/la/book/9783662567173) for three reasons:
* I was curious if this book could serve as an introduction to my colleagues new to embedded software development.
* I was curious what the recent C++ standards (C++14/17) bring to the table.
* I wanted to see how the author tackles issues such as memory management and task scheduling in a real-time system. Having experience with such solutions myself, I always find it interesting to see alternative solutions.

# The content

To start off, for a book titled "Real-Time C++", missing any form of definition or discussion on what real-time entails left me with a bad impression. What kind of considerations should be taken into account? How would those considerations translate into code (requirements)? Unfortunately the author simplifies this point by disabling exceptions and running code on a microcontroller.

The book starts of with the basics of C++ and early on introduces both object oriented programming (OOP) and generic programming; showing how the latter can have a positive impact on code and RAM size. I appreciate these kind of examples, as I tend to see people shy away of the  "template voodoo" when they only learned to program in a object oriented (OO) fashion. Speaking of OO, when used as an example to demonstrate an application, the author clearly states that the OO design to toggle a LED was not an efficient nor realistic example (Section 2.6). Yet, this kind of design keeps popping up throughout the book. Up to the point where a complete class hierarchy of 6+ classes is created to control single a RGB LED (Section 9.8). What started so well now feels like a side step in terms of good design practices. Since this is a beginners book, it could have done itself a favour by showing alternative ways of designing components, accompanied with advantages and limitations.

A key example of the book being to generic is shown in Chapter 6, which focusses on code optimization. The author explains how to extract the intermediate generated assembly code from the compiler and how to generate a map file from the linker. But the question of how to actually use these instruments (interpretation of the data) is left to the imagination. Instead the reader is assured these skills will develop over time, I am still unsure how though.
Another example is the detailed description of basic optimizations such as loop unrolling and replacing multiplications with shifts operations. While interesting to know, I doubt its usefulness as modern day compilers are well equipped to perform such optimizations.

Such generic descriptions continue throughout the book. The author does bring up some interesting optimization techniques such as a way to place data in ROM or to align firmware with hardware development to achieve proper hardware/software integration. He also introduces the reader to some parts of 'Modern C++', including the use of `const`, `constexpr` and lambdas. However don't be surprised if the next example is another OO design which misses key virtual function declarations with no `override` keyword in sight.

# The verdict

In conclusion: the content and the lack of detail disappointed me. Overall I believe the book tries to be too generic, thereby lacking the required depth to give the book enough substance. It would have done itself a favour by picking one demo applications and fully going through the steps to design, implement, test and optimize the software.
