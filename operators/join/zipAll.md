# `zipAll` — Mealy 6-tuple

| | |
|---|---|
| Category | Join |
| Kind | Pipeable higher-order join |
| RxJS 7.x source | `src/internal/operators/zipAll.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `zipAll(project?): OperatorFunction<ObservableInput<T>, T[]>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`zipAll` is a pipeable higher-order join on the RxJS 7.x line. Stable. Collect inners from the source until it completes, then zip them by index. Emit a tuple whenever every inner can contribute one unused value. Complete when a tuple can no longer be formed.

In plain terms, the operator keeps this memory: Collected inners, a queue per inner, outerDone, stopped. At subscription, before any source notification, that memory is No inners. It reacts to these events: Higher-order alphabet plus inner next/error/complete.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: Source of(of(1, 2), of('a')) then zipAll writes next([1, 'a']) complete. The leftover 2 is dropped.

Details that a marble diagram often leaves out: No inners: complete without a next. Project form is deprecated style.

## Role in the notification machine

Collect inners from the source until it completes, then zip them by index. Emit a tuple whenever every inner can contribute one unused value. Complete when a tuple can no longer be formed.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

Collected inners, a queue per inner, outerDone, stopped.

## 2. Initial state (S0)

No inners.

## 3. Input alphabet (Z)

Higher-order alphabet plus inner next/error/complete.

## 4. Output alphabet (A)

`{ next(tuple), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Outer next appends an inner.
- Outer complete freezes the set and starts pairing.
- Inner next appends to that queue; shift when all queues are non-empty.

## 6. Output function (G : S × Z → A*)

- Inner next → next(tuple) if it completed a row, else ε.
- A complete that leaves a queue empty and done → complete.
- Any error → error.

## Worked trace

Source `of(of(1, 2), of('a'))` then zipAll writes `next([1, 'a']) complete`. The leftover 2 is dropped.

## Why this is Mealy rather than Moore

Emit-or-ε on an inner next depends on the other queues in state. Same rule as creation `zip`, applied after the outer completes collection.

## Edge cases fixed by the 7.x source

- No inners: complete without a next.
- Project form is deprecated style.

## Source anchors

- `src/internal/operators/zipAll.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
