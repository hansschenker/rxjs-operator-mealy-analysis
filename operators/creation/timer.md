# `timer` — Mealy 6-tuple

| | |
|---|---|
| Category | Creation |
| Kind | Cold scheduled producer |
| RxJS 7.x source | `src/internal/observable/timer.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `timer(due: number | Date, intervalOrScheduler?, scheduler?: SchedulerLike): Observable<number>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`timer` is a cold scheduled producer on the RxJS 7.x line. Stable. Subscribe schedules the first emission at `due`. That emission is `next(0)`. If no period is given, it then completes. If a period is given, it continues as `interval`, emitting 1, 2, ... . A `Date` due time is absolute.

In plain terms, the operator keeps this memory: S = { idle, waitingDue(n), ticking(n), stopped }. At subscription, before any source notification, that memory is idle. It reacts to these events: { subscribe, dueTick, periodTick, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: timer(500) writes next(0) complete. timer(500, 1000) writes next(0) then next(1), next(2), ... until unsubscribe.

Details that a marble diagram often leaves out: `interval` is `timer(period, period)`. Due time `0` still goes through the scheduler; it is not a synchronous `of(0)`.

## Role in the notification machine

Subscribe schedules the first emission at `due`. That emission is `next(0)`. If no period is given, it then completes. If a period is given, it continues as `interval`, emitting 1, 2, ... . A `Date` due time is absolute.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { idle, waitingDue(n), ticking(n), stopped }`.

## 2. Initial state (S0)

`idle`.

## 3. Input alphabet (Z)

`{ subscribe, dueTick, periodTick, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(n), complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `idle × subscribe → waitingDue(0)`.
- `waitingDue(0) × dueTick → stopped` if no period.
- `waitingDue(0) × dueTick → ticking(1)` if a period is set.
- `ticking(n) × periodTick → ticking(n+1)`.
- `unsubscribe → stopped`.

## 6. Output function (G : S × Z → A*)

- `dueTick → next(0)` and, with no period, `· complete`.
- `periodTick` in `ticking(n) → next(n)`.

## Worked trace

`timer(500)` writes `next(0) complete`. `timer(500, 1000)` writes `next(0)` then `next(1)`, `next(2)`, ... until unsubscribe.

## Why this is Mealy rather than Moore

`dueTick` in `waitingDue` may write `next·complete` or only `next`, depending on whether period is part of the machine parameters and the input is the due tick rather than a period tick.

## Edge cases fixed by the 7.x source

- `interval` is `timer(period, period)`.
- Due time `0` still goes through the scheduler; it is not a synchronous `of(0)`.

## Source anchors

- `src/internal/observable/timer.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
