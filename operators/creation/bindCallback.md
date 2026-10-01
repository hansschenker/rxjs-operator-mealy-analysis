# `bindCallback` — Mealy 6-tuple

| | |
|---|---|
| Category | Creation |
| Kind | Cold creation function (callback adapter) |
| RxJS 7.x source | `src/internal/observable/bindCallback.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `bindCallback(callbackFunc, resultSelector?, scheduler?): (...args) => Observable<T>` |
| Status on the 7.x line | Stable. `resultSelector` is deprecated and removed in later majors. |

SuperGrok is the main contributor of this analysis.

## Explanation

`bindCallback` is a cold creation function (callback adapter) on the RxJS 7.x line. Stable. `resultSelector` is deprecated and removed in later majors. `bindCallback` returns a function. Calling that function does not yet run the callback API; subscribing does. The machine appends its own callback, invokes `callbackFunc` once, and turns the callback arguments into one `next` followed by `complete`. Multiple callback arguments are packed into an array unless a (deprecated) result selector projects them.

In plain terms, the operator keeps this memory: S = { idle, waiting, stopped }. waiting means the underlying function has been invoked and the callback has not fired. At subscription, before any source notification, that memory is idle. It reacts to these events: { subscribe(args), callback(args), unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: bindCallback(fs.readFile)(path) stays cold until subscribe. Subscribe invokes readFile; the callback input writes next([errIgnoredOrData]) · complete and stops. (Node-style error-first is bindNodeCallback, not this operator.)

Details that a marble diagram often leaves out: The callback is expected once. A second callback after `stopped` is dropped. Scheduler, if passed, shifts the `next·complete` word onto that scheduler; the state transition still happens when the callback fires. Result selector deprecation does not change the tuple shape, only the projection inside `G`.

## Role in the notification machine

`bindCallback` returns a function. Calling that function does not yet run the callback API; subscribing does. The machine appends its own callback, invokes `callbackFunc` once, and turns the callback arguments into one `next` followed by `complete`. Multiple callback arguments are packed into an array unless a (deprecated) result selector projects them.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { idle, waiting, stopped }`. `waiting` means the underlying function has been invoked and the callback has not fired.

## 2. Initial state (S0)

`idle`.

## 3. Input alphabet (Z)

`{ subscribe(args), callback(args), unsubscribe }`.

## 4. Output alphabet (A)

`{ next(value | args[]), error(e), complete }`. A throw from `callbackFunc` or from the result selector is `error`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `idle × subscribe(args) → waiting` if the call returns normally.
- `idle × subscribe(args) → stopped` if `callbackFunc` throws synchronously.
- `waiting × callback(args) → stopped`.
- `waiting × unsubscribe → stopped` (a late callback is ignored).
- `stopped × _ → stopped`.

## 6. Output function (G : S × Z → A*)

- `idle × subscribe → ε` on the success path; `error(e)` if the call throws.
- `waiting × callback(args) → next(project(args)) · complete`.
- `waiting × unsubscribe → ε`.

## Worked trace

`bindCallback(fs.readFile)(path)` stays cold until subscribe. Subscribe invokes `readFile`; the callback input writes `next([errIgnoredOrData]) · complete` and stops. (Node-style error-first is `bindNodeCallback`, not this operator.)

## Why this is Mealy rather than Moore

`waiting` plus `callback` writes a value word; `waiting` plus `unsubscribe` writes `ε`. Output depends on the input symbol, which is the Mealy condition.

## Edge cases fixed by the 7.x source

- The callback is expected once. A second callback after `stopped` is dropped.
- Scheduler, if passed, shifts the `next·complete` word onto that scheduler; the state transition still happens when the callback fires.
- Result selector deprecation does not change the tuple shape, only the projection inside `G`.

## Source anchors

- `src/internal/observable/bindCallback.ts` on 7.x (creation function, not an operator file).
- Shares internals with `bindNodeCallback` via `bindCallbackInternals`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
