# `audit` — Mealy 6-tuple

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/audit.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `audit(durationSelector: (value: T) => ObservableInput<any>): MonoTypeOperatorFunction<T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

On a source next, if no duration is open, subscribe to `durationSelector(value)` and remember the latest value. Further source nexts update the remembered value but do not restart the duration. When the duration emits, emit the latest value and become idle. Duration complete without a next also ends the audit window in 7.x and can emit. Source complete emits any pending value then completes.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { idle, auditing(latest), stopped }`.

## 2. Initial state (S0)

`idle`.

## 3. Input alphabet (Z)

`{ next(v), error, complete, durationNext, durationComplete, durationError, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(v), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `idle × next(v) → auditing(v)`.
- `auditing × next(v) → auditing(v)` (duration unchanged).
- `auditing × durationNext|durationComplete → idle`.
- Source complete or error → `stopped`.

## 6. Output function (G : S × Z → A*)

- `idle × next → ε` (starts duration).
- `auditing × next → ε`.
- `durationNext → next(latest)`.
- `complete → next(latest) · complete` if auditing, else `complete`.

## Worked trace

Values 1 then 2 while the duration is open, then duration next, writes `next(2)` once.

## Why this is Mealy rather than Moore

Duration input writes the latest state. Source next only updates state. Same auditing state, different inputs, different words.

## Edge cases fixed by the 7.x source

- Trailing value is flushed on source complete.
- Duration error errors the output.

## Source anchors

- `src/internal/operators/audit.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
