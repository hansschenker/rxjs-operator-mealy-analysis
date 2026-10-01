# `mergeScan` — Mealy 6-tuple

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable higher-order accumulator |
| RxJS 7.x source | `src/internal/operators/mergeScan.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `mergeScan(accumulator, seed, concurrent = Infinity): OperatorFunction<T, R>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`mergeScan` is a pipeable higher-order accumulator on the RxJS 7.x line. Stable. Like `scan`, but the accumulator returns an observable. Seed is the initial acc. Each outer value starts `accumulator(acc, value)` and the inner's emissions are both outputs and the latest acc. Concurrency defaults to Infinity. Complete when outer is done and inners are idle.

In plain terms, the operator keeps this memory: S = { acc, active, queue, outerDone, stopped }. At subscription, before any source notification, that memory is acc = seed, active 0, empty queue. It reacts to these events: Higher-order alphabet.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: of(1, 1).pipe(mergeScan((acc, v) => of(acc + v), 0)) writes next(1) next(2) complete when serialized by the accumulator dependency.

Details that a marble diagram often leaves out: High concurrency can start accumulators with a stale acc if the implementation does not serialize the seed read. 7.x `mergeScan` uses `mergeInternals` and passes the latest acc when the inner is subscribed; overlapping inners can still interleave emissions. Seed is required.

## Role in the notification machine

Like `scan`, but the accumulator returns an observable. Seed is the initial acc. Each outer value starts `accumulator(acc, value)` and the inner's emissions are both outputs and the latest acc. Concurrency defaults to Infinity. Complete when outer is done and inners are idle.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { acc, active, queue, outerDone, stopped }`.

## 2. Initial state (S0)

`acc = seed`, active 0, empty queue.

## 3. Input alphabet (Z)

Higher-order alphabet.

## 4. Output alphabet (A)

`{ next(acc'), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `outerNext` queues work that closes over the acc at start time according to `mergeInternals` scan semantics.
- Inner next replaces `acc`.
- Idle and outer done → `stopped`.

## 6. Output function (G : S × Z → A*)

- `innerNext(r) → next(r)` and `acc := r`.
- Seed is not emitted by itself.
- Errors copy through.

## Worked trace

`of(1, 1).pipe(mergeScan((acc, v) => of(acc + v), 0))` writes `next(1) next(2) complete` when serialized by the accumulator dependency.

## Why this is Mealy rather than Moore

Inner next both emits and updates `acc`. The next accumulator call reads that state. Output letter is the inner input, next state is derived from it.

## Edge cases fixed by the 7.x source

- High concurrency can start accumulators with a stale acc if the implementation does not serialize the seed read. 7.x `mergeScan` uses `mergeInternals` and passes the latest acc when the inner is subscribed; overlapping inners can still interleave emissions.
- Seed is required.

## Source anchors

- `src/internal/operators/mergeScan.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
