# Mealy 6-tuple method for RxJS operators

SuperGrok is the main contributor of this project.

An operator is a machine that reads a word of notifications and writes a word of notifications. The Mealy form is used because the letter written at a step depends on the memory **and** the notification that just arrived.

## Tuple

1. **State space `S`.** Memory between notifications: counters, buffers, the active inner, the latest value, the refCount, the retry attempt. Many operators have an infinite state space because they store arbitrary values (`scan`, `distinct`, `groupBy`). The control skeleton is still finite.
2. **Initial state `S0`.** Memory at subscription, before the first source notification. Creation functions start in `idle` and treat `subscribe` as an input.
3. **Input alphabet `Z`.** Source notifications `{ next(v), error(e), complete }`, plus `unsubscribe`. Time operators add `tick`. Higher-order operators add inner notifications. Join operators index the input by source (`next_i`).
4. **Output alphabet `A`.** Downstream notifications. Side effects that matter to the machine (subscribe inner, unsubscribe inner, abort XHR, call `tap`) are named as actions when they are not notifications.
5. **Transition `T : S × Z → S`.** Next memory. `stopped` is absorbing.
6. **Output `G : S × Z → A*`.** A finite word. `ε` means the input was consumed and nothing was forwarded. `next(v) · complete` is a two-letter word (used by `take`, `first`, `materialize`).

## Why not Moore

A Moore machine writes from the state alone. `filter` in a single `active` state must sometimes emit and sometimes not; the decision is the predicate on the input. `debounceTime` stores `lastValue` and writes it only on the timer input or on `complete`, not on the source `next` that stored it. Those are Mealy outputs.

## What was read

- Pipeable operators: `https://github.com/ReactiveX/rxjs/tree/7.x/src/internal/operators`
- Creation functions that are not in that tree: `src/internal/observable/*` and `src/internal/ajax/ajax.ts`
- Tree revision: `e5351d02e225e275ac0e497c7b66eaa5f0c88791`

Behavior that the marble diagrams often skip is taken from the source: `debounceTime` keeps one task and reschedules from `lastTime + dueTime`; it flushes on complete and does not flush on error. `partition` is two `filter` subscriptions, not a shared splitter. `take(0)` completes without subscribing. `share` defaults to resetting on error, complete, and refCount zero.

## Notation

- `⊥` means "no value yet".
- `A*` is the set of finite words over `A`, including `ε`.
- Parameters (`bufferSize`, `concurrent`, `predicate`, `seed`) are constants of the machine, not inputs, unless the operator re-reads them per notification.
