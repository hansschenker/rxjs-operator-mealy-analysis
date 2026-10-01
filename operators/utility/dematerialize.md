# `dematerialize` — Mealy 6-tuple

| | |
|---|---|
| Category | Utility |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/dematerialize.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `dematerialize(): OperatorFunction<Notification<T>, T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`dematerialize` is a pipeable operator on the RxJS 7.x line. Stable. Inverse of `materialize`. A next-notification becomes `next(value)`. An error-notification becomes `error`. A complete-notification becomes `complete`. The machine stops on those terminal notifications.

In plain terms, the operator keeps this memory: S = { active, stopped }. At subscription, before any source notification, that memory is active. It reacts to these events: { next(Notification), error, complete, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: Materialized next(1), next(complete) dematerializes to next(1) complete.

Details that a marble diagram often leaves out: A real complete on the source completes without synthesizing an extra value. Expects Notification objects; other values are an error in practice.

## Role in the notification machine

Inverse of `materialize`. A next-notification becomes `next(value)`. An error-notification becomes `error`. A complete-notification becomes `complete`. The machine stops on those terminal notifications.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { active, stopped }`.

## 2. Initial state (S0)

`active`.

## 3. Input alphabet (Z)

`{ next(Notification), error, complete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(v), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Next-notification stays active.
- Error-notification or complete-notification → `stopped`.
- A real source error also stops.

## 6. Output function (G : S × Z → A*)

- `next(Notification.createNext(v)) → next(v)`.
- `next(Notification.createError(e)) → error(e)`.
- `next(Notification.createComplete()) → complete`.

## Worked trace

Materialized `next(1), next(complete)` dematerializes to `next(1) complete`.

## Why this is Mealy rather than Moore

The notification kind inside the input selects the output letter. That is input-dependent output from one active state.

## Edge cases fixed by the 7.x source

- A real complete on the source completes without synthesizing an extra value.
- Expects Notification objects; other values are an error in practice.

## Source anchors

- `src/internal/operators/dematerialize.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
