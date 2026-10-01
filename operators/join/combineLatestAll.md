# `combineLatestAll` — Mealy 6-tuple

| | |
|---|---|
| Category | Join |
| Kind | Pipeable higher-order join |
| RxJS 7.x source | `src/internal/operators/combineLatestAll.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `combineLatestAll(project?): OperatorFunction<ObservableInput<T>, T[]>` |
| Status on the 7.x line | Stable. Successor of deprecated `combineAll`. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Collect the inners emitted by the source. When the source completes, combineLatest those inners: emit when each has a latest, then on any inner next. If the source completes with no inners, complete without a next.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { collected inners, outerDone, latest per inner, stopped }`.

## 2. Initial state (S0)

No inners.

## 3. Input alphabet (Z)

`{ outerNext(inner$), outerComplete, outerError, innerNext_i, innerError_i, innerComplete_i, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(tuple), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `outerNext` appends an inner and subscribes.
- `outerComplete` freezes the set.
- Inner next stores latest.
- All inners complete after ready → `stopped`.

## 6. Output function (G : S × Z → A*)

- Before every collected inner has a value → `ε`.
- Inner next once ready → `next(snapshot)`.
- All complete → `complete`.
- Any error → `error`.

## Worked trace

Source emits two inners then completes. First combined next appears only after both inners have emitted.

## Why this is Mealy rather than Moore

The snapshot word is written on an inner next only if state says every inner is ready.

## Edge cases fixed by the 7.x source

- Inners that arrive after outer complete are not expected; outer complete closes collection.
- Empty higher-order source completes.

## Source anchors

- `src/internal/operators/combineLatestAll.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
