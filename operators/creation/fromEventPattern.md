# `fromEventPattern` — Mealy 6-tuple

| | |
|---|---|
| Category | Creation |
| Kind | Hot-source adapter with custom add/remove |
| RxJS 7.x source | `src/internal/observable/fromEventPattern.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `fromEventPattern(addHandler, removeHandler?, resultSelector?): Observable<T>` |
| Status on the 7.x line | Stable. `resultSelector` deprecated. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Generalization of `fromEvent` for APIs that are not DOM/EventEmitter. Subscribe calls `addHandler` with the machine's handler. Each handler call is `next`. Unsubscribe calls `removeHandler` with the same handler.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { idle, listening, stopped }`.

## 2. Initial state (S0)

`idle`.

## 3. Input alphabet (Z)

`{ subscribe, handler(args), unsubscribe }`.

## 4. Output alphabet (A)

`{ next(value), error(e) }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `idle × subscribe → listening` if `addHandler` returns.
- `idle × subscribe → stopped` if `addHandler` throws.
- `listening × handler(args) → listening`.
- `listening × unsubscribe → stopped`.

## 6. Output function (G : S × Z → A*)

- `subscribe` throw → `error(e)`.
- `handler(args) → next(project(args))`. Several arguments become an array when no result selector is used.
- `unsubscribe → ε` after `removeHandler`.

## Worked trace

A custom bus: subscribe calls `add`, two handler fires write two `next`s, unsubscribe calls `remove`.

## Why this is Mealy rather than Moore

Same pattern as `fromEvent`: the handler symbol is productive only in `listening`.

## Edge cases fixed by the 7.x source

- `removeHandler` is optional in the type but required for a correct unsubscribe transition.
- Handler identity must be stable so remove matches add.

## Source anchors

- `src/internal/observable/fromEventPattern.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
