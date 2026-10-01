# `mapTo` — Mealy 6-tuple

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/mapTo.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `mapTo(value: R): OperatorFunction<T, R>` |
| Status on the 7.x line | Deprecated in 7.x. Use `map(() => value)`. |

SuperGrok is the main contributor of this analysis.

## Explanation

`mapTo` is a pipeable operator on the RxJS 7.x line. Deprecated in 7.x. Use `map(() => value)`. Replace every source next with the same constant. No index. Error and complete pass through.

In plain terms, the operator keeps this memory: S = { active, stopped }. At subscription, before any source notification, that memory is active. It reacts to these events: { next(_), error, complete, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: of(1, 2, 3).pipe(mapTo('x')) writes next('x') next('x') next('x') complete.

Details that a marble diagram often leaves out: The constant is captured when `mapTo` is called, not per next. Deprecated, tuple unchanged.

## Role in the notification machine

Replace every source next with the same constant. No index. Error and complete pass through.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { active, stopped }`.

## 2. Initial state (S0)

`active`.

## 3. Input alphabet (Z)

`{ next(_), error, complete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(constant), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `active × next → active`.
- Terminal → `stopped`.

## 6. Output function (G : S × Z → A*)

- `active × next(_) → next(constant)`.
- `error` and `complete` copy through.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: S = { active, stopped }.

Memory at subscribe: active.

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| active × next | active | next(constant) |
| Terminal | stopped | error and complete copy through |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

`of(1, 2, 3).pipe(mapTo('x'))` writes `next('x') next('x') next('x') complete`.

## Why this is Mealy rather than Moore

Even though the letter is constant, it is written because a next input arrived. `complete` in the same state writes a different letter.

## Edge cases fixed by the 7.x source

- The constant is captured when `mapTo` is called, not per next.
- Deprecated, tuple unchanged.

## Source anchors

- `src/internal/operators/mapTo.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
