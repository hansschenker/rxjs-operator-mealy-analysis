# `every` — Mealy 6-tuple

| | |
|---|---|
| Category | Conditional and Boolean |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/every.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `every(predicate, thisArg?): OperatorFunction<T, boolean>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`every` is a pipeable operator on the RxJS 7.x line. Stable. Emit `false` and complete on the first value that fails the predicate. If the source completes and none failed, emit `true` and complete. Empty source emits `true` (vacuous truth). Predicate throw is an error.

In plain terms, the operator keeps this memory: S = { checking(i), stopped }. At subscription, before any source notification, that memory is checking(0). It reacts to these events: { next, error, complete, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: every(x => x < 3) on 1 2 3 writes next(false) complete at 3 and unsubscribes.

Details that a marble diagram often leaves out: Empty source is true. Unsubscribes on the first failure.

## Role in the notification machine

Emit `false` and complete on the first value that fails the predicate. If the source completes and none failed, emit `true` and complete. Empty source emits `true` (vacuous truth). Predicate throw is an error.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { checking(i), stopped }`.

## 2. Initial state (S0)

`checking(0)`.

## 3. Input alphabet (Z)

`{ next, error, complete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(boolean), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Passing next → `checking(i+1)`.
- Failing next → `stopped`.
- Complete → `stopped`.

## 6. Output function (G : S × Z → A*)

- Failing next → `next(false) · complete`.
- Passing next → `ε`.
- Complete → `next(true) · complete`.

## Worked trace

`every(x => x < 3)` on `1 2 3` writes `next(false) complete` at 3 and unsubscribes.

## Why this is Mealy rather than Moore

Failing input writes false; complete input writes true. Same checking family, different input, different letter.

## Edge cases fixed by the 7.x source

- Empty source is true.
- Unsubscribes on the first failure.

## Source anchors

- `src/internal/operators/every.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
