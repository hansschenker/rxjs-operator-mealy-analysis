# `max` — Mealy 6-tuple

| | |
|---|---|
| Category | Mathematical and Aggregate |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/max.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `max(comparer?): OperatorFunction<T, T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`max` is a pipeable operator on the RxJS 7.x line. Stable. Reduce with a greater-than comparer. Emit the max on complete. Empty source errors with `EmptyError`. Comparer defaults to numeric `>`.

In plain terms, the operator keeps this memory: S = { empty, holding(max), stopped }. At subscription, before any source notification, that memory is empty. It reacts to these events: { next, error, complete, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: of(3, 1, 2).pipe(max()) writes next(3) complete.

Details that a marble diagram often leaves out: Empty errors. Comparer throw is an error. Implemented as a reduce.

## Role in the notification machine

Reduce with a greater-than comparer. Emit the max on complete. Empty source errors with `EmptyError`. Comparer defaults to numeric `>`.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { empty, holding(max), stopped }`.

## 2. Initial state (S0)

`empty`.

## 3. Input alphabet (Z)

`{ next, error, complete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next, error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `empty × next(v) → holding(v)`.
- `holding × next(v) → holding(v)` if v wins the comparer, else unchanged.
- Complete → `stopped`.

## 6. Output function (G : S × Z → A*)

- `next → ε`.
- `complete` in `holding → next(max) · complete`.
- `complete` in `empty → error(EmptyError)`.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: S = { empty, holding(max), stopped }.

Memory at subscribe: empty.

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| empty × next(v) | holding(v) | ε |
| holding × next(v) | holding(v) if v wins the comparer, else unchanged | next(max) · complete |
| Complete | stopped | error(EmptyError) |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

`of(3, 1, 2).pipe(max())` writes `next(3) complete`.

## Why this is Mealy rather than Moore

Complete writes state. Next updates state only if the input wins the comparison.

## Edge cases fixed by the 7.x source

- Empty errors.
- Comparer throw is an error.
- Implemented as a reduce.

## Source anchors

- `src/internal/operators/max.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
