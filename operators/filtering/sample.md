# `sample` — Mealy 6-tuple

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/sample.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `sample(notifier: Observable<any>): MonoTypeOperatorFunction<T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Remember the latest source value. When `notifier` emits, emit that latest value if one arrived since the previous sample, then clear the pending flag. Notifier emissions with no fresh value write `ε`.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { none, fresh(v), stopped }`.

## 2. Initial state (S0)

`none`.

## 3. Input alphabet (Z)

`{ next(v), error, complete, notifierNext, notifierError, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(v), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `next(v) → fresh(v)`.
- `notifierNext` in `fresh` → `none`.
- Source complete → `stopped`.

## 6. Output function (G : S × Z → A*)

- `next → ε`.
- `notifierNext` in `fresh(v) → next(v)`.
- `notifierNext` in `none → ε`.
- Source complete does not flush the fresh value in `sample` (unlike audit).

## Worked trace

Value 1, notifier, value 2, value 3, notifier writes `next(1) next(3)`.

## Why this is Mealy rather than Moore

Notifier next writes `next` or `ε` depending on the fresh flag in state.

## Edge cases fixed by the 7.x source

- No flush on source complete.
- Notifier error is an error output.

## Source anchors

- `src/internal/operators/sample.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
