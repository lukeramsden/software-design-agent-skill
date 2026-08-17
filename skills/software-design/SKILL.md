---
name: software-design
description: A practitioner's distillation of John Ousterhout's *A Philosophy of Software Design* (2018) for use during code review, design discussions, and writing. Use it as a checklist of named diagnostics, not as a chapter summary.
---

> "This book is about one thing: complexity."  — Ousterhout

---

## 1. The thesis in one paragraph

The greatest limitation in software is the human capacity to understand the system being built. Therefore the central design goal is to **manage complexity** — anything about the *structure* of a system that makes it hard to understand or modify. Complexity is what a *reader* experiences in the moment, not a function of size or features. It accumulates incrementally from hundreds of small dependencies and obscurities, each one individually defensible, until the system can no longer be cleaned up cheaply. Two strategies fight it: **eliminate** (simplify and clarify) and **encapsulate** (modular design with deep modules and information hiding). Every other rule in this document is downstream of those two.

---

## 2. The 15 principles (Ousterhout's canonical list)

1. **Complexity is incremental** — sweat the small stuff.
2. **Working code isn't enough** — strategic over tactical.
3. **Make continual small investments** in design (~10–20% of dev time).
4. **Modules should be deep.**
5. **Make the common case as simple as possible** in interfaces.
6. **Simple interface > simple implementation** — module devs suffer so users don't.
7. **General-purpose modules are deeper.**
8. **Separate general-purpose and special-purpose code.**
9. **Different layers should have different abstractions.**
10. **Pull complexity downward.**
11. **Define errors (and special cases) out of existence.**
12. **Design it twice** (or more, with radically different alternatives).
13. **Comments should describe things not obvious from the code.**
14. **Software should be designed for ease of reading, not ease of writing.**
15. **The increments of software development should be abstractions, not features.**

## 3. The 14 red flags (Ousterhout's canonical list)

Use these names verbatim in code review — naming the flag is half the diagnosis.

| # | Flag | What it looks like |
|---|------|--------------------|
| 1 | **Shallow Module** | Interface complexity ≈ implementation complexity. Trivial wrapper methods, single-line classes, anything where the interface costs more than it saves. |
| 2 | **Information Leakage** | Same design decision encoded in two or more modules (a file format known to both reader and writer; a parameter format duplicated across parser and serialiser). The master red flag. |
| 3 | **Temporal Decomposition** | Modules carved by execution order ("first read, then parse, then write") instead of by knowledge, causing the same knowledge to live in multiple stages. |
| 4 | **Overexposure** | The common-path API forces users to learn rarely-used features. Java's `BufferedInputStream` is the canonical case. |
| 5 | **Pass-Through Method** | A method that does nothing but forward args to another method with the same signature. |
| 6 | **Repetition** | Non-trivial code (or near-identical code) repeated. Signals a missing abstraction. |
| 7 | **Special-General Mixture** | Special-purpose code embedded in a general-purpose mechanism, leaking the use case into the mechanism. |
| 8 | **Conjoined Methods** | Two methods (or two pieces of code) that can't be understood without reading each other. |
| 9 | **Comment Repeats Code** | A comment a reader could write without understanding the code. Often: comment uses the same words as the entity's name. |
| 10 | **Implementation Documentation Contaminates Interface** | The interface comment describes how, not what — exposing internal data structures, RPC names, private constants. |
| 11 | **Vague Name** | `count`, `result`, `time`, `data`, `blinkStatus` — broad enough to refer to many things. |
| 12 | **Hard to Pick Name** | Difficulty naming = the underlying concept is muddled (often: it's two things in a trench coat). |
| 13 | **Hard to Describe** | If a complete-yet-simple comment is hard to write, the design is bad. *The master diagnostic*. |
| 14 | **Nonobvious Code** | A first-time reader can't understand it on a quick reading. |

A few more named flags from the body of the book that aren't in the official list but are worth knowing: **False Abstraction** (Ch 4), **Tactical Tornado** (Ch 3), **Pass-Through Variable** (Ch 7).

---

## 4. Diagnostic vocabulary

When something feels wrong, use these terms.

**Three symptoms** — what complexity feels like to a reader:
- **Change amplification** — a simple change touches many places.
- **Cognitive load** — readers must hold a lot in their head to make a change safely.
- **Unknown unknowns** — readers can't tell what they need to know. *Worst of the three* — change amplification is annoying but tractable; high cognitive load is expensive but tractable; unknown unknowns mean the only check is reading every line, which doesn't scale.

**Two causes** — what's structurally wrong:
- **Dependencies** — code that can't be understood/modified in isolation. Make remaining ones explicit and obvious; compiler-checked beats convention-checked.
- **Obscurity** — important information not visible. Generic names, undocumented invariants, side-table lookups, inconsistencies. *"The need for extensive documentation is often a red flag that the design isn't quite right."*

The mapping: dependencies → change amplification + cognitive load; obscurity → unknown unknowns + cognitive load.

---

## 5. Modules

### Deep > shallow
A module's *benefit* is functionality; its *cost* is interface size. The best modules hide a lot behind a small interface.
- *Canonical deep:* Unix file I/O (5 syscalls hide 100Ks of LOC); garbage collection (no interface — *adding* GC to a language *shrinks* the language's interface).
- *Canonical shallow:* `LinkedList`; `private void addNullValueForAttribute(String a) { data.put(a, null); }` — costs an interface element to learn, takes more keystrokes to call than to inline.

### Reject classitis
"Classes are good, so more classes are better" → many small shallow classes whose interfaces sum into system-wide complexity. **Length is rarely a reason to split a method.** A 200-line method with simple signature and clear blocks is fine — it's deep.

### Information hiding ≠ private
`private` fields exposed via getters/setters hide nothing. The question is whether the information *escapes the module*, not whether it has the access modifier.

### Information leakage is the master red flag
When you spot the same knowledge in two places, ask: *"How can I reorganise these so this knowledge only affects one module?"* Two repairs:
1. **Merge** the modules.
2. **Extract** the knowledge into a new module — but only if that module can have a *simple* interface. Otherwise you've just moved the leak.

Watch for **back-door leakage**: knowledge shared between modules without appearing in any interface. More dangerous than interface leakage because it's invisible.

### General-purpose interface, today's functionality
Functionality reflects today's needs; the *interface* is general enough to support multiple uses. The general-purpose interface is usually *simpler* than the specialised one even if you only ever have one caller, because it isolates the caller's concept (`Position`, `Range`) from the module's concept.

Three questions:
1. What's the simplest interface covering all my current needs?
2. In how many situations will this method be used? (One = red flag.)
3. Is this API easy to use for my current needs? (Lots of glue at call sites = too generic.)

### Pull complexity downward
Module developers suffer so users don't. Most modules have more users than developers. *Simple interface > simple implementation.*

**Configuration parameters punt complexity upward.** Before exporting one, ask: *"could a user actually determine a better value than this code can?"* If not, compute it. Worked example: TCP retry interval — could be a config knob, or could be measured automatically from observed RTTs and adjusted dynamically.

---

## 6. Layering

> Different layer, different abstraction.

If two adjacent layers expose similar concepts, you have one of:
- **Pass-through methods** — fix by exposing the lower-level class directly, redistributing functionality, or merging.
- **Pass-through variables** — variable threaded through methods that don't use it. Repair with a *context object* (one per system instance, immutable variables, available everywhere but explicit only in constructors). Acknowledged-imperfect but better than alternatives.
- **Shallow decorators** — before reaching for the Decorator pattern, ask: could the new functionality go in the underlying class? in the use site? in an existing decorator? as a stand-alone class?

> "Each piece of design infrastructure added to a system, such as an interface, argument, function, class, or definition, adds complexity, since developers must learn about this element. In order for an element to provide a net gain against complexity, it must eliminate some complexity that would be present in the absence of the design element."

---

## 7. Errors

Exception handling is one of the worst sources of complexity in software systems. *Reduce the number of places where exceptions must be handled.* Four tools, in order of preference:

1. **Define out of existence.** Redesign the operation so the "error" becomes a normal case. Examples:
   - Tcl `unset` should *ensure variable doesn't exist*, not delete; no error to throw.
   - Java `String.substring` should clamp out-of-range indices, not throw `IndexOutOfBoundsException`.
   - Unix file deletion ("mark for deletion") vs Windows ("refuse if open").
2. **Mask** at a low level — TCP retransmits lost packets so callers see a reliable byte stream; NFS hangs on server unavailability rather than aborting.
3. **Aggregate** many handlers into one high in the call chain — a top-level dispatcher catches a base "abort this request" exception with subclasses for distinct error categories. Each lower method just attaches a human-readable message. New error types plug in for free.
4. **Crash** for unrecoverable conditions — OOM, internal inconsistencies, truly unhandleable I/O failures. Provide a `ckalloc`-style wrapper if your language doesn't already.

The standard rebuttal "but throwing catches bugs" is wrong: defining errors away simplifies the API, which reduces total code, which reduces *other* bugs. **The best way to reduce bugs is to make software simpler.**

**Don't** define errors away when callers genuinely need the information (e.g., a network module that swallowed all transport errors made robust apps impossible to build on top of it).

The same logic generalises: **design special cases out of existence**. A text editor's "no selection" state is better modeled as a selection where `start == end`, eliminating every `if (selectionExists)` branch.

---

## 8. Comments

Comments are not documentation tax — they are a **design tool**. The act of writing one is the cheapest opportunity to discover that your design is bad.

### The rule
**Comments should describe things that aren't obvious from the code.** "Obvious" is from the *first-time reader's* perspective, never the author's. If a reviewer says it isn't obvious, it isn't — don't argue.

### How comments augment code
Useful comments are at a *different level of detail* than the code:
- **Lower-level / precision** — units, boundary inclusivity, null semantics, ownership ("who frees this?"), invariants. Best for variable declarations.
- **Higher-level / intuition** — the simplest sentence that explains everything in the block. Best inside methods and for interface comments.

Same-level comments repeat the code and are worthless. Diagnostic test: *"Could someone who has never seen this code write this comment by looking only at the code next to it?"* If yes, the comment adds nothing.

### Variables: think nouns, not verbs
Document *what the variable represents*, not the sites that mutate it. Anti-pattern: `// Toggled to TRUE when X. Toggled to FALSE when Y.` Better: `// True means a heartbeat has been received since the last election timer reset.`

### Interface comments
- **Must not contain implementation details.** A reader should be able to call without reading the body.
- For a class: what abstraction does it provide? what does each instance represent? what are the limitations?
- For a method: behavior as perceived by callers + each arg/return + side effects + exceptions + preconditions.

### Write the comments first
Class interface comment → method signatures + interface comments → iterate → instance variables → method bodies. **When the code is done, the comments are done.** The comment-driven workflow surfaces design problems before they're committed to code.

### The master diagnostic
**Hard to Describe** — if a complete-yet-simple comment is hard to write, the design is bad. The comment is the canary.

### Maintaining comments
- **Keep them near the code.** Far comments rot. Interface comments belong next to the method body, not in a header file.
- **Don't put design knowledge in commit messages.** A future developer won't scan the log.
- **Avoid duplication.** Document each decision once; cross-reference from elsewhere. A stale cross-reference is self-evident; a stale duplicate is invisible.
- **Higher-level comments are easier to maintain** because they don't reflect line-level details.
- **Check the diff before committing** — pre-commit scan catches stale comments and stray TODOs.

---

## 9. Names

> Bad names cause bugs.

The Sprite OS bug: a variable `block` was used both for physical disk-block numbers and logical file-block numbers. A logical was used where a physical was expected. Six months to find. Fix would have been `fileBlock` and `diskBlock`.

### Two qualities
- **Precise** — a reader who sees the name in isolation should be able to guess what it refers to. `count` of what? Better: `numIndexlets`. `blinkStatus`? Better: `cursorVisible` (and boolean names should be predicates).
- **Consistent** — pick a name for each kind-of-thing, use it everywhere for that purpose, *and never reuse it for any other purpose*. Consistency is cognitive leverage: readers can predict behavior without analysing it.

### Length scales with distance
**The greater the distance between a name's declaration and its uses, the longer the name should be.** `i` in a 5-line loop is fine; in a 100-line method it isn't. Loop conventions: `i` outermost, `j` next-nested.

### The naming diagnostic
**Hard to Pick Name** — if you can't name it cleanly, the underlying concept is probably muddled. Often: the variable is secretly two things. Splitting produces two cleaner variables, each with a natural name.

### Names can also be too specific
`delete(Range selection)` — `selection` mis-suggests the method only operates on UI-selected text. Better: `range`.

### A note on Go
The Go style guide prefers very short names. Ousterhout disagrees but offers the meta-rule: *"readability must be determined by readers, not writers."* If your readers find your style readable, fine.

---

## 10. Modifying existing code

> "If you're not making the design better, you are probably making it worse."

When you change code, **leave the design as it would have been if you'd known about that change from the start.** Refactoring is part of every modification. The default mindset *"smallest possible change"* is tactical programming and produces incremental complexity accumulation.

When real-world constraints force a quick fix:
1. Is there an alternative *almost* as clean that fits the deadline?
2. If not, schedule the proper refactor *now* for after the deadline.

---

## 11. Consistency

Two payoffs: **speed** (no re-learning) and **safety** (familiar-looking patterns are correctly familiar).

### Where to apply
Names, coding style, interfaces with multiple implementations, design patterns, invariants.

### Playbook
1. **Document** conventions where developers will see them.
2. **Enforce** with pre-commit scripts. Worked example: a 50-line script rejecting CR characters solved a Windows/Unix line-ending war overnight.
3. **Code reviews** are where conventions get taught.
4. **"When in Rome"** — match the local style first. *Anything that looks like a convention probably is one.*
5. **Don't change existing conventions** just because you have a "better idea." Two questions: (a) Is there significant new information? (b) Is the new approach so much better it's worth updating *every* old use? If both yes, do it fully — leave no trace of the old.

### Taking it too far
Consistency means *similar things done similarly* AND *dissimilar things done differently*. Forcing dissimilar things into the same approach is worse than inconsistency, because it produces false confidence.

---

## 12. Code should be obvious

The test: *"a reader can read the code quickly, without much thought, and their first guesses about the behavior are correct."* Only code review can establish this — the author can't judge.

### Three ways to make code obvious (the framework)
1. **Reduce information needed** — abstraction, eliminate special cases. Best path.
2. **Reuse information readers already have** — follow conventions, conform to expectations.
3. **Present what's needed** — good names, strategic comments.

### Things that make code less obvious
- **Event-driven control flow** — handler invocation is invisible at the call site. Compensate with interface comments stating *when* each handler fires.
- **Generic containers** (`Pair<X,Y>`) — `result.getKey()` says nothing about meaning. Define a small named struct/class instead. *Software should be designed for ease of reading, not ease of writing.*
- **Declared type ≠ allocated type** — `List l = new ArrayList<>()` misleads. Match them.
- **Code that violates reader expectations** — a `main` that returns but the process doesn't exit. Document the violation explicitly.

---

## 13. Performance

**Clean code is usually faster.** Deep classes mean fewer layer crossings. Defining special cases away eliminates the conditionals that check for them.

### Default mode
Develop awareness of which operations are fundamentally expensive. Reference numbers:
- Datacenter network round-trip: 10–50 µs (10s of thousands of instructions).
- Disk I/O: 5–10 ms (millions of instructions).
- Flash: 10–100 µs.
- `malloc`/`new`: significant.
- DRAM cache miss: a few hundred instruction times.

When two clean designs differ in cost, pick the cheap one. Hash table over ordered map unless you need ordering. Inline structs in arrays, not pointers to separately-allocated structs.

### When you need to optimise
1. **Measure first.** Programmer intuition about performance is unreliable, even for experienced developers.
2. **Identify the critical path.** Imagine the smallest possible code that handles the common case, with no special cases and no layer crossings — call this *the ideal*.
3. **Design back from the ideal.** Find a clean structure that comes as close as possible. Push special-case checks to a single upfront test that gates the entire common path.
4. **Re-measure.** If a change didn't measurably help, back it out (unless it also simplified the design).

The RAMCloud `Buffer` rewrite: 2× faster *and* 20% less code. Clean design and high performance are compatible — refute the false dichotomy when you encounter it.

---

## 14. Where Ousterhout pushes back on conventional wisdom

These are positions that often contradict received wisdom. Hold them when relevant.

| Conventional wisdom | Ousterhout's position |
|---|---|
| "Classes should be small." | Classes should be *deep*. Many small classes produces classitis — system-wide complexity from accumulated interfaces. |
| "Split methods over N lines." | Length is rarely the right reason. A long method with simple signature and clear blocks is *deep* — keep it. |
| "Self-documenting code needs no comments." | Comments capture what code can't: rationale, invariants, abstractions. Without comments, *there is no abstraction* — readers must read the implementation. |
| "Throw exceptions to surface errors." | Exceptions are the worst single source of complexity. Define errors out of existence first; mask, aggregate, or crash second. |
| "Make decisions configurable." | Configuration parameters punt complexity to users who often can't make the decision. Compute defaults; expose only when callers genuinely have better information. |
| "Use design patterns to solve problems." | Patterns when they fit, custom when they don't. Forcing a pattern is worse than not using one. *More patterns is not better.* |
| "Use getters and setters." | Getters/setters are shallow methods that re-expose private state. The right move is usually not to expose the variable in the first place. |
| "Inheritance is a core OO mechanism." | Interface inheritance is fine; *implementation inheritance* leaks state across the hierarchy. Prefer composition. |
| "Test-driven development produces good design." | TDD is tactical programming with a process veneer — the unit of design is the abstraction, not the failing test. (Exception: bug-fix tests.) |
| "Build special-purpose now, refactor general later." | The increments of software development should be **abstractions, not features**. |
| "The smart engineer gets it right the first time." | "It isn't that you aren't smart; it's that the problems are really hard." Always sketch a second design. |

---

## 15. Code review checklist

A scannable order of operations.

1. **Is it obvious to a first-time reader?** If the reviewer is confused, fix the code, not the reviewer.
2. **Are interfaces deep?** Check the interface comment alone — does it carry enough to use the module without reading the body? Or does it leak implementation?
3. **Is the same knowledge encoded in two places?** Information leakage is the most expensive bug to leave in.
4. **Pass-through methods? Pass-through variables? Shallow decorators?** Same-abstraction adjacent layers need to merge or split.
5. **Is anything special-cased that could be normalised?** Is there an `if (no-X)` branch where the design could just always have an empty-X?
6. **Are exceptions earning their keep?** Could the operation be redefined so the exception isn't an error? Could the exception be masked or aggregated?
7. **Are configuration parameters punting decisions?** Could the code compute the value?
8. **Are names precise and consistent?** Any name doing two jobs?
9. **Do comments augment with precision or intuition, or do they repeat the code?**
10. **Did the change leave the design as good as it would have been with the new requirement known up front?** If not, refactor as part of this change.

---

## 16. The investment frame

Strategic over tactical. ~10–20% of dev time on design investments. Initial projects 10–20% slower; within months the investment pays for itself; afterwards it's free. Tactical programmers ship 10–20% faster initially, then slow down for the rest of the system's life. *Once a codebase is spaghetti, it is nearly impossible to fix.*

> "The reward for being a good designer is that you get to spend a larger fraction of your time in the design phase, which is fun. Poor designers spend most of their time chasing bugs in complicated and brittle code."
