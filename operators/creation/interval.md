# `interval` — Mealy 6-tuple

| | |
|---|---|
| Category | Creation |
| Kind | Cold scheduled producer |
| RxJS 7.x source | `src/internal/observable/interval.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `interval(period = 0, scheduler = asyncScheduler): Observable<number>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`interval` is a cold scheduled producer on the RxJS 7.x line. Stable. Subscribe schedules a periodic action. The n-th tick writes `next(n)` starting at 0. There is no complete. Unsubscribe cancels the schedule.

In plain terms, the operator keeps this memory: S = { idle, ticking(n), stopped } with n ∈ ℕ. Infinite state space, finite control. At subscription, before any source notification, that memory is idle. It reacts to these events: { subscribe, tick, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: interval(1000) over three ticks writes next(0) next(1) next(2) and remains in ticking(3) until unsubscribe.

Details that a marble diagram often leaves out: `period <= 0` still schedules; it does not emit synchronously in a loop on the async scheduler. Each subscriber has a private counter. `interval` is cold.

## Role in the notification machine

Subscribe schedules a periodic action. The n-th tick writes `next(n)` starting at 0. There is no complete. Unsubscribe cancels the schedule.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { idle, ticking(n), stopped }` with `n ∈ ℕ`. Infinite state space, finite control.

## 2. Initial state (S0)

`idle`.

## 3. Input alphabet (Z)

`{ subscribe, tick, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(n) }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `idle × subscribe → ticking(0)` (first tick armed).
- `ticking(n) × tick → ticking(n+1)`.
- `ticking × unsubscribe → stopped`.

## 6. Output function (G : S × Z → A*)

- `ticking(n) × tick → next(n)`, then the index advances.
- `subscribe` and `unsubscribe` write `ε`.

## Worked trace

`interval(1000)` over three ticks writes `next(0) next(1) next(2)` and remains in `ticking(3)` until unsubscribe.

## Why this is Mealy rather than Moore

The tick input is the same symbol every time; the emitted number comes from the state. Output depends on `(ticking(n), tick)`.

## Edge cases fixed by the 7.x source

- `period <= 0` still schedules; it does not emit synchronously in a loop on the async scheduler.
- Each subscriber has a private counter. `interval` is cold.

## Source anchors

- `src/internal/observable/interval.ts` delegates to `timer(period, period, scheduler)`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
