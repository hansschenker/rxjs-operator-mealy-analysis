# `mergeAll` — Mealy 6-tuple

| | |
|---|---|
| Category | Join |
| Kind | Pipeable higher-order join |
| RxJS 7.x source | `src/internal/operators/mergeAll.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `mergeAll(concurrent = Infinity): OperatorFunction<ObservableInput<T>, T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`mergeAll` is a pipeable higher-order join on the RxJS 7.x line. Stable. Subscribe to inners as the source emits them, up to `concurrent`, and forward their values as they arrive. Extra inners queue.

In plain terms, the operator keeps this memory: S = { active count, queue, outerDone, stopped }. At subscription, before any source notification, that memory is Active 0, empty queue. It reacts to these events: Higher-order alphabet.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: Two delayed inners can interleave their nexts on the output.

Details that a marble diagram often leaves out: `mergeAll(1)` matches `concatAll`. Implemented through `mergeInternals`.

## Role in the notification machine

Subscribe to inners as the source emits them, up to `concurrent`, and forward their values as they arrive. Extra inners queue.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { active count, queue, outerDone, stopped }`.

## 2. Initial state (S0)

Active 0, empty queue.

## 3. Input alphabet (Z)

Higher-order alphabet.

## 4. Output alphabet (A)

`{ next, error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Start inners up to concurrency.
- `innerComplete` pulls the queue.
- All done → `stopped`.

## 6. Output function (G : S × Z → A*)

- `innerNext → next`.
- Completion word only when active and queue are empty and outer is done.

## Worked trace

Two delayed inners can interleave their nexts on the output.

## Why this is Mealy rather than Moore

Same machine as `mergeMap` with identity project.

## Edge cases fixed by the 7.x source

- `mergeAll(1)` matches `concatAll`.
- Implemented through `mergeInternals`.

## Source anchors

- `src/internal/operators/mergeAll.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
