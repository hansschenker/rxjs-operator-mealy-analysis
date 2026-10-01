# `withLatestFrom` — Mealy 6-tuple

| | |
|---|---|
| Category | Join |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/withLatestFrom.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `withLatestFrom(...others, project?): OperatorFunction<T, R>` |
| Status on the 7.x line | Stable. Project form deprecated in favor of a later `map`. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Subscribe to the other sources and remember their latest values. Only a next from the main source emits, and only once every other has a latest. The output is `[main, ...latests]` or the projection. Others completing does not complete the output. Main complete completes the output.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { latest_i: V | ⊥, stopped }`.

## 2. Initial state (S0)

All others `⊥`.

## 3. Input alphabet (Z)

`{ mainNext(v), mainError, mainComplete, otherNext_i, otherError_i, otherComplete_i, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(tuple), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `otherNext_i` stores latest.
- `mainNext` does not store a lasting main value.
- Main complete or any error → `stopped`.

## 6. Output function (G : S × Z → A*)

- `mainNext(v) → next([v, ...latests])` if every other has a latest, else `ε`.
- `otherNext → ε`.
- `mainComplete → complete`.

## Worked trace

Main emits before the other has emitted: `ε`. After the other emits `a` and main emits `1`: `next([1, a])`. A later other value does not emit by itself.

## Why this is Mealy rather than Moore

Only the main-next input writes, and it writes a letter built from other-state plus that input. Other nexts are state updates with `ε`.

## Edge cases fixed by the 7.x source

- Other complete without a value leaves the machine unable to emit.
- Other error fails the output.

## Source anchors

- `src/internal/operators/withLatestFrom.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
