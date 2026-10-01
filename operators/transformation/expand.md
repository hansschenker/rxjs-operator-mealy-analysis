# `expand` — Mealy 6-tuple

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable recursive higher-order operator |
| RxJS 7.x source | `src/internal/operators/expand.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `expand(project, concurrent = Infinity, scheduler?): OperatorFunction<T, T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`expand` is a pipeable recursive higher-order operator on the RxJS 7.x line. Stable. Emit the source value, and also subscribe to `project(value)` whose emissions are emitted and recursively expanded. `concurrent` bounds active inners. It is `mergeMap` with feedback of outputs into the project function. No implicit complete until the source and every recursive inner complete.

In plain terms, the operator keeps this memory: S = { active count, queue of values still to expand, outerDone, stopped }. At subscription, before any source notification, that memory is Active 0, empty queue. It reacts to these events: { outerNext(v), outerError, outerComplete, innerNext(v), innerError, innerComplete, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: of(1).pipe(expand(x => x < 3 ? of(x+1) : EMPTY)) writes next(1) next(2) next(3) complete.

Details that a marble diagram often leaves out: `concurrent: 1` serializes recursion and can change interleaving, not the set of values for a pure project. Scheduler shifts recursive subscribes.

## Role in the notification machine

Emit the source value, and also subscribe to `project(value)` whose emissions are emitted and recursively expanded. `concurrent` bounds active inners. It is `mergeMap` with feedback of outputs into the project function. No implicit complete until the source and every recursive inner complete.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { active count, queue of values still to expand, outerDone, stopped }`.

## 2. Initial state (S0)

Active 0, empty queue.

## 3. Input alphabet (Z)

`{ outerNext(v), outerError, outerComplete, innerNext(v), innerError, innerComplete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(v), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `outerNext` and `innerNext` enqueue an expansion and emit.
- Active expansions are started up to `concurrent`.
- Idle and outer done and empty queue → `stopped`.

## 6. Output function (G : S × Z → A*)

- `outerNext(v) → next(v)`.
- `innerNext(v) → next(v)`.
- Completion word only when nothing remains to expand.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: S = { active count, queue of values still to expand, outerDone, stopped }.

Memory at subscribe: Active 0, empty queue.

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| outerNext and innerNext enqueue an expansion and emit | outerNext and innerNext enqueue an expansion and emit | next(v) |
| Active expansions are started up to concurrent | Active expansions are started up to concurrent | next(v) |
| Idle and outer done and empty queue | stopped | Completion word only when nothing remains to expand |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

`of(1).pipe(expand(x => x < 3 ? of(x+1) : EMPTY))` writes `next(1) next(2) next(3) complete`.

## Why this is Mealy rather than Moore

An inner next both writes `next` and changes the expansion queue. Output and next state are jointly determined by `(queue state, innerNext)`.

## Edge cases fixed by the 7.x source

- `concurrent: 1` serializes recursion and can change interleaving, not the set of values for a pure project.
- Scheduler shifts recursive subscribes.

## Source anchors

- `src/internal/operators/expand.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
