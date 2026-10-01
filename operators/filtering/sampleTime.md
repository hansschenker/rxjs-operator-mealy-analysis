# `sampleTime` — Mealy 6-tuple

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/sampleTime.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `sampleTime(period, scheduler = asyncScheduler): MonoTypeOperatorFunction<T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`sampleTime` is a pipeable operator on the RxJS 7.x line. Stable. `sample` driven by a periodic scheduler instead of a notifier. Emits the latest value seen during the period, if any.

In plain terms, the operator keeps this memory: S = { none, fresh(v), stopped }. At subscription, before any source notification, that memory is none. It reacts to these events: { next(v), error, complete, tick, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: Values inside a period collapse to one next on the tick, the last of them.

Details that a marble diagram often leaves out: Period is scheduler time, not source count. Empty periods emit nothing.

## Role in the notification machine

`sample` driven by a periodic scheduler instead of a notifier. Emits the latest value seen during the period, if any.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { none, fresh(v), stopped }`.

## 2. Initial state (S0)

`none`.

## 3. Input alphabet (Z)

`{ next(v), error, complete, tick, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(v), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `next → fresh`.
- `tick` clears fresh.
- Terminal → `stopped`.

## 6. Output function (G : S × Z → A*)

- `tick` in `fresh(v) → next(v)`, else `ε`.
- No flush on complete.

## Worked trace

Values inside a period collapse to one next on the tick, the last of them.

## Why this is Mealy rather than Moore

Tick is the emitting input; source next only updates state.

## Edge cases fixed by the 7.x source

- Period is scheduler time, not source count.
- Empty periods emit nothing.

## Source anchors

- `src/internal/operators/sampleTime.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
