# `timestamp` — Mealy 6-tuple

| | |
|---|---|
| Category | Utility |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/timestamp.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `timestamp(timestampProvider = dateTimestampProvider): OperatorFunction<T, Timestamp<T>>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`timestamp` is a pipeable operator on the RxJS 7.x line. Stable. Emit `{ value, timestamp }` using the provider's now. No memory between values.

In plain terms, the operator keeps this memory: S = { active, stopped }. At subscription, before any source notification, that memory is active. It reacts to these events: { next(v), error, complete, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: Each click becomes a timestamped value. The timestamp is the arrival time, not a delta.

Details that a marble diagram often leaves out: Provider is injectable so tests can fake time. Does not delay.

## Role in the notification machine

Emit `{ value, timestamp }` using the provider's now. No memory between values.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { active, stopped }`.

## 2. Initial state (S0)

`active`.

## 3. Input alphabet (Z)

`{ next(v), error, complete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(Timestamp), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Next stays active.
- Terminal → `stopped`.

## 6. Output function (G : S × Z → A*)

- `next(v) → next({ value: v, timestamp: now })`.
- Error and complete copy through.

## Worked trace

Each click becomes a timestamped value. The timestamp is the arrival time, not a delta.

## Why this is Mealy rather than Moore

The letter is a function of the input value and the clock read while handling that input. Little state is required.

## Edge cases fixed by the 7.x source

- Provider is injectable so tests can fake time.
- Does not delay.

## Source anchors

- `src/internal/operators/timestamp.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
