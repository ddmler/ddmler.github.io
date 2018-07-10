---
layout: post
title:  "Answer set programming: The basics"
date:   2018-07-06 18:00:00 +0100
categories: asp
description: A prolog-like declarative programming language used for solving NP-hard search problems.
---

Part 1: Answer set programming: The basics (You are here)

Part 2: [Answer set programming: Sudoku solver][part2]

Answer set programming is a declarative form of programming. This means you don't need to code a way to solve the problem, you just have to describe your problem and let a so-called solver handle the rest. This is the same as your SQL query, in which you only describe what you would like to have and the database itself figures out how best to get this data for you. There are multiple solvers out there, one example would be [clingo][clingo]. To make it easy to get started, you can run clingo [in your browser][browser] or [with Docker][docker].

To get started create a new file called `test.lp`:

```prolog
x.
```

Run it with: `clingo test.lp`

which will output:

```
Answer: 1
x
SATISFIABLE
```

As we can see we get 1 answer which is x. This means x = true. What we just now did was we told the solver that we want to know which variables in our program need to be true, so that we achieve our goal (in this case our goal is for x to be true). Another way to think about it is that we simply told the solver that x needs to be true (because it always has to, for our program to be satisfiable). So lets look at some more constructs:

## Conditions
```prolog
x :- y.
```

If y is true, so is x.

## Constraints
```prolog
:- y.
```

As we have seen with conditions if y is true so has to be the left part, which is empty, which is the same as false. This would mean false = true if y is true. The solver gets that this was a mistake and so y has to always be false.

## Predicates
```prolog
sudoku(1..9, 1..9).
```

The first character has to be lowercase. This will generate a complete 9x9 sudoku field, one sudoku predicate for each field. These can be nested for example:

```prolog
% a comment: the same as sudoku(1,1).
sudoku(x(1), y(1)).
```

## Variables
```prolog
x(1..9).
y(1..9).
sudoku(Var1, Var2) :- x(Var1), y(Var2).
```

Variables have to start with an uppercase character. The result of this would again be our whole sudoku board. The conditional works as follows: if x(2) and y(5) is true, then sudoku(2, 5) is also true. The same will be done for all combinations of x and y values here. It is also possible to use your standard programming operators like `+, -, ==` and so on.

## Choice
```prolog
0{ z(1);z(2);z(3) }3.

{sudoku(X, Y, N) : n(N)}=1 :- x(X) ,y(Y).
```

The first line defines a list of 3 items (z(1) to z(3)) and will have 4 choices: nothing, only one of the list, two or all three of the z list. The second line will generate all sudoku fields and choose exactly 1 number from n for each field.

## Sudoku

With these few constructs we have seen now, we can already create a generic sudoku solver in answer set programming in less than 10 lines. You can try it yourself or look at the solution in part 2 (coming soon).

## Incremental steps

A lot of times you find yourself in a situation where you have to use multiple steps. An easy example would be that you want to move a chess piece more than one field from your starting point. After each step the environment changes and you would have to reflect that with some extra variables on each predicate that shows the step number. The inbuilt incmode functionality provides you with this extra time component and automatically tries new steps until it reaches a solution or a given maximum.

```prolog
#include <incmode>.
#const imax   = 30. % max number of steps

#program base.
% everything that doesn't depend on the current step

#program step(t).
% everything that depends on the current step. t is the time (step number) variable
x(1..9, t)

#program check(t).
% provides a query(t) which is true as long as the limit of number of steps is not reached
% we can use it like this if goal(t) indicates if the goal is reached at this step:
:- query(t), not goal(t).
```

This program will execute the complete step as long as the check constraint is not broken (= either the goal or the limit of steps is reached)

## Optimization

```prolog
maxrow(R) :- R=#max{S:countrow(S)}.
```

Sets R to the maximum of S, where S is a list of all numbers that exist for the predicate countrow.

```prolog
#minimize{T : time(T)}.
```

This will choose from all possible solutions the one (or multiple) with the minimum in the given attribute. In this case it would look for the solution which needs the least amount of time.

## What's next?

Try clingo [in your browser][browser] for some examples to play around with. You can also include a python or lua script to write extra code in these languages if you find it hart to express something with declarative programming. Don't forget to check out [part 2 to see a sudoku solver][part2].

[clingo]: https://potassco.org/
[browser]: https://potassco.org/clingo/run/
[docker]: https://github.com/ddmler/docker-clingo
[part2]: https://ddmler.github.io/asp/2018/07/10/answer-set-programming-sudoku-solver.html
