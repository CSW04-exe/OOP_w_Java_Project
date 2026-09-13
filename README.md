# Java Matrix Calculator

A desktop application, written in Java, that performs core linear algebra operations on matrices through both a command-line entry point and a full Swing graphical interface. I built it as a hands-on exercise in object-oriented design for my Object-Oriented Programming with Java course.

## Purpose

I built this project to turn abstract OOP coursework into something concrete and usable: a matrix calculator I can actually open, type numbers into, and get correct linear algebra results back from. Matrices were a natural fit for practicing OOP because the domain has real structure to model (rows, columns, elements, operations that transform one matrix into another) and real edge cases to handle (non-square matrices, singular matrices, mismatched dimensions).

Beyond satisfying the class assignment, my goal was to practice the full loop of building a small but real Java application: designing a domain class, separating computation from presentation, wiring up a GUI, persisting state between runs, and pulling in an external library once the math got harder than what was reasonable to hand-roll myself.

## Problem and Approach

The assigned problem was to build a working matrix calculator in Java that supports the standard set of matrix operations — addition, subtraction, multiplication, scalar scaling, transpose, inverse, determinant, and eigenvalues/eigenvectors — and to present it through a usable interface rather than only a command line.

I approached the design by splitting the work into two layers, both living under a `domainModel` package to keep the domain logic separate from the UI:

- **Computation layer** (`MatrixCalculator.java`) — a set of static methods that take raw `int[][]`/`double[][]` arrays and return the result of each operation. Addition, subtraction, multiplication, transpose, and scaling are straightforward nested-loop array operations I wrote by hand. Matrix inverse is implemented from scratch using Gauss–Jordan elimination on an augmented `[matrix | identity]` array, including pivoting when a zero shows up on the diagonal and detecting singular (non-invertible) matrices.
- **Presentation layer** (`MatrixCalculatorGUI.java`) — a `JFrame`-based Swing GUI that lets a user type in matrix dimensions, fill in values through `JTable` grids, click a button per operation, and see the result in a third table.

For determinant and eigenvalue/eigenvector calculation, I shifted my approach partway through: rather than implementing LU decomposition and eigen-decomposition by hand, I pulled in Apache Commons Math (`commons-math3`) and delegated to its `LUDecomposition` and `EigenDecomposition` classes. That was a deliberate trade-off on my part — reimplementing eigen-decomposition correctly and numerically stably from scratch felt out of scope for the assignment, so I used a well-tested library for that piece while keeping the simpler operations hand-written.

I also planned for a dedicated `Matrix` domain class (`domainModel/Matrix.java`) that would encapsulate a matrix behind `getElement`/`setElement`/`getRows`/`getCols` accessors, the way a textbook OOP design would. In the version I actually shipped, that class never got finished — its method bodies are still template stubs — and the calculator ended up operating directly on 2D arrays through the static utility methods instead. I'm leaving that gap in on purpose rather than deleting the file, because it's an honest part of how the design evolved.

## Structure and Methodologies

**Classes and responsibilities**
- `Matrix` — the intended OOP wrapper around a `private double[][] data` field, exposing `getElement`, `setElement`, `getRows`, and `getCols`. It represents my first attempt at proper encapsulation for this domain, though it was never wired into the rest of the app.
- `MatrixCalculator` — the computational core: a static-utility-class style with every operation (`addMatrices`, `subtractMatrices`, `multiplyMatrices`, `transposeMatrix`, `scaleMatrix`, `inverseMatrix`, `detMatrix`, `eigenvalueMatrix`, `eigenvectorMatrix`) implemented as a stateless `public static` method. It also owns a small CSV persistence layer (`saveMatrixToCSV` / `loadMatrixFromCSV`) and a command-line `main` for exercising the logic without the GUI.
- `MatrixCalculatorGUI` — a `JFrame` subclass that owns all Swing state (text fields for dimensions, `JTable`s for the two input matrices and the result, and one button per operation), reads matrices out of the tables, calls into `MatrixCalculator`, and writes results back into the result table.

**Data structures**
- Matrices are represented as plain 2D Java arrays: `int[][]` for user input, and `double[][]` for results that aren't whole numbers (an inverse or a scaled matrix, for example).
- `MatrixCalculator.inverseMatrix` builds an augmented `n x 2n` array (`[matrix | identity]`) to run Gauss–Jordan elimination in place.

**Frameworks and libraries**
- **Java Swing** (`javax.swing`, `java.awt`) for the GUI — `JFrame`, `JTable`, `JScrollPane`, `JButton`, `JOptionPane`, and `GridBagLayout`/`BoxLayout`/`FlowLayout` to arrange the dimension inputs, the three matrix tables, and the operation buttons.
- **Apache Commons Math 3** (`org.apache.commons:commons-math3:3.6.1`), the one external dependency declared in `pom.xml`, used for `MatrixUtils`, `LUDecomposition` (determinant), and `EigenDecomposition` (eigenvalues/eigenvectors).
- **Maven** as the build tool, targeting Java 21 (`maven.compiler.release`), with `domainModel.MatrixCalculator` configured as the executable main class.
- Plain Java file I/O (`BufferedReader`/`BufferedWriter`, `FileReader`/`FileWriter`) for a small CSV-based persistence layer that saves the last computed result to `history.csv` and reloads it as a "History" table the next time the GUI starts.

**OOP and design practices I used**
- Separation of concerns between a computation class and a GUI class — an informal MVC-style split, with the GUI acting as view/controller and `MatrixCalculator` as the model logic.
- Method overloading: `saveMatrixToCSV` has both an `int[][]` and a `double[][]` overload so the same persistence code handles both whole-number results (add, multiply) and fractional results (inverse, scale).
- Defensive programming and input validation throughout the GUI — dimension parsing, empty-cell handling (defaulting to 0), square-matrix checks before inverse/determinant/eigen operations, and try/catch blocks around every button action paired with `JOptionPane` error dialogs.
- A static-utility-class style for the computational core, keeping every matrix operation stateless rather than tied to instance state.

There is no `src/test` directory in this project — I did not write an automated test suite; correctness was checked by running the GUI and CLI entry points manually.

## Process

1. **Early scaffolding** — I started the repository with placeholder/test commits, which is also why the unfinished `Matrix.java` stub is still sitting in the codebase — an early, abandoned attempt at a proper object-oriented matrix wrapper before I settled on an array-based approach.
2. **Core logic drop** — the bulk of the computational logic (`MatrixCalculator.java`) went in as one commit, based on a screenshot shared in my team's Discord, which is why the array-based, static-method style differs from the encapsulated `Matrix` class I'd started earlier — two different design instincts meeting in the same codebase.
3. **Making it "fully functional"** — I built out the Swing GUI and iterated on it repeatedly: getting the matrix tables sized and laid out correctly, wiring every button to its operation, and fixing auto-resize behavior on the tables.
4. **Fixing correctness issues** — I corrected the scale operation to properly handle `double` scalars instead of truncating to integers, alongside cleaning up the on-screen instructions.
5. **Reaching for a library** — I added Apache Commons Math once I needed determinant and eigen-decomposition, rather than continuing to hand-write increasingly complex numerical algorithms myself.
6. **Adding persistence** — I introduced the CSV save/load path so a result would survive an application restart, shown as a "History" panel on the next launch.
7. **Squashing a numerical bug** — the determinant calculation needed a follow-up fix after floating-point rounding produced incorrect results, which is why `detMatrix` explicitly rounds its output today.
8. **Polish and documentation** — I clarified the in-app instructions for end users and wrote up this project description.

This progression — stub class, then a working procedural core, then a GUI wrapped around it, then a library swapped in for the hard math, then bug fixes and documentation — reflects how I actually worked through a multi-person, deadline-driven OOP class project, warts (like the unfinished `Matrix` class) and all.

## Outcome

What I ended up with is a Swing desktop application that reliably performs eight distinct matrix operations — add, subtract, multiply, transpose, scale, inverse, determinant, and eigenvalue/eigenvector extraction — validates its own inputs, reports meaningful errors instead of crashing (e.g. "Matrix is singular (not invertible)!" or "Cannot calculate eigenvalue. Make sure matrix is square and valid."), and remembers the last result across restarts via CSV persistence. There is no automated test suite in the repository, so the honest measure of success here is functional and behavioral: every operation exposed in the GUI has a corresponding, exercised code path, and the one known numerical bug I found (determinant rounding) was caught and fixed rather than left in.

Working on this project reinforced several concrete skills for me:

- **Numerical algorithm implementation** — writing Gauss–Jordan elimination by hand for matrix inversion, including pivoting and singularity detection, rather than treating it as a black box.
- **Knowing when to stop hand-rolling and use a library** — recognizing that eigen-decomposition was a better fit for a well-vetted dependency (Apache Commons Math) than a from-scratch implementation, and integrating that dependency cleanly through Maven.
- **Event-driven GUI programming** — wiring Swing components and action listeners to a separate computation layer, and handling the many ways a user can enter bad input.
- **Working in an evolving codebase** — building on a teammate's contribution, and living with (and learning from) an earlier design decision of my own, the unfinished `Matrix` class, that didn't end up being the direction the project took.

More broadly, this project demonstrates my ability to take a somewhat open-ended assignment, break it into a computation layer and a presentation layer, iterate toward a functioning tool across many small commits, and debug a real numerical correctness issue after the fact rather than just getting something to compile. I'm leaving the unfinished `Matrix` class in deliberately as an honest artifact of that process — a reminder that the first OOP design isn't always the one that ships, and that recognizing when to pivot is itself part of what I learned.

## How to Build/Run

This is a standard Maven project targeting Java 21.

```
mvn package
mvn exec:java
```

`mvn exec:java` runs the class configured as `exec.mainClass` in `pom.xml` (`domainModel.MatrixCalculator`), which is the command-line entry point. To launch the Swing GUI instead, run the GUI class directly, e.g.:

```
mvn compile exec:java -Dexec.mainClass=domainModel.MatrixCalculatorGUI
```
