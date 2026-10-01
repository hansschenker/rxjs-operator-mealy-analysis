# `pairs` — Mealy 6-tuple

| | |
|---|---|
| Category | Creation |
| Kind | Deprecated object enumerator |
| RxJS 7.x source | `src/internal/observable/pairs.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `pairs(obj, scheduler?): Observable<[string, T]>` |
| Status on the 7.x line | Deprecated. Use `from(Object.entries(obj))`. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

On subscribe, enumerate own enumerable keys of `obj` and emit `[key, value]` pairs, then complete. Scheduler spreads the emissions.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`{ pending(i), stopped }` over the entry list captured at subscribe.

## 2. Initial state (S0)

`pending(0)` after the entry list is built.

## 3. Input alphabet (Z)

`{ subscribe, step, unsubscribe }`.

## 4. Output alphabet (A)

`{ next([key, value]), complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Step advances i.
- Past the last entry → stopped.
- Unsubscribe → stopped.

## 6. Output function (G : S × Z → A*)

- Step at i < n → next(entries[i]).
- Step at n → complete.

## Worked trace

`pairs({ a: 1, b: 2 })` writes `next(['a', 1]) next(['b', 2]) complete` in enumeration order.

## Why this is Mealy rather than Moore

The step input writes the entry stored at the index state, the same shape as `of` and `range`.

## Edge cases fixed by the 7.x source

- Deprecated.
- Entries are read at subscribe, not when `pairs` is called, so a mutated object is seen per subscription.

## Source anchors

- `src/internal/observable/pairs.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
