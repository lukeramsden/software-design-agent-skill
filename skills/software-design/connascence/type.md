## [Connascence of Type](type.md "Permalink to Connascence of Type")

Connascence of type is when multiple components must agree on the type of an entity. In a statically typed language, these issues are often (but not always) caught by the compiler. Consider the following trivial C++ code:

```
std::string cost;

cost = 10.95; // OOPS!
```

Dynamically typed languages typically suffer from less obvious instances of connascence of type. Consider a function that calculates your age, given your day, month, and year of birth:

```
def calculate_age(birth_day, birth_month, birth_year):
    # do the calculation here:
```

How is this function supposed to be called? Here are a few different options:

```
calculate_age(1, 9, 1984)
calculate_age(1, 9, 84)
calculate_age('1', '9', '1984')
calculate_age('1', 'September', '1984')
```

---

Source: https://connascence.io/type.html

Original content from connascence.io, licensed under [CC BY-SA 3.0](https://creativecommons.org/licenses/by-sa/3.0/). Converted from HTML to Markdown; site navigation and footer omitted.
