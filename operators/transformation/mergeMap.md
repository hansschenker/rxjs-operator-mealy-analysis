# `mergeMap` — Mealy 6-tuple

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable higher-order operator |
| RxJS 7.x source | `src/internal/operators/mergeMap.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `mergeMap(project, resultSelector?, concurrent = Infinity): OperatorFunction<T, R>` |
| Status on the 7.x line | Stable. `resultSelector` deprecated. Also known as `flatMap`. |

SuperGrok is the main contributor of this analysis.

## Explanation

`mergeMap` is a pipeable higher-order operator on the RxJS 7.x line. Stable. `resultSelector` deprecated. Also known as `flatMap`. Project each outer value to an inner and subscribe immediately up to `concurrent`. Further outers queue. Forward inner nexts as they arrive, interleaved. Complete when outer is done, queue is empty, and no inner is active.

In plain terms, the operator keeps this memory: S = { active count, queue, outerDone, stopped }. At subscription, before any source notification, that memory is Active 0, empty queue, outer not done. It reacts to these events: Higher-order alphabet.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: of(1, 2).pipe(mergeMap(x => of(x, x))) writes four nexts, possibly interleaved if inners were async. With of they subscribe sequentially but both are allowed.

Details that a marble diagram often leaves out: `concurrent: 1` reduces to `concatMap`. `flatMap.ts` is a deprecated alias.

## Role in the notification machine

Project each outer value to an inner and subscribe immediately up to `concurrent`. Further outers queue. Forward inner nexts as they arrive, interleaved. Complete when outer is done, queue is empty, and no inner is active.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { active count, queue, outerDone, stopped }`.

## 2. Initial state (S0)

Active 0, empty queue, outer not done.

## 3. Input alphabet (Z)

Higher-order alphabet.

## 4. Output alphabet (A)

`{ next, error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `outerNext` enqueues and starts inners while `active < concurrent`.
- `innerComplete` decrements active and starts a queued inner.
- Idle, empty queue, outer done → `stopped`.
- Any error → `stopped` and inners unsubscribed.

## 6. Output function (G : S × Z → A*)

- `innerNext → next`.
- `outerNext → ε`.
- `innerComplete → complete` only in the idle-and-outer-done state, else `ε`.

## Worked trace

`of(1, 2).pipe(mergeMap(x => of(x, x)))` writes four nexts, possibly interleaved if inners were async. With `of` they subscribe sequentially but both are allowed.

## Why this is Mealy rather than Moore

The complete-or-not decision on `innerComplete` reads active count and outerDone. Concurrency only changes `T`.

## Edge cases fixed by the 7.x source

- `concurrent: 1` reduces to `concatMap`.
- `flatMap.ts` is a deprecated alias.

## Source anchors

- `src/internal/operators/mergeMap.ts` and `mergeInternals.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
