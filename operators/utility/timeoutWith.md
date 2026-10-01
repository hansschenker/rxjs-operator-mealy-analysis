# `timeoutWith` — Mealy 6-tuple

| | |
|---|---|
| Category | Utility |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/timeoutWith.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `timeoutWith(due, withObservable, scheduler?): OperatorFunction<T, T | R>` |
| Status on the 7.x line | Deprecated. Use `timeout({ each, with })`. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Numeric timeout that switches to `withObservable` instead of erroring. Same waiting-timer machine as `timeout` with a replacement branch.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { waiting, switched, stopped }`.

## 2. Initial state (S0)

`waiting`.

## 3. Input alphabet (Z)

`{ next, error, complete, timeoutTick, innerNext, innerError, innerComplete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next, error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Timeout tick → `switched` and subscribe to `with`.
- Source next rearms while waiting.

## 6. Output function (G : S × Z → A*)

- Timeout tick → `ε` (switch action).
- Inner notifications then copy through.
- No TimeoutError on the default path.

## Worked trace

A silent source and `timeoutWith(1000, of('fallback'))` writes `next('fallback') complete`.

## Why this is Mealy rather than Moore

Timeout input selects the switch word. Deprecated alias of the `with` branch of `timeout`.

## Edge cases fixed by the 7.x source

- Deprecated.
- Scheduler defaults to async.

## Source anchors

- `src/internal/operators/timeoutWith.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
