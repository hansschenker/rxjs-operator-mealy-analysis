# `startWith` — Mealy 6-tuple

| | |
|---|---|
| Category | Join |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/startWith.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `startWith(...values, scheduler?): MonoTypeOperatorFunction<T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`startWith` is a pipeable operator on the RxJS 7.x line. Stable. On subscribe, emit the given values (via `concat(of(...values), source)` semantics) and then subscribe to the source and forward it. Scheduler shifts the prefix.

In plain terms, the operator keeps this memory: S = { prefix(i), forwarding, stopped }. At subscription, before any source notification, that memory is prefix(0). It reacts to these events: { subscribe, step, sourceNext, sourceError, sourceComplete, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: of(2, 3).pipe(startWith(1)) writes next(1) next(2) next(3) complete.

Details that a marble diagram often leaves out: Values are emitted even if the source never emits. A scheduler argument is not a prefix value.

## Role in the notification machine

On subscribe, emit the given values (via `concat(of(...values), source)` semantics) and then subscribe to the source and forward it. Scheduler shifts the prefix.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { prefix(i), forwarding, stopped }`.

## 2. Initial state (S0)

`prefix(0)`.

## 3. Input alphabet (Z)

`{ subscribe, step, sourceNext, sourceError, sourceComplete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(v), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Prefix steps advance `i`.
- After the last prefix value, enter `forwarding` and subscribe to source.
- Source terminal → `stopped`.

## 6. Output function (G : S × Z → A*)

- Prefix step → `next(values[i])`.
- Source notifications copy through after the prefix.

## Worked trace

`of(2, 3).pipe(startWith(1))` writes `next(1) next(2) next(3) complete`.

## Why this is Mealy rather than Moore

Prefix letters are `G(prefix(i), step)`. Source letters are a different input after the state flips.

## Edge cases fixed by the 7.x source

- Values are emitted even if the source never emits.
- A scheduler argument is not a prefix value.

## Source anchors

- `src/internal/operators/startWith.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
