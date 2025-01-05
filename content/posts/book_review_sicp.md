+++
date = '2024-12-30T01:12:46+01:00'
title = '[Book Review]: Structure and Interpretation of Computer Programs'
tags = ['typescript']
+++

![sicp book cover](/sicp-cover.jpeg)

I just finished reading that book, just before the end of the year. I wish I read these 600 pages earlier in my career.

The book consists of 5 parts. It starts with the basic elements of programming: arithmetic, variables, functions... Then from these basic elements, we build towards more real-life problems to solve, using data structure, order of growth, function compositions and so on...
We define models that correspond to the problems we solve, or should I say, a language to express how we interact with the environment, so naturally we end with the 2 last parts having a taste of writing an interpreter and a compiler.

<!-- Knowing a bit about maths and having studied Nand2Tetris definitely helped understand some parts quicker, but honestly, I did not try to go too much in depth towards the end of the book since dealing with registers and so is quite far from what I do (also, I planned to learn about compilers later). -->

Here's some ideas from the book that I liked.

<!-- - mecanism for developing complex ideas (1.1): -->
<!---->
<!--   - primitive expressions -->
<!--   - means of combination -->
<!--   - means of abstraction -->
<!---->
<!-- - first-class means (1.3.4): -->
<!---->
<!--   - can be named by variables -->
<!--   - can be passed as arguments to procedures -->
<!--   - can be returned by procedures -->
<!--   - can be included in data strutures -->
<!---->
<!-- - levels of abstraction -->

## 1. Functional vs Object oriented vs Streams

<!-- (3.5.5) -->

In the book, we spend the first two parts using only functional programming. It is only in the third (after about 200 pages) that we are introduced to assignment which lead to encapsulation of local state.
There is discussion about these two way and their respective challenges.
Here's an example:

```typescript
class Account() {
  constructor(private balance: number) {}

  withdraw(amount: number): number {
    this.balance -= number;
    return this.balance;
  }
}

const acc = new Account(100);
acc.withdraw(20);
// 80
acc.withdraw(20);
// 60
```

In object-oriented, we model things as entities that we can interact with and that can interact with each other. We model the changes of state with assignment.
Successive calls to the same procedure can have different results, which can make a complex system difficult to reason with and to test, especially when multiple local states are correlated.

```typescript
const withdrawSteam = (balance: number, amountStream: number[]): number => {
  return amountStream.reduce((acc, cur) => acc - cur, balance);
};
console.log(withdrawSteam(100, [20, 20]));
// 60
console.log(withdrawSteam(100, [20, 20]));
// 60
```

In functional, we model things like an electrical circuit where each element outputs a computation based on its inputs.
Successive calls will output the same results each time.
In essence, we model time explicitly, however, time-related problems like when two changes to the same account occur (using mutexes), or and priority rules when merging streams need to be addressed.

## 2. Dispatching on type and additivity

<!-- (2.4.3) -->

This is a powerful technique to obtain modularity in the implementation while keeping a clean API.

```typescript
const getRealPart = (value) => {
  if(type === "rectangular") {
    ...
  } else if (type === "polar") {
    ...
  }
};
const getImagPart = (value) => {...};
const getMagnitude = (value) => {...};
const getAngle = (value) => {...};
```

Here, if you need to add another type, you would need to go into every function.

```typescript
import { rectangularOperations } from "somewhere-else";

const polarOperations = {
  getRealPart(value) {...},
  getImagPart(value) {...},
  getMagnitude(value) {...},
  getAngle(value) {...},
};
const operationTable = {
  polar: polarOperations,
  rectangular: rectangularOperations,
};

const applyOnComplexNumber = (
  operation: string,
  complexNumber: { type: string; value: ComplexNumberValue },
): [any, null] | [null, Error] => {
  if (!operationTable[complexNumber.type]) {
    return [null, new Error("type not implemented")];
  }
  if (!operationTable[complexNumber.type].operation) {
    return [
      null,
      new Error(
        `operation {operation} not implemented on type {complexNumber.type}`,
      ),
    ];
  }
  return [operationTable[complexNumber.type](complexNumber.value), null];
};
```

This way, we can easily package these operations by type and add another like a plugin.

## 3. Constraints

<!-- (3.3.5) -->

This pattern can be used when the computation is not unidirectional and you need to propagate constraints, when a value depends on one or more others. This is a common thing in frontend frameworks nowadays, but I was pleasantly surprised to find this in a book originally written in 1985.

```typescript
class Temperature {
  private _val: { c: null | number; f: null | number } = {
    c: null,
    f: null,
  };
  private _cb: Array<(c: null | number, f: null | number) => unknown> = [];

  set c(value: null | number) {
    if (value === null) {
      this._val = {
        c: null,
        f: null,
      };
    } else {
      this._val = {
        c: value,
        f: (9 / 5) * value + 32,
      };
    }
    this.onChange();
  }

  set f(value: null | number) {
    if (value === null) {
      this._val = {
        c: null,
        f: null,
      };
    } else {
      this._val = {
        f: value,
        c: (value - 32) * (5 / 9),
      };
    }
    this.onChange();
  }

  addCallback(f: (c: null | number, f: null | number) => unknown) {
    this._cb.push(f);
  }

  onChange() {
    this._cb.forEach((cb) => cb(this._val.c, this._val.f));
  }
}

const temp = new Temperature();
temp.addCallback((c, f) => {
  console.log(`Celcius temp: ${c ?? "???"}`);
  console.log(`Fahrenheit temp: ${f ?? "???"}`);
});
temp.c = 25;
// "Celcius temp: 25"
// "Fahrenheit temp: 77"
temp.f = 212;
// "Celcius temp: 100"
// "Fahrenheit temp: 212"
temp.f = null;
// "Celcius temp: ???"
// "Fahrenheit temp: ???"
```
