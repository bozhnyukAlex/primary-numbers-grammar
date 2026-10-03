# Prime Numbers Grammar

**Recognizing unary prime numbers with a Turing machine and constructing grammar derivations in Kotlin.**

[![Language: Kotlin](https://img.shields.io/badge/implementation-Kotlin-7F52FF)](src/)
[![Runtime: JVM](https://img.shields.io/badge/runtime-JVM-555555)](src/Main.kt)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue)](LICENSE)

An educational formal languages project exploring the connection between **machine computation and grammar rewriting**. The implemented T0 workflow loads a Turing machine, converts its transitions into an unrestricted grammar, checks decimal inputs in unary representation, and reconstructs a derivation for accepted inputs.

[Quick start](#quick-start) · [How it works](#how-it-works) · [Generated output](#generated-output) · [Implementation](#implementation) · [Project status](#project-status)

## The problem

Recognize the language of unary representations of prime numbers:

```text
L = { 1^p | p is a prime number }
```

Here, `1^p` means a word containing `p` copies of the symbol `1`. For example, decimal `2` becomes `11`, and decimal `5` becomes `11111`.

The original assignment connects two pairs of formal models:

| Machine model | Corresponding grammar | Repository status |
| --- | --- | --- |
| Turing machine | Type-0 unrestricted grammar | T0 workflow implemented |
| Linear bounded automaton (LBA) | Type-1 context-sensitive grammar | Machine definition included; Kotlin workflow incomplete |

The implementation writes a **linear derivation trace** consisting of successive words. It does not export a parse-tree visualization.

## Quick start

### Requirements

- A JDK with the `java` command available.
- The Kotlin command-line compiler, `kotlinc`.
- A POSIX shell for the compilation command below.

The source uses the Kotlin and Java standard libraries; there is no Gradle or Maven build configuration.

### Compile and run

```bash
git clone https://github.com/bozhnyukAlex/primary-numbers-grammar.git
cd primary-numbers-grammar/src

kotlinc $(find . -name '*.kt') -include-runtime -d main.jar
java -jar main.jar T0
```

**Run the application from `src/`.** Resource paths are resolved relative to that directory, including `../res/automatons/prime_tm.txt`.

### Interactive example

Enter a decimal integer at the prompt. The application converts it to unary internally:

```text
>> 2
Yes, number is prime
Derivation is written in file
>> quit
```

For rejected inputs, the application prints:

```text
No, number is not prime
```

Use small inputs starting at `2` when exploring the project. Unary encoding allocates a word whose length equals the input number, and the machine simulation has no step limit.

## How it works

1. **Load the machine.** Read states, alphabets, and transitions from [`prime_tm.txt`](res/automatons/prime_tm.txt).
2. **Construct the grammar.** Translate the machine into productions using `ConverterT0`, then write the grammar to disk.
3. **Encode the input.** Convert a decimal number `n` to a word of `n` symbols `1`.
4. **Simulate execution.** Run the Turing machine and record the transitions used for that input.
5. **Reconstruct an accepted derivation.** Apply grammar productions corresponding to the recorded transitions, then remove auxiliary symbols to recover the terminal word.

The derivation builder follows the machine's recorded execution. It does not perform a general search over all possible grammar derivations.

## Generated output

Paths below are relative to the repository root:

| File | Contents | When it changes |
| --- | --- | --- |
| [`res/grammars/grammar_t0.txt`](res/grammars/grammar_t0.txt) | Generated terminals, nonterminals, start symbol, and productions | Overwritten when the T0 application starts |
| [`res/derivations/T0.txt`](res/derivations/T0.txt) | Successive words in the derivation for an accepted input | Overwritten after each accepted input |

A rejected input leaves the previous derivation file unchanged.

### Reading a derivation

The repository includes a derivation for `2`. This abbreviated excerpt shows its initial configuration and the final cleanup:

```text
(epsilon|_), (epsilon|_), (epsilon|_), (epsilon|_), q00, (1|1), (1|1), (epsilon|_), (epsilon|_)
...
1, 1, finish, finish, finish
1, 1, finish, finish
1, 1, finish
1, 1
```

- Paired symbols such as `(1|1)` encode an original input symbol and its current tape symbol.
- State symbols such as `q00` track the simulated machine configuration.
- `epsilon` denotes an empty component, and `_` denotes a blank tape symbol.
- `finish` is the accepting state of the supplied Turing machine.
- After auxiliary symbols are removed, the final word `1, 1` represents decimal `2`.

Each line is a successive word in the trace; the file does not contain separate production labels.

## Implementation

| Source | Responsibility |
| --- | --- |
| [`src/Main.kt`](src/Main.kt) | Application entry point |
| [`Console.kt`](src/org/prime/util/Console.kt) | Mode selection, input loop, resource paths, and output files |
| [`Converter.kt`](src/org/prime/util/Converter.kt) | Turing-machine-to-grammar conversion |
| [`Grammar.kt`](src/org/prime/util/Grammar.kt) | Grammar representation and production application |
| [`DerivationBuilder.kt`](src/org/prime/util/DerivationBuilder.kt) | Reconstruction of a derivation from recorded transitions |
| [`UtilsFactory.kt`](src/org/prime/util/UtilsFactory.kt) | Mode-specific converter and derivation-builder selection |
| [`machines/TapeMachine.kt`](src/org/prime/util/machines/TapeMachine.kt) | Machine execution and transition history |
| [`machines/MachineReader.kt`](src/org/prime/util/machines/MachineReader.kt) | Loading machine descriptions from text files |
| [`res/automatons/`](res/automatons/) | Turing machine and LBA definitions |

## Project status

### Implemented

- Loading and simulating the supplied Turing machine.
- Converting its transitions to an unrestricted T0 grammar.
- Decimal input with internal unary encoding.
- Recording transitions and reconstructing accepted derivations.
- Exporting the grammar and the most recent accepted derivation.

### Incomplete

The CLI recognizes `T1`, but that workflow is not runnable: `ConverterT1.machineToGrammar`, `DerivationBuilderT1.buildDerivation`, and `LinearBoundedAutomaton.prepareInputForTape` contain `TODO("Not yet implemented")`.

The LBA description in [`prime_lba.txt`](res/automatons/prime_lba.txt) uses `a` as its unary input symbol. Its Kotlin input preparation and grammar conversion remain unfinished.

### Scope and limitations

- This is a coursework implementation of formal models, intended for small examples.
- Input validation is limited; negative integers are not handled gracefully.
- Simulation has no timeout or maximum transition count.
- Resource paths depend on the current working directory.
- Output files are overwritten rather than accumulated.
- The repository has no automated test suite or pinned compiler version.

## Author and license

Created by [Alexander Bozhnyuk](https://github.com/bozhnyukAlex).

Licensed under the [MIT License](LICENSE).

