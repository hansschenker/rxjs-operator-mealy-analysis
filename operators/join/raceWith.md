# `raceWith` — Mealy 6-tuple

| | |
|---|---|
| Category | Join |
| Kind | Pipeable join |
| RxJS 7.x source | `src/internal/operators/raceWith.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `raceWith(...otherSources): OperatorFunction<T, T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Subscribe to the source and the others. The first to emit a next, error, or complete wins. Losers are unsubscribed. The winner is forwarded.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`{ racing, forwarding(winner), stopped }`.

## 2. Initial state (S0)

`racing`.

## 3. Input alphabet (Z)

`{ next_i, error_i, complete_i, unsubscribe }`.

## 4. Output alphabet (A)

`{ next, error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- First signal from i → forwarding(i) or stopped if that signal is terminal.
- Later signals from losers are not delivered.

## 6. Output function (G : S × Z → A*)

- Winning next → next, then later winner notifications copy through.
- Winning error → error.
- Winning complete → complete.

## Worked trace

A synchronous source wins against a later timer and the timer is unsubscribed.

## Why this is Mealy rather than Moore

In `racing`, the input kind selects next, error, or complete, and also selects the winner stored by T.

## Edge cases fixed by the 7.x source

- Subscription order matters for synchronous sources.
- Same machine as creation `race`, with the piped source included.

## Source anchors

- `src/internal/operators/raceWith.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
