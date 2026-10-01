# `zipWith` — Mealy 6-tuple

| | |
|---|---|
| Category | Join |
| Kind | Pipeable join |
| RxJS 7.x source | `src/internal/operators/zipWith.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `zipWith(...otherSources): OperatorFunction<T, any[]>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`zipWith` is a pipeable join on the RxJS 7.x line. Stable. Zip the piped source with the other sources by index. Each side has a queue. Emit when every queue is non-empty, then shift one from each. Complete when any side completes and cannot form another tuple.

In plain terms, the operator keeps this memory: Queue per source plus done flags, or stopped. At subscription, before any source notification, that memory is Empty queues. It reacts to these events: { next_i, error_i, complete_i, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: of(1, 2).pipe(zipWith(of('a'))) writes next([1, 'a']) complete.

Details that a marble diagram often leaves out: Values are consumed, not reused as in combineLatest. Same pairing rule as creation `zip`.

## Role in the notification machine

Zip the piped source with the other sources by index. Each side has a queue. Emit when every queue is non-empty, then shift one from each. Complete when any side completes and cannot form another tuple.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

Queue per source plus done flags, or stopped.

## 2. Initial state (S0)

Empty queues.

## 3. Input alphabet (Z)

`{ next_i, error_i, complete_i, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(tuple), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Next appends. Shift a row when all queues are non-empty.
- Complete on an empty queue → stopped.
- Error → stopped.

## 6. Output function (G : S × Z → A*)

- Next → next(tuple) if the append filled the last hole, else ε.
- Complete → complete if no further row is possible, else ε.
- Error → error.

## Worked trace

`of(1, 2).pipe(zipWith(of('a')))` writes `next([1, 'a']) complete`.

## Why this is Mealy rather than Moore

The output letter is the shifted row, which exists only because state queues plus this input formed a full tuple.

## Edge cases fixed by the 7.x source

- Values are consumed, not reused as in combineLatest.
- Same pairing rule as creation `zip`.

## Source anchors

- `src/internal/operators/zipWith.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
