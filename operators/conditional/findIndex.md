# `findIndex` — Mealy 6-tuple

| | |
|---|---|
| Category | Conditional and Boolean |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/findIndex.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `findIndex(predicate, thisArg?): OperatorFunction<T, number>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`findIndex` is a pipeable operator on the RxJS 7.x line. Stable. Emit the index of the first match and complete. If none, emit `-1` and complete. Unsubscribes after a match.

In plain terms, the operator keeps this memory: S = { searching(i), stopped }. At subscription, before any source notification, that memory is searching(0). It reacts to these events: { next, error, complete, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: findIndex(x => x === 3) on 1 2 3 writes next(2) complete.

Details that a marble diagram often leaves out: Index is zero-based. Does not error on a miss.

## Role in the notification machine

Emit the index of the first match and complete. If none, emit `-1` and complete. Unsubscribes after a match.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { searching(i), stopped }`.

## 2. Initial state (S0)

`searching(0)`.

## 3. Input alphabet (Z)

`{ next, error, complete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(number), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Non-match → `searching(i+1)`.
- Match or complete → `stopped`.

## 6. Output function (G : S × Z → A*)

- Match → `next(i) · complete`.
- Complete with no match → `next(-1) · complete`.
- Non-match → `ε`.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: S = { searching(i), stopped }.

Memory at subscribe: searching(0).

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| Non-match | searching(i+1) | next(i) · complete |
| Match or complete | stopped | next(-1) · complete |
| Non-match | named by the output row; memory change is in the transition rows above | ε |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

`findIndex(x => x === 3)` on `1 2 3` writes `next(2) complete`.

## Why this is Mealy rather than Moore

The emitted index is state at the matching input. Miss is a complete-input output.

## Edge cases fixed by the 7.x source

- Index is zero-based.
- Does not error on a miss.

## Source anchors

- `src/internal/operators/findIndex.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
