# `distinctUntilKeyChanged` — Mealy 6-tuple

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/distinctUntilKeyChanged.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `distinctUntilKeyChanged(key, compare?): MonoTypeOperatorFunction<T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

`distinctUntilChanged` with `keySelector = value => value[key]`. Same one-key memory.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { empty, holding(keyValue), stopped }`.

## 2. Initial state (S0)

`empty`.

## 3. Input alphabet (Z)

`{ next(v), error, complete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(v), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Same as `distinctUntilChanged`, key is `v[key]`.

## 6. Output function (G : S × Z → A*)

- Emit on first value and whenever `v[key]` compares unequal to memory.

## Worked trace

Objects `{id:1, n:0}`, `{id:1, n:1}`, `{id:2, n:2}` with key `id` write the first and the third objects.

## Why this is Mealy rather than Moore

Same Mealy comparator as `distinctUntilChanged`.

## Edge cases fixed by the 7.x source

- The whole object is emitted, not the key.
- Missing key yields `undefined` and participates in comparison.

## Source anchors

- `src/internal/operators/distinctUntilKeyChanged.ts` delegates to `distinctUntilChanged`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
