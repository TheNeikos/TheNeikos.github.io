---
title: "Rustbreak"
description: "Self-contained fast Keyvalue Database"
repo: "rustbreak"
tags: ["rust", "database"]
weight: 1
draft: false
---

## Summary

Rustbreak is a very simple to use database. It is meant to be used as a quick
drop in database for all sorts of tools, where setting up an SQL infrastructure
would be too much.

## When to use

Use Rustbreak only if you understand these caveats:

- Single file (depending on filesystem this might cause problems later on)
- Meant to be human readable and editable
- Does not scale to multiple concurrent processes (it is thread safe though)

If you need, or forseeable need better than those points, then __do__ not use
Rustbreak! It will not behave well, or work as advised.

*This only holds for Version 2.0, later versions might add more or remove some
of these problems.*

### [Latest Documentation][docs]

[docs]: https://docs.rs/rustbreak/

### Licensing

Rustbreak is dual licensed, it is available for free under the GPL license for
all purposes. If you however plan on making money with it without open-sourcing
your software, you might be better served with a commercial license. You can
request one by contacting me by email.
