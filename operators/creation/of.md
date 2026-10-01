# `of` — Mealy 6-tuple

| | |
|---|---|
| Category | Creation |
| Kind | Cold list producer |
| RxJS 7.x source | `src/internal/observable/of.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `of(...values, scheduler?: SchedulerLike): Observable<T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Subscribe emits each captured argument in order and then completes. With a scheduler, each emission and the complete are scheduled actions. Arguments are captured when `of` is called, not when it is subscribed — unlike `defer`.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { pending(i) | 0 ≤ i ≤ n } ∪ { stopped }`. `n` is the argument count. `pending(i)` means the next argument to emit is index `i`.

## 2. Initial state (S0)

`pending(0)`.

## 3. Input alphabet (Z)

`{ subscribe, step, unsubscribe }`. Synchronous path treats subscribe as the start of the step loop.

## 4. Output alphabet (A)

`{ next(values[i]), complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `pending(i) × step → pending(i+1)` for `i < n`.
- `pending(n) × step → stopped`.
- `unsubscribe → stopped`.

## 6. Output function (G : S × Z → A*)

- `pending(i) × step → next(values[i])` for `i < n`.
- `pending(n) × step → complete`.

## Worked trace

`of('a','b')` writes `next('a') next('b') complete` on subscribe.

## Why this is Mealy rather than Moore

Index state selects which argument `G` writes. The step input is what fires that letter.

## Edge cases fixed by the 7.x source

- A trailing scheduler argument is not a value.
- `of()` writes only `complete`.

## Source anchors

- `src/internal/observable/of.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
