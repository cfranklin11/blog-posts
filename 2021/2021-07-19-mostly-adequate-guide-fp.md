Link: https://mostly-adequate.gitbook.io/mostly-adequate-guide/

- "The philosophy of functional programming postulates that side effects are a primary cause of incorrect behavior."
Chapter 03: Pure Happiness with Pure Functions

- Chapter 04: Currying: Using currying to build specialised versions of lower-level utility functions is kind of like inheritance in OOP, using abstract classes to provide base behaviour for more-specific concrete classes

- "We hold composition as a design principle above all others. This is because it keeps our app simple and reasonable."
Chapter 05: Coding by Composing

- Chapter 06: Example Application: When creating curried versions of multi-param functions, he sometimes changes the order. This is because he already knows what order the arguments will come in (e.g. we'll know which container element to select before we have the HTML to set it to), but I wonder if it's a matter of guessing and going back if you guessed wrong, or if there's some rule of thumb. Also, what do you do if you have one case where the args come in one order, but come in a different order elsewhere? Do you have two localised versions to keep them separate? That sort of inconsistency seems like it would be confusing.

- Summary of the mathematical identities/principles/laws
  - Map's composition law: Ch. 6
  - Associative property: Ch. 5 (compose)

- Chapter 07: Hindley-Milner and Me: Learning yet another typing syntax is a bit tiresome, especially one without a linter (there seem to be a few projects, but none with updates in the last couple of years)

- "This particular category has categories as objects and functors as morphisms, which is enough to make one's brain perspire. We won't delve too far into this, but it's nice to appreciate the architectural implications or even just the simple abstract beauty in the pattern."
Chapter 08: Tupperware

- Two sections: chapters 1-6 are more practical clean code (I can apply this to day-to-day coding), with some backing theory; chapters 7-12 get into FP-specific stuff that are only useable in an all-or-nothing FP codebase, much more theory.

I. Chapters 1-6
  - Pure functions
    - Similar to hexagonal architecture (isolate side effects at the edges of applications)
    - Minimise state
  - Currying
    - Partial functions (Python)
  - Composition (pointfree style)
    - Piping (R, Tidyverse)
    - Readability vs method chaining
  - Good general practices for writing better code
II. Chapters 7-12
  - Functors
    - Container of data
    - Result, Error, Null in Rust
    - Forces consistency in handling of errors & nulls
  - When am I ever going to use this?
III. FP vs OOP
  - It seems that FP goes from theory (mathematical principles) to higher-quality code, and OOP goes from empirical observations (folk wisdom) to higher-quality code. The former you can prove, the latter you can see.
  - Both FP and OOP engage in a bit of utopianism and no-true-scotsman (if you follow this path, you have great code; if your code sucks, you didn't do it right), but mediocre OOP code is much more understandable than mediocre FP code, because it's based on concrete objects rather than mathematical theory.
