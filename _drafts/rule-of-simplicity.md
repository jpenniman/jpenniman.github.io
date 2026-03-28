---
layout: post
title: Rule of Simplicity
date: 2020-03-20
image: /blog/images/blank-tile.png
author: Jason M Penniman
excerpt: "Rule of Simplicity: Design for simplicity; add complexity only where you must."
tags:
- software engineering
- unix philosophy
---

> Rule of Simplicity: Design for simplicity; add complexity only where you must.

# What is the Rule of Simplicity?

The Rule of Simplicity is one of the foundational guidelines in the Unix philosophy, famously articulated by Doug McIlroy and later summarized by Eric S. Raymond. It states that software should be designed to be as simple as possible, making it easier to understand, maintain, and extend. Simplicity is not about doing less, but about doing only what is necessary, and doing it clearly.

# Why Simplicity Matters

Simplicity in software brings many benefits:

- **Maintainability:** Simple code is easier to read, debug, and modify.
- **Reliability:** Fewer moving parts mean fewer opportunities for bugs.
- **Composability:** Simple tools can be combined in powerful ways.
- **Accessibility:** New contributors can understand and use simple tools more quickly.

# Examples in Unix

Many classic Unix tools embody the Rule of Simplicity:

- `cat`: Concatenates and displays files. It does one thing, and does it well.
- `grep`: Searches for patterns in text. Its interface is straightforward, yet it can be combined with other tools for complex tasks.
- `ls`: Lists directory contents. Its output is simple and easily parsed by other programs.

These tools are small, focused, and easy to understand, making them reliable building blocks for larger workflows.

# Applying Simplicity Today

Modern software development can benefit from the Rule of Simplicity:

- **Write small, focused functions and modules.**
- **Avoid unnecessary features or options.**
- **Prefer clear, readable code over clever tricks.**
- **Document intent and usage simply.**

When in doubt, ask: "Is there a simpler way to achieve this?"

# Common Pitfalls

Some common traps that violate the Rule of Simplicity include:

- **Overengineering:** Adding features or abstractions that aren't needed.
- **Feature creep:** Letting a tool grow beyond its original purpose.
- **Obscure code:** Writing code that's clever but hard to understand.

To avoid these, regularly review your code and designs, and seek feedback from others.

# Conclusion

The Rule of Simplicity is timeless advice for software engineers. By striving for simplicity, we create tools and systems that are robust, maintainable, and a pleasure to use. In the spirit of Unix, let simplicity guide your next project.

