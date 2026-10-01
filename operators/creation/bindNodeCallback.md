# `bindNodeCallback` — Mealy 6-tuple

| | |
|---|---|
| Category | Creation |
| Kind | Cold creation function (Node error-first adapter) |
| RxJS 7.x source | `src/internal/observable/bindNodeCallback.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `bindNodeCallback(callbackFunc, resultSelector?, scheduler?): (...args) => Observable<T>` |
| Status on the 7.x line | Stable. `resultSelector` deprecated. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Same adapter shape as `bindCallback`, but the callback is Node-style `(err, ...results)`. A truthy first argument is an error output and there is no `next`. A null/undefined error yields `next` of the remaining args (a single result unwrapped, several results as an array) and then `complete`.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { idle, waiting, stopped }`.

## 2. Initial state (S0)

`idle`.

## 3. Input alphabet (Z)

`{ subscribe(args), callback(err, results), unsubscribe }`.

## 4. Output alphabet (A)

`{ next(result), error(err), complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `idle × subscribe → waiting` (or `stopped` if the function throws).
- `waiting × callback(err, _) → stopped` when `err` is truthy.
- `waiting × callback(null, results) → stopped`.
- `waiting × unsubscribe → stopped`.

## 6. Output function (G : S × Z → A*)

- `waiting × callback(err, _) → error(err)` if `err` is truthy.
- `waiting × callback(null, results) → next(unwrapped) · complete`.
- `subscribe` that throws → `error(e)`.
- `unsubscribe → ε`.

## Worked trace

Subscribe to `bindNodeCallback(fs.readFile)(path)`. Callback `(null, buf)` writes `next(buf) · complete`. Callback `(enoent, _)` writes `error(enoent)` and stops.

## Why this is Mealy rather than Moore

Identical state `waiting` maps to either `error` or `next·complete` according to the callback symbol. That input-dependence is why the machine is Mealy.

## Edge cases fixed by the 7.x source

- Only the first callback counts.
- A falsy error (`null`/`undefined`) is success. A truthy error short-circuits results.

## Source anchors

- `src/internal/observable/bindNodeCallback.ts`.
- Error-first convention is the only structural difference from `bindCallback`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
