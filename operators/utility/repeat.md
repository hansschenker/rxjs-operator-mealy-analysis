# `repeat` — Mealy 6-tuple

| | |
|---|---|
| Category | Utility |
| Kind | Pipeable resubscribe |
| RxJS 7.x source | `src/internal/operators/repeat.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `repeat(countOrConfig?): MonoTypeOperatorFunction<T>` |
| Status on the 7.x line | Stable. Config form `{ count, delay }` on 7.x. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Forward the source. On complete, resubscribe if repeats remain. `count` is the number of times the source is subscribed in total in the numeric form used by 7.x docs (repeat(1) means one subscription, no extra repeat). Delay waits before the resubscribe. Error is not repeated; it is forwarded. Infinite count repeats forever.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`{ forwarding(n), waitingDelay, stopped }`.

## 2. Initial state (S0)

`forwarding(1)`.

## 3. Input alphabet (Z)

`{ next, error, complete, delayTick, unsubscribe }`.

## 4. Output alphabet (A)

`{ next, error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Next stays.
- Complete → waitingDelay if another subscription is allowed, else stopped.
- delayTick → forwarding(n+1).
- Error → stopped.

## 6. Output function (G : S × Z → A*)

- Next → next.
- Complete → ε if a repeat will happen, else complete.
- Error → error.
- Resubscribe is an action.

## Worked trace

`of(1).pipe(repeat(2))` writes `next(1) next(1) complete`. The complete of the first subscription is swallowed.

## Why this is Mealy rather than Moore

Complete writes ε or complete depending on the remaining-count state. That is the dual of retry, which does the same on error.

## Edge cases fixed by the 7.x source

- Error does not repeat.
- Delay notifier error becomes the output error.
- count Infinity never writes the final complete.

## Source anchors

- `src/internal/operators/repeat.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
