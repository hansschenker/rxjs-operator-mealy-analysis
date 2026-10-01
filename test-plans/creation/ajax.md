# Test plan: `ajax`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/creation/ajax.md](../../operators/creation/ajax.md).

| | |
|---|---|
| Category | Creation |
| Kind | Cold creation function (HTTP producer) |
| RxJS 7.x source | `src/internal/ajax/ajax.ts` |
| Signature | `ajax(urlOrRequest: string |
| Status | Stable in 7.x. Config object is the supported form; string URL is shorthand. |

## Behavior under test

`ajax` does not transform an upstream. Subscription is the start event: it opens one XMLHttpRequest / fetch-equivalent, and cancellation aborts it. The machine is cold — every subscriber builds its own request from `AjaxConfig` (or from a string URL coerced into a config). Progress events are optional `next` outputs when `includeDownloadProgress` / upload progress is configured; the terminal success value is a single `AjaxResponse`. A non-2xx status is an error notification (`AjaxError`), not a next, unless `includeDownloadProgress` handling says otherwise for progress frames.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { idle, opened(request), stopped }`. `opened` remembers the in-flight request handle so unsubscribe can abort it. There is no value memory: the response is not accumulated inside the machine beyond the XHR buffer owned by the host.

Memory at subscribe: `idle`. No request exists before subscribe.

Events: `{ subscribe, progress(p), load(response), fail(error), unsubscribe }`. `progress` is absent unless the config asks for it. `load` is the host success signal; `fail` covers network failure, abort, timeout, and HTTP error statuses mapped to `AjaxError`.

Possible notifications: `{ next(AjaxResponse), next(progress), error(AjaxError), complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``idle × subscribe → opened(request)` (request constructed and sent).`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``opened × progress(p) → opened` (handle unchanged).`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``opened × load(response) → stopped`.`
5. Transition 4: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``opened × fail(error) → stopped`.`
6. Transition 5: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``opened × unsubscribe → stopped` (abort the request).`
7. Transition 6: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``stopped × _ → stopped`.`
8. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``idle × subscribe → ε` (the send itself is an action; no notification yet).`
9. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``opened × progress(p) → next(progressEvent)` when progress is included, else this input is not in Z.`
10. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``opened × load(response) → next(AjaxResponse) · complete`.`
11. Output 4: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``opened × fail(error) → error(AjaxError)`.`
12. Output 5: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``opened × unsubscribe → ε` (abort, no terminal notification if the consumer already left).`
13. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
14. Source edge 1: `Unsubscribe aborts; a late `load` after abort must not emit (the stopped state drops it).`
15. Source edge 2: `Each subscription creates its own XHR. `ajax` is not multicast.`
16. Source edge 3: `Query serialization, headers, cross-domain, and `responseType` are config parameters of the machine, not extra states.`

## Sequence from the analysis

`subscribe` → `opened`; host `load` → output `next(response) · complete`, state `stopped`. A second subscriber is a fresh machine in `idle`, not a replay.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
