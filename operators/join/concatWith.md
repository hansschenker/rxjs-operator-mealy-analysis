# `concatWith` — Mealy 6-tuple

| | |
|---|---|
| Category | Join |
| Kind | Pipeable join |
| RxJS 7.x source | `src/internal/operators/concatWith.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `concatWith(...otherSources): OperatorFunction<T, T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`concatWith` is a pipeable join on the RxJS 7.x line. Stable. Forward the source to completion, then subscribe to each additional source in order. One active subscription. An error skips the rest.

In plain terms, the operator keeps this memory: { reading(i) | 0 ≤ i ≤ n } ∪ { stopped }. i = 0 is the piped source. At subscription, before any source notification, that memory is reading(0). It reacts to these events: { innerNext, innerError, innerComplete, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: of(1).pipe(concatWith(of(2, 3))) writes next(1) next(2) next(3) complete.

Details that a marble diagram often leaves out: Later sources are not subscribed until earlier ones complete. Implemented as concat of the source plus the rest.

## Role in the notification machine

Forward the source to completion, then subscribe to each additional source in order. One active subscription. An error skips the rest.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`{ reading(i) | 0 ≤ i ≤ n } ∪ { stopped }`. `i = 0` is the piped source.

## 2. Initial state (S0)

`reading(0)`.

## 3. Input alphabet (Z)

`{ innerNext, innerError, innerComplete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next, error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `innerNext` stays on i.
- `innerComplete` moves to i+1 or stopped.
- `innerError → stopped`.

## 6. Output function (G : S × Z → A*)

- `innerNext → next`.
- `innerComplete → ε` if another source remains, else `complete`.
- `innerError → error`.

## Worked trace

`of(1).pipe(concatWith(of(2, 3)))` writes `next(1) next(2) next(3) complete`.

## Why this is Mealy rather than Moore

Complete-versus-ε on innerComplete is selected by the index state.

## Edge cases fixed by the 7.x source

- Later sources are not subscribed until earlier ones complete.
- Implemented as concat of the source plus the rest.

## Source anchors

- `src/internal/operators/concatWith.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
