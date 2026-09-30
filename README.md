# ChampHQ

Headquarters for all your agents.

This repo is the coordination layer for the Champions Group agent estate. It is deliberately small: it holds the shared map of what exists, not the products themselves.

A Champions Group product.

---

## What lives here

```
Atlas/
  Products/
    ChampHQ/
      Odysseus Productization Plan.md
```

`Atlas/Products/` is the pattern. Every product gets a directory holding the material that outlives a single sprint: the productization plan, the positioning, the open questions. The product code stays in its own repo; this repo holds the thing that makes the portfolio legible as a whole.

---

## How to use it

1. Create `Atlas/Products/<ProductName>/`.
2. Add the productization plan as the entry document.
3. Link the implementation repo at the top of that plan, so the plan and the code never drift apart.

Treat the plan as the source of truth for *why* a product exists and what it is for. Treat the product repo as the source of truth for *how* it currently behaves. When they disagree, that disagreement is the finding.

---

## Why it is not code

Every agent in the estate can read a repository. What none of them can do is invent the context that explains what the estate is *for*. That context is the reason this repo exists, and it is why it is one file today rather than a monorepo.

---

## Status

Early. The Atlas structure is established and the Odysseus productization plan is in progress.
