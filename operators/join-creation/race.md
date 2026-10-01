# `race` — Mealy 6-tuple

| | |
|---|---|
| Category | Join creation |
| Kind | Join creation function |
| RxJS 7.x source | `src/internal/observable/race.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `race(...sources): Observable<T>` |
| Status on the 7.x line | Stable. Pipeable cousin is `raceWith`. |

SuperGrok is the main contributor of this analysis.

## Explanation

`race` is a join creation function on the RxJS 7.x line. Stable. Pipeable cousin is `raceWith`. Subscribe to all sources. The first source to emit a next (or to terminate, in the 7.x race implementation the first notifier wins) becomes the winner; the others are unsubscribed. Forward the winner until it terminates.

In plain terms, the operator keeps this memory: S = { racing, forwarding(winner), stopped }. At subscription, before any source notification, that memory is racing after subscribe. It reacts to these events: { subscribe, next_i, error_i, complete_i, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: race(slow$, fastOf(1)) writes next(1) complete and unsubscribes slow$ as soon as 1 arrives.

Details that a marble diagram often leaves out: Empty race completes. Synchronous first source wins before later sources are fully armed only according to subscription order; a sync source earlier in the list wins.

## Role in the notification machine

Subscribe to all sources. The first source to emit a next (or to terminate, in the 7.x race implementation the first notifier wins) becomes the winner; the others are unsubscribed. Forward the winner until it terminates.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { racing, forwarding(winner), stopped }`.

## 2. Initial state (S0)

`racing` after subscribe.

## 3. Input alphabet (Z)

`{ subscribe, next_i, error_i, complete_i, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(v), error(e), complete }` plus the action of unsubscribing losers.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `racing × next_i → forwarding(i)` and drop other subscriptions.
- `racing × error_i → stopped` if that error is the first signal (7.x: first notification wins, including error/complete).
- `racing × complete_i → stopped` if complete wins the race.
- `forwarding(i)` copies i's transitions; signals from losers are not in Z anymore.

## 6. Output function (G : S × Z → A*)

- `racing × next_i(v) → next(v)`.
- `racing × error_i(e) → error(e)`.
- `racing × complete_i → complete`.
- Later winner notifications are forwarded.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: S = { racing, forwarding(winner), stopped }.

Memory at subscribe: racing after subscribe.

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| racing × next_i | forwarding(i) and drop other subscriptions | next(v) |
| racing × error_i | stopped if that error is the first signal (7.x: first notification wins, including error/complete) | error(e) |
| racing × complete_i | stopped if complete wins the race | complete |
| forwarding(i) copies i's transitions; signals from losers are not in Z anymore | forwarding(i) copies i's transitions; signals from losers are not in Z anymore | Later winner notifications are forwarded |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

`race(slow$, fastOf(1))` writes `next(1) complete` and unsubscribes `slow$` as soon as `1` arrives.

## Why this is Mealy rather than Moore

In `racing`, the same control state produces a forward-next, an error, or a complete depending on which input wins. Winner identity is then stored.

## Edge cases fixed by the 7.x source

- Empty race completes.
- Synchronous first source wins before later sources are fully armed only according to subscription order; a sync source earlier in the list wins.

## Source anchors

- `src/internal/observable/race.ts` and `src/internal/operators/raceWith.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
