---
layout: post
title:  "Representing money in Python"
categories: python finance
---

I am working on a personal project to create a financial budgeting system. One of the first challenges is how to represent and perform operations on money.

---

A first thought could be to use floating point numbers, however floating point numbers cannot be used to represent exact numbers. For example, say you spend 2,20. Saving this number as floating point will actually save it as 2,19999 or 2.200001 depending on the platform. This quantization error is undesirable for financial systems. I need a specialized type for money representation.

## Requirements

Money consists out of 2 components:
* A specific amount;
* A currency identifier.

For my application basic arithmetic operations (addition, subtraction, multiplication and division) will be required. Each operation should support various data types and the result must be rounded correctly. Additionally I want to be able to divide bills into parts, for example when splitting a bill with other people or when spreading a quarterly bill over each month.

With each of the above the correct currency attributes must be taken into account. Think of details such as the precision of the amount. Having two digits after the decimal point is most common, however there are currencies with three or zero. A money data type should incorporate this knowledge when performing arithmetic operations. The [Babal](http://babel.pocoo.org/en/latest/) package integrates data from the Common Locale Data Repository ([CLDR](http://cldr.unicode.org/)), which provides various localization data such as currency information. Babel can be used to format currency values into string representations or queried for precision.

An extra is the functionality to convert between currencies.

### Requirement list:
* Proper arithmetic and rounding support.
* Python 3 support.
* Babel integration.
* Exchange conversion (optional).

## Finding a suitable data type

In Python the [Decimal](https://docs.python.org/3.6/library/decimal.html) class provides the ability to correctly represent an amount of money and basic arithmetic support. However Decimal misses the possibility to specify a currency and implement the correct handling. Luckily there are a few packages available which wrap the Decimal class:
* [py-money](https://pypi.org/project/py-money/)
  * The required functionality seems to be present with rounding explicitly taken into account.
  * There a only a few issues and pull requests, though the pull requests do not seem be get merged due to inactivity.
  * Additionally there is little activity in terms of commits, forks and starts.
  * Has Babel integration.
  * Test coverage: 99%, some error cases are not being tested.
* [py-moneyd](https://pypi.org/project/py-moneyed/)
  * Required functionality seems present, rounding needs to be performed manually.
  * There is no developer actively working on the library, there do seems to be plans and discussions but nobody to actually implement it.
  * There is some activity though it is mainly focussed on version bumps and issue discussion.
  * No Babel integration.
  * Test coverage: 97%, most error cases are not tested.
* [money](https://pypi.org/project/money/)
  * This repository seems to be abandoned, given the commit history and status of active pull requests.
  * The library does have include functionality to exchange currencies.
  * Has Babel integration

## Current outcome

For now py-money seems to provide the functionality required with proper arithmetic and rounding logic.
