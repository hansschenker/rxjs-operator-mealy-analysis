# `defaultIfEmpty` — Mealy 6-tuple

| | |
|---|---|
| Category | Conditional and Boolean |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/defaultIfEmpty.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `defaultIfEmpty(defaultValue): OperatorFunction<T, T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`defaultIfEmpty` is a pipeable operator on the RxJS 7.x line. Stable. Mirror the source. If the source completes without any next, emit `defaultValue` and then complete. One flag of memory.

In plain terms, the operator keeps this memory: S = { empty, seen, stopped }. At subscription, before any source notification, that memory is empty. It reacts to these events: { next, error, complete, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: EMPTY.pipe(defaultIfEmpty(0)) writes next(0) complete. of(1).pipe(defaultIfEmpty(0)) writes next(1) complete.

Details that a marble diagram often leaves out: Default is a single value, not an observable. Error does not substitute the default.

## Role in the notification machine

Mirror the source. If the source completes without any next, emit `defaultValue` and then complete. One flag of memory.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { empty, seen, stopped }`.

## 2. Initial state (S0)

`empty`.

## 3. Input alphabet (Z)

`{ next, error, complete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next, error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `empty × next → seen`.
- `seen × next → seen`.
- Complete → `stopped`.

## 6. Output function (G : S × Z → A*)

- `next(v) → next(v)`.
- `complete` in `empty → next(defaultValue) · complete`.
- `complete` in `seen → complete`.

## Worked trace

`EMPTY.pipe(defaultIfEmpty(0))` writes `next(0) complete`. `of(1).pipe(defaultIfEmpty(0))` writes `next(1) complete`.

## Why this is Mealy rather than Moore

Complete writes an extra next only from the empty state. That is the whole operator.

## Edge cases fixed by the 7.x source

- Default is a single value, not an observable.
- Error does not substitute the default.

## Source anchors

- `src/internal/operators/defaultIfEmpty.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
