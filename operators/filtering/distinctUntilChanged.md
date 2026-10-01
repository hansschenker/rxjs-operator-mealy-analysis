# `distinctUntilChanged` — Mealy 6-tuple

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/distinctUntilChanged.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `distinctUntilChanged(comparator?, keySelector?): MonoTypeOperatorFunction<T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`distinctUntilChanged` is a pipeable operator on the RxJS 7.x line. Stable. Emit when the current key is not equal to the previous key. Comparator defaults to `===`. Only the last key is stored, not the full history.

In plain terms, the operator keeps this memory: S = { empty, holding(key), stopped }. At subscription, before any source notification, that memory is empty. It reacts to these events: { next(v), error, complete, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: 1, 1, 2, 2, 1 writes next(1) next(2) next(1). The last 1 passes because it differs from 2.

Details that a marble diagram often leaves out: First value always passes. Comparator receives keys after `keySelector`.

## Role in the notification machine

Emit when the current key is not equal to the previous key. Comparator defaults to `===`. Only the last key is stored, not the full history.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { empty, holding(key), stopped }`.

## 2. Initial state (S0)

`empty`.

## 3. Input alphabet (Z)

`{ next(v), error, complete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(v), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `empty × next(v) → holding(key(v))`.
- `holding(k) × next(v) → holding(key(v))` whether or not it emits.
- Terminal → `stopped`.

## 6. Output function (G : S × Z → A*)

- `empty × next(v) → next(v)`.
- `holding(k) × next(v) → next(v)` if `compare(k, key(v))` is false, else `ε`.
- Throw in comparator or key selector → `error`.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: S = { empty, holding(key), stopped }.

Memory at subscribe: empty.

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| empty × next(v) | holding(key(v)) | next(v) |
| holding(k) × next(v) | holding(key(v)) whether or not it emits | next(v) if compare(k, key(v)) is false, else ε |
| Terminal | stopped | error |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

`1, 1, 2, 2, 1` writes `next(1) next(2) next(1)`. The last 1 passes because it differs from 2.

## Why this is Mealy rather than Moore

Comparison reads state and input. Equal inputs write `ε`; unequal inputs write `next`. Same state machine as a change detector.

## Edge cases fixed by the 7.x source

- First value always passes.
- Comparator receives keys after `keySelector`.

## Source anchors

- `src/internal/operators/distinctUntilChanged.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
