# Technica 2022 Tech+Research: Course Assistant

This repository contains materials for UMD's [Technica 2022](https://gotechnica.org/) [Tech+Research](https://inclusion.cs.umd.edu/events/techresearch) "course assistant" project, supported by Amazon AWS.
The event was held October 14-16, 2022.

Have you ever wondered how courses get scheduled?
Each course needs to be assigned to a suitably large classroom at a time its instructor finds acceptable.
Unfortunately, not all classrooms are big enough for all courses, and not all instructors' (and students') most-preferred times will be available.
How can we meet our minimum goal (schedule all classes) and maximize our constituents' preferences, even as we have thousands of students, dozens or hundreds of courses, and dozens of rooms?
This is a real problem for UMD CS!

This research project will explore different approaches to solving this problem.
Student researchers will develop solutions to the problem and scientifically compare them both analytically and empirically.
The mentors will teach the participants about cutting-edge automated reasoning technology based on SMT -- "SAT modulo theories" -- solvers, which can be used as the basis for a solution, and which is seeing increasing use in industry, particularly at Amazon.
With this technology you can specify constraints on your solution, and the solver will automatically find it.
But it takes some skill to encode a solution amenable to SMT -- the mentors will help the students develop this skill.

Target Outcomes:  

* Learn about automated reasoning technology and how to use it.
* Try out computer science research!

## Table of Contents

* [Setup](#setup)
* [Background Reading](#background-reading)
  * [Intro to SAT](#intro-to-sat)
  * [Solving Puzzles with SMT](#solving-puzzles-with-smt)
    * [Cats and Dogs](#cats-and-dogs)
    * [Sudoku](#sudoku)
* [Project: An Automated Assistant for Course Management](#project-an-automated-assistant-for-course-management)

## Setup

Clone this repository:

```bash
git clone https://github.com/khieta/technica-2022
```

All materials assume that you have a Python 3 installation available.
You can check your current version  of Python by running `python3 --version`.

You will need to install the Z3 automated theorem prover and its
Python bindings. This should only require:

```bash
pip3 install z3-solver
```

Now to check that everything was correctly installed, open a Python terminal and run:

```python
from z3 import *
```

You can copy and paste the code examples below into this terminal.

## Background Reading

### Intro to SAT

Say that we want to find a *satisfying assignment* for `(a | !b) & (!a | c)`, i.e., values for `a`, `b`, and `c` that make the expression true.
One way to do this is to try all possible values of  `a`, `b`, and `c`.
There are 2^3 = 8 possible choices of `a`, `b`, and `c` -- can you list them?

In general, for *n* variables, there will be 2^*n* possible assignments.
So the approach of "trying everything" is not efficient for large *n*.
In fact, this is the [SAT problem](https://en.wikipedia.org/wiki/Boolean_satisfiability_problem), which is NP-complete, meaning that it is "one of the hardest problems to which solutions can be verified quickly".

```python
a = Bool('a')
b = Bool('b')
c = Bool('c')

s = Solver()
s.add(Or(a, Not(b)))
s.add(Or(Not(a), c))
s.check() # do these equations have a solution?
s.model() # get the solution
```

**Output**: `[b = False, a = False, c = False]`

This says that the formula `(a | !b) & (!a | c)` will be true when `a`, `b`, and `c` are all false.
But what if we want a different solution?

We can add an additional constraint that says that we don't want this solution.

```python
s.add(Not(And(Not(a), Not(b), Not(c))))
s.check()
s.model()
```

**Output**: `[b = False, c = True]`

This says that the formula `(a | !b) & (!a | c)` will be true when `b` is false and `c` is true (it doesn't matter what `a` is).

Let's try adding one more constraint:

```python
s.add(And(Not(a), b))
s.check()
```

**Output**: `unsat`

*unsat* means *unsatisfiable*. The solver is telling us that there is *no possible choice* of `a`, `b`, `c` that makes the current constraints true.

### Intro to SMT

*SMT* stands for *SAT modulo theories*, which means that we can reason about more than just boolean values. For example, we can find a solution for the system of equations {`3x - 2y = 1`, `y - x = 1`}.

```python
x, y = Ints('x y')
solve(3 * x - 2 * y == 1, y - x == 1)
```

**Output**: `[x = 3, y = 4]`

*Note*: `solve` is shorthand for the `check` and `model` we were using above.

We can also find strings `s` and `t` such that `s ++ t == "foo" + s`:

```python
s, t = Strings('s t')
solve(s + t == "foo" + s)
```

**Output**: `[s = "", t = "foo"]`

### Solving Puzzles with SMT

SMT solvers are useful for more than finding solutions to a random set of equations -- we just have to know how to encode the problem in a form that the SMT solver understands.
Here are two examples of puzzles that an SMT solver can solve.
Next time you're working on a puzzle, ask yourself whether it could be solved using SMT.

(Solutions from <https://ericpony.github.io/z3py-tutorial/guide-examples.htm>)

#### Cats and Dogs

Say that you want to spend exactly 100 dollars to buy exactly 100 animals. Dogs cost 15 dollars, cats cost 1 dollar, and mice cost 25 cents each. You have to buy at least one of each. How many of each should you buy?

```python
# Create 3 integer variables
dog, cat, mouse = Ints('dog cat mouse')

solve(dog >= 1,                   # at least one dog
      cat >= 1,                   # at least one cat
      mouse >= 1,                 # at least one mouse
      dog + cat + mouse == 100,   # 100 animals total
      # We have 100 dollars (10000 cents):
      #   dogs cost 15 dollars (1500 cents), 
      #   cats cost 1 dollar (100 cents), and 
      #   mice cost 25 cents 
      1500 * dog + 100 * cat + 25 * mouse == 10000)
```

#### Sudoku

[Sudoku](https://sudoku.com/) is a puzzle where the goal is to insert numbers in boxes to satisfy the following condition: each row, column, and 3x3 box must contain the digits 1 through 9 exactly once.
The program in [sudoku.py](./sudoku.py) finds the solution for the puzzle shown below.

![sudoku example](https://ericpony.github.io/z3py-tutorial/examples/sudoku.png)

## Project: An Automated Assistant for Course Management

Develop a class-to-room scheduler for UMD:

* Input: List of classes, with sizes
* Input: Available rooms, with available time slots and sizes
* Output: Assignment of classes to rooms

We've provided a basic solution in **TODO**, now it's up to you to turn this into a research project!
Here are a few ideas for directions you might explore:

* Add new features to our solution. What do you think would be most useful?
What do you think would be most useful to add?
  * Teachers may have a preference for time of day.
  * Teachers prefer classrooms close to their offices.
  * Some classes are Tu/Th, and some are Mo/We/Fr.
  * Small classes should be in small classrooms (don't put a 15 person seminar in a lecture hall).
  * Different classes have different durations.
  * Some classes shouldn't overlap (e.g., classes that many people take together).

* How do you think we should evaluate the solution?

  * Try implementing the scheduler without an SMT solver. How does your solution look different? Was it harder/easier? How does it compare in terms of performance?
  * Try a different type of solver (e.g., ILP), how does your solution look different?
  * Try your solution on larger inputs -- does it scale?
