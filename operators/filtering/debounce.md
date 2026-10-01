# `debounce` — Mealy 6-tuple

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/debounce.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `debounce(durationSelector: (value: T) => ObservableInput<any>): MonoTypeOperatorFunction<T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Each source next unsubscribes the previous duration and subscribes to `durationSelector(value)`, storing the value. When that duration emits, emit the stored value. A newer source next cancels the previous duration. Source complete emits the pending value if any, then completes.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { idle, pending(v, duration), stopped }`.

## 2. Initial state (S0)

`idle`.

## 3. Input alphabet (Z)

`{ next(v), error, complete, durationNext, durationError, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(v), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `next(v)` replaces pending duration → `pending(v, newDuration)`.
- `durationNext → idle`.
- Terminal → `stopped`.

## 6. Output function (G : S × Z → A*)

- `next(v) → ε`.
- `durationNext → next(v)` for the value that armed this duration.
- `complete → next(pending) · complete` if pending, else `complete`.

## Worked trace

Values 1, 2, 3 each restarting the duration, then a quiet duration next, writes `next(3)` only.

## Why this is Mealy rather than Moore

The duration-next input emits whatever value is stored. A source next in the same pending family writes `ε` and replaces memory.

## Edge cases fixed by the 7.x source

- Duration selector sees the latest value.
- Complete flushes.

## Source anchors

- `src/internal/operators/debounce.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
