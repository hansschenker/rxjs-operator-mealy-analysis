# `observeOn` — Mealy 6-tuple

| | |
|---|---|
| Category | Utility |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/observeOn.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `observeOn(scheduler, delay = 0): MonoTypeOperatorFunction<T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Schedule each next, error, and complete on `scheduler`. Order is preserved by the scheduler queue. Delay shifts each scheduled action.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { queue of scheduled notifications, stopped }`.

## 2. Initial state (S0)

Empty queue.

## 3. Input alphabet (Z)

`{ next, error, complete, scheduledFire, unsubscribe }`.

## 4. Output alphabet (A)

`{ next, error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Input notifications enqueue.
- `scheduledFire` dequeues.
- Unsubscribe clears the queue.

## 6. Output function (G : S × Z → A*)

- Source notification → `ε` at arrival.
- `scheduledFire →` the queued letter.

## Worked trace

`of(1, 2).pipe(observeOn(asyncScheduler))` writes `next(1) next(2) complete` on a later turn, not inside the synchronous subscribe.

## Why this is Mealy rather than Moore

The fire input writes the letter stored from an earlier notification input. Arrival itself writes `ε`.

## Edge cases fixed by the 7.x source

- Unlike `delay`, error is also scheduled.
- Delay 0 still leaves the current stack if the scheduler is async.

## Source anchors

- `src/internal/operators/observeOn.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
