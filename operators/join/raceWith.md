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

## Explanation

`raceWith` is a pipeable join on the RxJS 7.x line. Stable. Subscribe to the source and the others. The first to emit a next, error, or complete wins. Losers are unsubscribed. The winner is forwarded.

In plain terms, the operator keeps this memory: { racing, forwarding(winner), stopped }. At subscription, before any source notification, that memory is racing. It reacts to these events: { next_i, error_i, complete_i, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: A synchronous source wins against a later timer and the timer is unsubscribed.

Details that a marble diagram often leaves out: Subscription order matters for synchronous sources. Same machine as creation `race`, with the piped source included.

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

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: { racing, forwarding(winner), stopped }.

Memory at subscribe: racing.

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| First signal from i | forwarding(i) or stopped if that signal is terminal | next, then later winner notifications copy through |
| Later signals from losers are not delivered | Later signals from losers are not delivered | error |
| Winning complete | named by the output row; memory change is in the transition rows above | complete |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

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
