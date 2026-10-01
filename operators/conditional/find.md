# `find` — Mealy 6-tuple

| | |
|---|---|
| Category | Conditional and Boolean |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/find.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `find(predicate, thisArg?): OperatorFunction<T, T | undefined>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`find` is a pipeable operator on the RxJS 7.x line. Stable. Emit the first matching value and complete. If the source completes with no match, emit `undefined` and complete. Does not error on a miss. Unsubscribes after a match.

In plain terms, the operator keeps this memory: S = { searching(i), stopped }. At subscription, before any source notification, that memory is searching(0). It reacts to these events: { next, error, complete, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: find(x => x > 10) on 1 2 3 writes next(undefined) complete.

Details that a marble diagram often leaves out: Unlike `first`, a miss is `undefined`, not `EmptyError`. Index is passed to the predicate.

## Role in the notification machine

Emit the first matching value and complete. If the source completes with no match, emit `undefined` and complete. Does not error on a miss. Unsubscribes after a match.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { searching(i), stopped }`.

## 2. Initial state (S0)

`searching(0)`.

## 3. Input alphabet (Z)

`{ next, error, complete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(T | undefined), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Non-match increments i.
- Match → `stopped`.
- Complete → `stopped`.

## 6. Output function (G : S × Z → A*)

- Match → `next(v) · complete`.
- Non-match → `ε`.
- Complete → `next(undefined) · complete`.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: S = { searching(i), stopped }.

Memory at subscribe: searching(0).

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| Non-match increments i | Non-match increments i | next(v) · complete |
| Match | stopped | ε |
| Complete | stopped | next(undefined) · complete |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

`find(x => x > 10)` on `1 2 3` writes `next(undefined) complete`.

## Why this is Mealy rather than Moore

Complete input writes undefined because state is still searching. A matching next writes the value instead.

## Edge cases fixed by the 7.x source

- Unlike `first`, a miss is `undefined`, not `EmptyError`.
- Index is passed to the predicate.

## Source anchors

- `src/internal/operators/find.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
