# `throttleTime` — Mealy 6-tuple

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/throttleTime.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `throttleTime(duration, scheduler?, config?): MonoTypeOperatorFunction<T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`throttleTime` is a pipeable operator on the RxJS 7.x line. Stable. `throttle` with a timer duration. Default leading edge: emit, then ignore for `duration`. Config can enable trailing emit at the end of the window.

In plain terms, the operator keeps this memory: S = { idle, throttling(pending | ⊥), stopped }. At subscription, before any source notification, that memory is idle. It reacts to these events: { next(v), error, complete, tick, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: throttleTime(1000) on a burst emits the first value and suppresses the rest until the timer fires.

Details that a marble diagram often leaves out: Default config is leading only. Scheduler defaults to asyncScheduler.

## Role in the notification machine

`throttle` with a timer duration. Default leading edge: emit, then ignore for `duration`. Config can enable trailing emit at the end of the window.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { idle, throttling(pending | ⊥), stopped }`.

## 2. Initial state (S0)

`idle`.

## 3. Input alphabet (Z)

`{ next(v), error, complete, tick, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(v), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `idle × next → throttling`.
- `throttling × next` updates pending if trailing.
- `tick → idle` or re-enters throttling on a trailing emit.

## 6. Output function (G : S × Z → A*)

- `idle × next → next(v)` when leading.
- `tick → next(pending)` when trailing and pending, else `ε`.

## Worked trace

`throttleTime(1000)` on a burst emits the first value and suppresses the rest until the timer fires.

## Why this is Mealy rather than Moore

Timer tick and source next produce different words from the throttling state.

## Edge cases fixed by the 7.x source

- Default config is leading only.
- Scheduler defaults to asyncScheduler.

## Source anchors

- `src/internal/operators/throttleTime.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
