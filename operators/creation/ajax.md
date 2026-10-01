# `ajax` — Mealy 6-tuple

| | |
|---|---|
| Category | Creation |
| Kind | Cold creation function (HTTP producer) |
| RxJS 7.x source | `src/internal/ajax/ajax.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `ajax(urlOrRequest: string | AjaxConfig): Observable<AjaxResponse<T>>` |
| Status on the 7.x line | Stable in 7.x. Config object is the supported form; string URL is shorthand. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

`ajax` does not transform an upstream. Subscription is the start event: it opens one XMLHttpRequest / fetch-equivalent, and cancellation aborts it. The machine is cold — every subscriber builds its own request from `AjaxConfig` (or from a string URL coerced into a config). Progress events are optional `next` outputs when `includeDownloadProgress` / upload progress is configured; the terminal success value is a single `AjaxResponse`. A non-2xx status is an error notification (`AjaxError`), not a next, unless `includeDownloadProgress` handling says otherwise for progress frames.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { idle, opened(request), stopped }`. `opened` remembers the in-flight request handle so unsubscribe can abort it. There is no value memory: the response is not accumulated inside the machine beyond the XHR buffer owned by the host.

## 2. Initial state (S0)

`idle`. No request exists before subscribe.

## 3. Input alphabet (Z)

`{ subscribe, progress(p), load(response), fail(error), unsubscribe }`. `progress` is absent unless the config asks for it. `load` is the host success signal; `fail` covers network failure, abort, timeout, and HTTP error statuses mapped to `AjaxError`.

## 4. Output alphabet (A)

`{ next(AjaxResponse), next(progress), error(AjaxError), complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `idle × subscribe → opened(request)` (request constructed and sent).
- `opened × progress(p) → opened` (handle unchanged).
- `opened × load(response) → stopped`.
- `opened × fail(error) → stopped`.
- `opened × unsubscribe → stopped` (abort the request).
- `stopped × _ → stopped`.

## 6. Output function (G : S × Z → A*)

- `idle × subscribe → ε` (the send itself is an action; no notification yet).
- `opened × progress(p) → next(progressEvent)` when progress is included, else this input is not in Z.
- `opened × load(response) → next(AjaxResponse) · complete`.
- `opened × fail(error) → error(AjaxError)`.
- `opened × unsubscribe → ε` (abort, no terminal notification if the consumer already left).

## Worked trace

`subscribe` → `opened`; host `load` → output `next(response) · complete`, state `stopped`. A second subscriber is a fresh machine in `idle`, not a replay.

## Why this is Mealy rather than Moore

The same `opened` state produces either `next·complete` or `error` depending on whether the input is `load` or `fail`. A Moore machine would have to split `opened` into ghost states just to encode the pending output.

## Edge cases fixed by the 7.x source

- Unsubscribe aborts; a late `load` after abort must not emit (the stopped state drops it).
- Each subscription creates its own XHR. `ajax` is not multicast.
- Query serialization, headers, cross-domain, and `responseType` are config parameters of the machine, not extra states.

## Source anchors

- Implementation lives in `src/internal/ajax/ajax.ts`, not under `src/internal/operators`. The operator directory has no `ajax.ts` on 7.x.
- Cold: the factory runs on subscribe, which is why `S0 = idle` and `subscribe ∈ Z`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
