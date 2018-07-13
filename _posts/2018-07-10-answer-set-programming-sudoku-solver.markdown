---
layout: post
title:  "Answer set programming: Sudoku solver"
date:   2018-07-10 18:00:00 +0100
categories: asp
description: Creating a generic sudoku solver in under 10 lines in answer set programming.
---

- Part 1: [Answer set programming: The basics][part1]
- Part 2: Answer set programming: Sudoku solver (You are here)

Answer set programming is a prolog-like form of declarative programming that is primarily used to solve NP-hard search problems. Lets see how easy it is to create a generic sudoku solver that can solve any sudoku you throw at it:

```prolog
x(1..9).
y(1..9).
n(1..9).

{sudoku(X,Y,N): n(N)}=1 :- x(X) ,y(Y).

subgrid(X,Y,A,B) :- x(X), x(A), y(Y), y(B),(X-1)/3 == (A-1)/3, (Y-1)/3 == (B-1)/3.

:- sudoku(X,Y,N), sudoku(A,Y,N), X!=A.
:- sudoku(X,Y,N), sudoku(X,B,N), Y!=B.
:- sudoku(X,Y,V), sudoku(A,B,V), subgrid(X,Y,A,B), X != A, Y != B.

#show sudoku/3.
```

The first three lines just define that our sudoku board is 9x9 fields and we use the numbers from 1 to 9 as the fields contents. The next line starting with `{sudoku(X,Y,N): ...` defines that each field of our board has to contain exactly 1 number from 1 to 9 (our n). The next line defines a little helper that tells us which fields are in the same subgrid. In this case we choose a 3x3 subgrid. If the helper is set for two fields, then they're in the same subgrid, if it is not set they're not. The next 3 lines are constraints. The first one eliminates all possibilities where a number isn't unique in it's line and the second one for the column. The third rule eliminates possibilities where a number isn't unique in its subgrid. The last line just outputs all resulting sudoku fields.

We didn't even try to have the least amount of lines in this one. We could easily sacrifice readability to condense it even more.

The way this program would be executed is by having another file that contains all given numbers in this format: `sudoku(X,Y,N).` and supplying both as an argument to an answer set programming solver like clingo (learn more about clingo: [Answer set programming: The basics][part1]):

`clingo given.lp sudoku.lp`

Lets do an example run. Our `given.lp` looks like this:

```prolog
sudoku(2,3,8). sudoku(3,3,9). sudoku(5,4,1). sudoku(6,4,2). sudoku(4,4,3). sudoku(1,4,4). sudoku(8,4,5). sudoku(7,4,6). sudoku(3,4,7). sudoku(9,4,8). sudoku(2,4,9). sudoku(1,5,1). sudoku(2,5,2). sudoku(3,5,3). sudoku(7,5,4). 
```

The program output will then be:

```
Answer: 1
sudoku(2,3,8) sudoku(3,3,9) sudoku(5,4,1) sudoku(6,4,2) sudoku(4,4,3) sudoku(1,4,4) sudoku(8,4,5) sudoku(7,4,6) sudoku(3,4,7) sudoku(9,4,8) sudoku(2,4,9) sudoku(1,5,1) sudoku(2,5,2) sudoku(3,5,3) sudoku(7,5,4) sudoku(2,1,1) sudoku(7,1,2) sudoku(8,1,3) sudoku(9,1,4) sudoku(4,1,5) sudoku(3,1,6) sudoku(1,1,7) sudoku(5,1,8) sudoku(6,1,9) sudoku(9,2,1) sudoku(5,2,2) sudoku(1,2,3) sudoku(2,2,4) sudoku(3,2,5) sudoku(4,2,6) sudoku(6,2,7) sudoku(8,2,8) sudoku(7,2,9) sudoku(4,3,1) sudoku(1,3,2) sudoku(6,3,3) sudoku(5,3,4) sudoku(9,3,5) sudoku(8,3,6) sudoku(7,3,7) sudoku(6,5,5) sudoku(5,5,6) sudoku(9,5,7) sudoku(4,5,8) sudoku(8,5,9) sudoku(7,6,1) sudoku(8,6,2) sudoku(9,6,3) sudoku(6,6,4) sudoku(1,6,5) sudoku(2,6,6) sudoku(4,6,7) sudoku(3,6,8) sudoku(5,6,9) sudoku(6,7,1) sudoku(3,7,2) sudoku(2,7,3) sudoku(4,7,4) sudoku(5,7,5) sudoku(9,7,6) sudoku(8,7,7) sudoku(7,7,8) sudoku(1,7,9) sudoku(8,8,1) sudoku(9,8,2) sudoku(5,8,3) sudoku(3,8,4) sudoku(7,8,5) sudoku(6,8,6) sudoku(2,8,7) sudoku(1,8,8) sudoku(4,8,9) sudoku(3,9,1) sudoku(4,9,2) sudoku(7,9,3) sudoku(8,9,4) sudoku(2,9,5) sudoku(1,9,6) sudoku(5,9,7) sudoku(6,9,8) sudoku(9,9,9)
SATISFIABLE

Models       : 1+
Time         : 0.093s (Solving: 0.00s 1st Model: 0.00s Unsat: 0.00s)
```

As we can see the first fields it lists are the ones we have given it and after that there are all other fields that make this a valid sudoku. Also note that `Models: 1+` means there are different solutions possible for our given fields (you can list all of them by using `clingo 0 given.lp sudoku.lp`). The whole program ran for 0.093 seconds, so it found a solution pretty fast.


[part1]: https://ddmler.github.io/asp/2018/07/06/answer-set-programming-the-basics.html
