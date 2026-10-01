# `skipLast` — Mealy 6-tuple

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/skipLast.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `skipLast(count: number): MonoTypeOperatorFunction<T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`skipLast` is a pipeable operator on the RxJS 7.x line. Stable. Hold a ring buffer of `count` values. Once the buffer is full, each new value emits the oldest and pushes the new one. The last `count` values are never emitted. Complete does not flush them.

In plain terms, the operator keeps this memory: S = { buffer (queue of length ≤ count), stopped }. At subscription, before any source notification, that memory is Empty queue. It reacts to these events: { next(v), error, complete, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: skipLast(2) on 1 2 3 4 5 writes next(1) next(2) next(3) complete. 4 and 5 stay buffered.

Details that a marble diagram often leaves out: `count <= 0` forwards everything. Must observe complete or unsubscribe to know the tail; it cannot emit the tail earlier.

## Role in the notification machine

Hold a ring buffer of `count` values. Once the buffer is full, each new value emits the oldest and pushes the new one. The last `count` values are never emitted. Complete does not flush them.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { buffer (queue of length ≤ count), stopped }`.

## 2. Initial state (S0)

Empty queue.

## 3. Input alphabet (Z)

`{ next(v), error, complete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(v), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `next(v)` enqueues. If length would exceed count, dequeue.
- Complete → `stopped` without flushing.

## 6. Output function (G : S × Z → A*)

- `next(v) → next(oldest)` if the buffer was already full, else `ε`.
- `complete → complete`.

## Worked trace

`skipLast(2)` on `1 2 3 4 5` writes `next(1) next(2) next(3) complete`. 4 and 5 stay buffered.

## Why this is Mealy rather than Moore

A next writes the value that falls out of the buffer, which is state, not the input value itself.

## Edge cases fixed by the 7.x source

- `count <= 0` forwards everything.
- Must observe complete or unsubscribe to know the tail; it cannot emit the tail earlier.

## Source anchors

- `src/internal/operators/skipLast.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
