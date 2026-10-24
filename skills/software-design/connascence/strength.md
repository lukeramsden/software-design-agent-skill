## [Strength](strength.md "Permalink to Strength")

The *strength* of a form of connascence is determined by the ease with which that type of coupling can be refactored. For example, [connascence of name](name.md) is a weak form of connascence because renaming entities across a codebase is usually reasonably trivial. However, [connascence of meaning](meaning.md) is considered a stronger form of connascence since semantic meaning is harder to find across an entire codebase.

Static connascences are considered to be weaker than dynamic connascences, since static connascences can be determined simply by examining the source code. Dynamic connascences require knowledge of run-time behavior, and thus are harder to reason about.

Strength and [locality](locality.md) should be considered together. Stronger forms of connascence are often found within the same function, class, or module where their impact can be more easily observed.

---

Source: https://connascence.io/strength.html

Original content from connascence.io, licensed under [CC BY-SA 3.0](https://creativecommons.org/licenses/by-sa/3.0/). Converted from HTML to Markdown; site navigation and footer omitted.
