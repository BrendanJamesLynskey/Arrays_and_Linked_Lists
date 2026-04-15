# Arrays and Linked Lists — Presentation

An interactive slide deck covering contiguous and linked memory models, static and dynamic arrays, singly/doubly/circular linked lists, skip lists, XOR linked lists, unrolled linked lists, and real-world performance considerations. Aimed at mid-level software engineers.

## [Open Presentation](https://brendanjameslynskey.github.io/Arrays_and_Linked_Lists/index.html)

## [Markdown Version](presentation.md)

---

## Contents

| # | Topic |
|---|-------|
| 01 | Title |
| 02 | Contiguous vs linked memory |
| 03 | Static arrays — allocation and layout |
| 04 | Array indexing and cache locality |
| 05 | Dynamic arrays — growth strategy |
| 06 | Amortised analysis of dynamic arrays |
| 07 | Singly linked lists — node structure and traversal |
| 08 | Singly linked list operations — insert, delete |
| 09 | Doubly linked lists — bidirectional traversal |
| 10 | Sentinel nodes — eliminating boundary checks |
| 11 | Circular linked lists |
| 12 | Skip lists — probabilistic balancing |
| 13 | Skip list operations — search, insert, delete |
| 14 | XOR linked lists — memory-efficient doubly linked |
| 15 | Unrolled linked lists |
| 16 | Arrays vs linked lists — performance comparison |
| 17 | Real-world implementations |
| 18 | Cache performance and memory fragmentation |
| 19 | Applications and use cases |
| 20 | Summary and further reading |

---

## Slide Controls

| Action | Key |
|--------|-----|
| Next / Previous | `→` `←` or swipe |
| Overview | `Esc` |
| Fullscreen | `F` |
| Export to PDF | append `?print-pdf` to URL |

---

## Technology

[Reveal.js 4.6](https://revealjs.com) · [highlight.js](https://highlightjs.org) (Monokai) · Playfair Display + DM Sans + JetBrains Mono

Single self-contained `index.html` — no build step, no npm.

---

## References

- Cormen, T. et al. *Introduction to Algorithms* (CLRS), 4th ed. MIT Press, 2022
- Pugh, W. "Skip Lists: A Probabilistic Alternative to Balanced Trees." *Communications of the ACM*, 1990
- Stroustrup, B. "Why you should avoid Linked Lists" — keynote talk on cache effects
- Linux kernel `include/linux/list.h` — intrusive doubly linked list implementation
- Sedgewick, R. & Wayne, K. *Algorithms*, 4th ed. Addison-Wesley, 2011

## License

Educational use. Code examples provided as-is.
