# `materialize` — Mealy 6-tuple

| | |
|---|---|
| Category | Utility |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/materialize.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `materialize(): OperatorFunction<T, Notification<T>>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Turn next into `next(Notification.createNext(v))`. Turn error into `next(Notification.createError(e))` followed by `complete`. Turn complete into `next(Notification.createComplete())` followed by `complete`. Downstream sees no error from the source; errors are values.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { active, stopped }`.

## 2. Initial state (S0)

`active`.

## 3. Input alphabet (Z)

`{ next, error, complete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(Notification), complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Source next stays active.
- Source error or complete → `stopped`.

## 6. Output function (G : S × Z → A*)

- `next(v) → next(N.Next(v))`.
- `error(e) → next(N.Error(e)) · complete`.
- `complete → next(N.Complete()) · complete`.

## Worked trace

A failing source becomes a completing source whose last value is an error notification.

## Why this is Mealy rather than Moore

Error input writes a two-letter word of next-then-complete. Next input writes a single next. Same active state.

## Edge cases fixed by the 7.x source

- The output does not error for source errors.
- Useful before a delay if error timing must be queued like values.

## Source anchors

- `src/internal/operators/materialize.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
