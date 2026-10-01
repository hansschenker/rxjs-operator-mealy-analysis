# `skipWhile` — Mealy 6-tuple

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/skipWhile.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `skipWhile(predicate: (value, index) => boolean): MonoTypeOperatorFunction<T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`skipWhile` is a pipeable operator on the RxJS 7.x line. Stable. Drop values while `predicate` is true. The first value that fails the predicate, and everything after it, is emitted. The predicate is not consulted again after that.

In plain terms, the operator keeps this memory: S = { skipping(i), forwarding, stopped }. At subscription, before any source notification, that memory is skipping(0). It reacts to these events: { next(v), error, complete, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: skipWhile(x => x < 3) on 1 2 3 1 writes next(3) next(1) complete. The later 1 passes.

Details that a marble diagram often leaves out: Index increments only while skipping. Once forwarding, predicate throws cannot happen because it is not called.

## Role in the notification machine

Drop values while `predicate` is true. The first value that fails the predicate, and everything after it, is emitted. The predicate is not consulted again after that.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { skipping(i), forwarding, stopped }`.

## 2. Initial state (S0)

`skipping(0)`.

## 3. Input alphabet (Z)

`{ next(v), error, complete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(v), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `skipping × next → skipping(i+1)` if predicate true.
- `skipping × next → forwarding` if predicate false.
- Terminal → `stopped`.

## 6. Output function (G : S × Z → A*)

- Skipped next → `ε`.
- The failing next and later nexts → `next(v)`.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: S = { skipping(i), forwarding, stopped }.

Memory at subscribe: skipping(0).

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| skipping × next | skipping(i+1) if predicate true | ε |
| skipping × next | forwarding if predicate false | next(v) |
| Terminal | stopped | nothing named on a separate output row |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

`skipWhile(x => x < 3)` on `1 2 3 1` writes `next(3) next(1) complete`. The later 1 passes.

## Why this is Mealy rather than Moore

Predicate on the input flips state and also decides whether that same input is in the output word.

## Edge cases fixed by the 7.x source

- Index increments only while skipping.
- Once forwarding, predicate throws cannot happen because it is not called.

## Source anchors

- `src/internal/operators/skipWhile.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
