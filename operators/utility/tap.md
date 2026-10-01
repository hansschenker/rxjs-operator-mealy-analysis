# `tap` — Mealy 6-tuple

| | |
|---|---|
| Category | Utility |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/tap.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `tap(observerOrNext?, error?, complete?): MonoTypeOperatorFunction<T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`tap` is a pipeable operator on the RxJS 7.x line. Stable. Mirror every notification, and also call the matching observer callback. A throw in a callback becomes an error output and stops mirroring. Also known historically as `do`.

In plain terms, the operator keeps this memory: S = { active, stopped }. At subscription, before any source notification, that memory is active. It reacts to these events: { next, error, complete, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: tap(console.log) on of(1) writes next(1) complete and performs the log side effect before each letter.

Details that a marble diagram often leaves out: Unsubscribe can call a finalize-style teardown if an observer with `unsubscribe` is used; 7.x tap supports an observer object. Does not change values.

## Role in the notification machine

Mirror every notification, and also call the matching observer callback. A throw in a callback becomes an error output and stops mirroring. Also known historically as `do`.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { active, stopped }`.

## 2. Initial state (S0)

`active`.

## 3. Input alphabet (Z)

`{ next, error, complete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next, error, complete }` plus side-effect actions.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Next stays active if the callback returns.
- Callback throw → `stopped`.
- Terminal → `stopped` after the callback.

## 6. Output function (G : S × Z → A*)

- `next(v) → next(v)` after `observer.next(v)`.
- `error(e) → error(e)` after the error callback.
- `complete → complete` after the complete callback.
- Callback throw → `error(thrown)` and the original notification is not mirrored.

## Worked trace

`tap(console.log)` on `of(1)` writes `next(1) complete` and performs the log side effect before each letter.

## Why this is Mealy rather than Moore

Side effect is part of `G(active, notification)`. The downstream letter normally equals the input letter, which is still a Mealy output of that input.

## Edge cases fixed by the 7.x source

- Unsubscribe can call a finalize-style teardown if an observer with `unsubscribe` is used; 7.x tap supports an observer object.
- Does not change values.

## Source anchors

- `src/internal/operators/tap.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
