# `debounceTime` — Mealy 6-tuple

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/debounceTime.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `debounceTime(dueTime, scheduler = asyncScheduler): MonoTypeOperatorFunction<T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Store the latest value and the time it arrived. One scheduled task is armed if none exists. When the task runs, if `now < lastTime + dueTime`, reschedule the remainder; otherwise emit `lastValue` and clear. Source complete flushes the pending value then completes. Source error does not flush. Unsubscribe clears `lastValue` and the task.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { idle, holding(lastValue, lastTime, task | null), stopped }`.

## 2. Initial state (S0)

`idle` (`activeTask = null`, `lastValue = null`).

## 3. Input alphabet (Z)

`{ next(v), error, complete, taskFire, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(v), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `next(v)` sets `lastValue`, `lastTime`, and arms `activeTask` only if it was null.
- `taskFire` either stays holding and reschedules, or goes `idle` if the due time has elapsed.
- `complete` or `error` or `unsubscribe → stopped`.

## 6. Output function (G : S × Z → A*)

- `next → ε`.
- `taskFire → ε` if rescheduled; `next(lastValue)` if due.
- `complete → next(lastValue) · complete` when a task is active, else `complete`.
- `error(e) → error(e)` with no flush.

## Worked trace

`1` at t=0, `2` at t=40, dueTime 100. The single task fires, sees lastTime 40, reschedules, then writes `next(2)`.

## Why this is Mealy rather than Moore

`taskFire` either writes `ε` or `next(lastValue)` depending on `lastTime` in state and the scheduler now, which arrives with the fire input. 7.x does not tear down the task on every next.

## Edge cases fixed by the 7.x source

- One task, not a reset-and-replace per value.
- Complete flushes; error does not.
- Finalize nulls `lastValue` and `activeTask`.

## Source anchors

- `src/internal/operators/debounceTime.ts`: `emitWhenIdle` compares `lastTime + dueTime` with `scheduler.now()`; complete calls `emit()` then `subscriber.complete()`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
