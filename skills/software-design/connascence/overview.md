# What is Connascence?

Connascence is a software quality metric & a taxonomy for different types of coupling.\
This site is a handy reference to the various types of connascence, with examples to help you improve your code.

### Subject to Change

*All code* is subject to change. As the real world changes, so too must our code. Connascence gives us an insight into the long-term impact our code will have on flexibility, as we write it. Maintaining a flexible codebase is essential for maintaining long-term development velocity.

### A Flexible Metric

Connascence is a metric, and like all metrics is an imperfect measure. However, connascence takes a more holistic approach, where each instance of connascence in a codebase must be considered on three separate axes:

1. [Strength](strength.md). Stronger connascences are harder to discover, or harder to refactor.
2. [Degree](degree.md). An entity that is connascent with thousands of other entities is likely to be a larger issue than one that is connascent with only a few.
3. [Locality](locality.md). Connascent elements that are close together in a codebase are better than ones that are far apart.

The three properties of Strength, Degree, and Locality give the programmer all the tools they need in order to make informed decisions about when they will permit certain types of coupling, and when the code ought to be refactored.

### A Vocabulary for Coupling

Arguably one of the most important benefits of connascence is that it gives developers a vocabulary to talk about different types of coupling. Connascence codifies what many experienced engineers have learned by trial and error: Having a common set of nouns to refer to different types of coupling allows us to share that experience more easily.

---

Source: https://connascence.io/

Original content from connascence.io, licensed under [CC BY-SA 3.0](https://creativecommons.org/licenses/by-sa/3.0/). Converted from HTML to Markdown; site navigation and footer omitted.
