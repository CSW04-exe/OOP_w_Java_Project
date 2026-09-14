# Java Matrix Calculator

**Type:** Team project
**Contributors:** Carter Ward, ch103hw4n6, dmc0022
**Course:** Object-Oriented Programming with Java
**Completed:** 04/23/2025

## Purpose

We built this project to turn abstract OOP coursework into something concrete and usable: a matrix calculator we could actually open, type numbers into, and get correct linear algebra results back from. Matrices were a natural fit for practicing OOP because the domain has real structure to model (rows, columns, elements, operations that transform one matrix into another) and real edge cases to handle (non-square matrices, singular matrices, mismatched dimensions). Beyond the assignment itself, our goal was to practice the full loop of building a small but real Java application, from domain design through a working GUI.

## Problem and Approach

The assignment was to build a working matrix calculator in Java supporting the standard set of operations — addition, subtraction, multiplication, scalar scaling, transpose, inverse, determinant, and eigenvalues/eigenvectors — presented through a usable interface rather than only a command line. We split the work into two layers under a `domainModel` package: a computation layer (`MatrixCalculator.java`) of static methods operating on raw 2D arrays, including a hand-written Gauss–Jordan elimination for matrix inversion with pivoting and singularity detection; and a presentation layer (`MatrixCalculatorGUI.java`), a Swing `JFrame` GUI with `JTable` grids for input and results.

## Structure and Methodologies

- `Matrix` — an intended OOP wrapper class with `getElement`/`setElement`/`getRows`/`getCols` accessors; never finished or wired into the app, left in as-is
- `MatrixCalculator` — static-utility computational core (add, subtract, multiply, transpose, scale, inverse, determinant, eigenvalue, eigenvector), plus CSV save/load and a CLI `main`
- `MatrixCalculatorGUI` — Swing GUI reading/writing `JTable`s and calling into `MatrixCalculator`
- Matrices represented as plain `int[][]`/`double[][]` 2D arrays
- Apache Commons Math 3 (`commons-math3`) for determinant (`LUDecomposition`) and eigenvalues/eigenvectors (`EigenDecomposition`), once hand-rolling those became impractical
- Maven build targeting Java 21, with CSV-based persistence (`BufferedReader`/`Writer`) so the last result survives a restart
- No automated test suite; correctness verified by running the GUI/CLI manually

## Process

1. Scaffolding — early placeholder commits, including the `Matrix` class stub that was later abandoned
2. Core logic — array-based `MatrixCalculator` computational methods dropped in
3. GUI build-out — Swing tables and buttons wired to each operation, with layout iteration
4. Library integration — Apache Commons Math added for determinant/eigen-decomposition
5. Persistence — CSV save/load added for a "History" view across restarts
6. Bug fixes — a determinant floating-point rounding bug was found and fixed
7. Polish — in-app instructions clarified and documentation written

## Outcome

We ended up with a Swing desktop app that reliably performs eight matrix operations, validates input, reports meaningful errors instead of crashing, and persists results across restarts via CSV. The unfinished `Matrix.java` stub remains in the repo on purpose — an honest artifact of an early OOP design (a proper encapsulated wrapper) that we abandoned in favor of the simpler array-based approach once time pressure hit. The project reinforced hand-implementing numerical algorithms (Gauss–Jordan elimination), knowing when to reach for a well-tested library instead (Apache Commons Math for eigen-decomposition), event-driven Swing GUI programming, and working in a shared, evolving team codebase.

## How to Build/Run

```
mvn package
mvn exec:java
```

`mvn exec:java` runs the CLI entry point (`domainModel.MatrixCalculator`). To launch the GUI instead:

```
mvn compile exec:java -Dexec.mainClass=domainModel.MatrixCalculatorGUI
```
