# `fromEvent` — Mealy 6-tuple

| | |
|---|---|
| Category | Creation |
| Kind | Hot-source adapter, cold registration |
| RxJS 7.x source | `src/internal/observable/fromEvent.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `fromEvent(target, eventName, options?, resultSelector?): Observable<T>` |
| Status on the 7.x line | Stable. `resultSelector` deprecated. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Subscribe registers a listener on `target` (`addEventListener`, `on`, or a compatible method). Each event is a `next`. There is no natural complete. Unsubscribe removes the listener. Optional `options` (capture, passive, once) are registration parameters.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { idle, listening, stopped }`.

## 2. Initial state (S0)

`idle`.

## 3. Input alphabet (Z)

`{ subscribe, event(e), unsubscribe }`.

## 4. Output alphabet (A)

`{ next(e | project(e)), error(e) }`. No `complete` in the normal word.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `idle × subscribe → listening`.
- `listening × event(e) → listening`.
- `listening × unsubscribe → stopped`.
- A throw in the result selector: `listening → stopped`.

## 6. Output function (G : S × Z → A*)

- `listening × event(e) → next(e)` (or projected).
- Result-selector throw → `error(e)`.
- `unsubscribe → ε`.

## Worked trace

`fromEvent(el, 'click')`: subscribe arms the listener; three clicks write `next next next`; unsubscribe removes the handler and writes `ε`.

## Why this is Mealy rather than Moore

Events write `next` only while state is `listening`. The same event symbol in `stopped` writes `ε`. Output is state-and-input dependent.

## Edge cases fixed by the 7.x source

- jQuery-style and Node `EventEmitter` targets are detected by method shape.
- `once: true` in options can move `listening → stopped` after the first event, and `G` still writes that one `next`.
- Two subscribers register two listeners unless the caller shares.

## Source anchors

- `src/internal/observable/fromEvent.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
