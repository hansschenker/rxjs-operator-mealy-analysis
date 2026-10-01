# `fromFetch` — Mealy 6-tuple

| | |
|---|---|
| Category | Creation |
| Kind | Cold fetch producer |
| RxJS 7.x source | `src/internal/observable/dom/fetch.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `fromFetch(input, init?): Observable<Response>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Subscribe calls `fetch`. The Response is a single next, then complete. Unsubscribe aborts via AbortController. A fetch rejection is an error. The body is not read unless the caller reads the Response.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`{ idle, inflight(controller), stopped }`.

## 2. Initial state (S0)

`idle`.

## 3. Input alphabet (Z)

`{ subscribe, resolve(response), reject(error), unsubscribe }`.

## 4. Output alphabet (A)

`{ next(Response), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Subscribe → inflight.
- Resolve or reject → stopped.
- Unsubscribe → stopped and abort.

## 6. Output function (G : S × Z → A*)

- Resolve → next(response) · complete.
- Reject → error.
- Unsubscribe → ε.

## Worked trace

A successful fetch writes `next(response) complete`. Unsubscribing before resolve aborts and writes nothing.

## Why this is Mealy rather than Moore

Resolve and reject are different inputs in `inflight` and write different words. Abort moves to stopped with ε.

## Edge cases fixed by the 7.x source

- Selector overload can project the Response to another observable; that projection is then flattened.
- Cold: one fetch per subscription.

## Source anchors

- `src/internal/observable/dom/fetch.ts` exports `fromFetch`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
