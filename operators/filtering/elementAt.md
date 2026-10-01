# `elementAt` — Mealy 6-tuple

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/elementAt.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `elementAt(index: number, defaultValue?): OperatorFunction<T, T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`elementAt` is a pipeable operator on the RxJS 7.x line. Stable. Emit the value at the given zero-based index and complete. If the source completes before that index, emit `defaultValue` if supplied, otherwise error with `ArgumentOutOfRangeError`.

In plain terms, the operator keeps this memory: S = { counting(i), stopped } for 0 ≤ i ≤ index. At subscription, before any source notification, that memory is counting(0). It reacts to these events: { next(v), error, complete, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: elementAt(1) on a b c writes next(b) complete and unsubscribes.

Details that a marble diagram often leaves out: Negative index errors. Unsubscribes after the match so later source values are not pulled.

## Role in the notification machine

Emit the value at the given zero-based index and complete. If the source completes before that index, emit `defaultValue` if supplied, otherwise error with `ArgumentOutOfRangeError`.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { counting(i), stopped }` for `0 ≤ i ≤ index`.

## 2. Initial state (S0)

`counting(0)`.

## 3. Input alphabet (Z)

`{ next(v), error, complete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(v), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `counting(i) × next → counting(i+1)` if `i < index`.
- `counting(index) × next → stopped`.
- Early complete → `stopped`.

## 6. Output function (G : S × Z → A*)

- `counting(index) × next(v) → next(v) · complete`.
- Earlier next → `ε`.
- Early complete → `next(default) · complete` if default given, else `error(ArgumentOutOfRangeError)`.

## Worked trace

`elementAt(1)` on `a b c` writes `next(b) complete` and unsubscribes.

## Why this is Mealy rather than Moore

Only the next that arrives while `i = index` writes a value. Earlier nexts write `ε`. Classic count Mealy.

## Edge cases fixed by the 7.x source

- Negative index errors.
- Unsubscribes after the match so later source values are not pulled.

## Source anchors

- `src/internal/operators/elementAt.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
