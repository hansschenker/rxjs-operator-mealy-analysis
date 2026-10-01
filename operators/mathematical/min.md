# `min` — Mealy 6-tuple

| | |
|---|---|
| Category | Mathematical and Aggregate |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/min.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `min(comparer?): OperatorFunction<T, T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`min` is a pipeable operator on the RxJS 7.x line. Stable. Same aggregate machine as `max` with the opposite comparer. Emit the min on complete. Empty source errors `EmptyError`.

In plain terms, the operator keeps this memory: S = { empty, holding(min), stopped }. At subscription, before any source notification, that memory is empty. It reacts to these events: { next, error, complete, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: of(3, 1, 2).pipe(min()) writes next(1) complete.

Details that a marble diagram often leaves out: Empty errors. Default comparer is numeric.

## Role in the notification machine

Same aggregate machine as `max` with the opposite comparer. Emit the min on complete. Empty source errors `EmptyError`.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { empty, holding(min), stopped }`.

## 2. Initial state (S0)

`empty`.

## 3. Input alphabet (Z)

`{ next, error, complete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next, error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- First next seeds holding.
- Later next replaces holding if it compares smaller.

## 6. Output function (G : S × Z → A*)

- `next → ε`.
- `complete` emits min or `EmptyError`.

## Worked trace

`of(3, 1, 2).pipe(min())` writes `next(1) complete`.

## Why this is Mealy rather than Moore

Same Mealy aggregate as `max`; only the comparison inside `T` differs.

## Edge cases fixed by the 7.x source

- Empty errors.
- Default comparer is numeric.

## Source anchors

- `src/internal/operators/min.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
