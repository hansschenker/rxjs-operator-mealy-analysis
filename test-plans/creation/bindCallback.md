# Test plan: `bindCallback`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/creation/bindCallback.md](../../operators/creation/bindCallback.md).

| | |
|---|---|
| Category | Creation |
| Kind | Cold creation function (callback adapter) |
| RxJS 7.x source | `src/internal/observable/bindCallback.ts` |
| Signature | `bindCallback(callbackFunc, resultSelector?, scheduler?): (...args) => Observable<T>` |
| Status | Stable. `resultSelector` is deprecated and removed in later majors. |

## Behavior under test

`bindCallback` returns a function. Calling that function does not yet run the callback API; subscribing does. The machine appends its own callback, invokes `callbackFunc` once, and turns the callback arguments into one `next` followed by `complete`. Multiple callback arguments are packed into an array unless a (deprecated) result selector projects them.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { idle, waiting, stopped }`. `waiting` means the underlying function has been invoked and the callback has not fired.

Memory at subscribe: `idle`.

Events: `{ subscribe(args), callback(args), unsubscribe }`.

Possible notifications: `{ next(value | args[]), error(e), complete }`. A throw from `callbackFunc` or from the result selector is `error`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``idle × subscribe(args) → waiting` if the call returns normally.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``idle × subscribe(args) → stopped` if `callbackFunc` throws synchronously.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``waiting × callback(args) → stopped`.`
5. Transition 4: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``waiting × unsubscribe → stopped` (a late callback is ignored).`
6. Transition 5: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``stopped × _ → stopped`.`
7. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``idle × subscribe → ε` on the success path; `error(e)` if the call throws.`
8. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``waiting × callback(args) → next(project(args)) · complete`.`
9. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``waiting × unsubscribe → ε`.`
10. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
11. Source edge 1: `The callback is expected once. A second callback after `stopped` is dropped.`
12. Source edge 2: `Scheduler, if passed, shifts the `next·complete` word onto that scheduler; the state transition still happens when the callback fires.`
13. Source edge 3: `Result selector deprecation does not change the tuple shape, only the projection inside `G`.`

## Sequence from the analysis

`bindCallback(fs.readFile)(path)` stays cold until subscribe. Subscribe invokes `readFile`; the callback input writes `next([errIgnoredOrData]) · complete` and stops. (Node-style error-first is `bindNodeCallback`, not this operator.)

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
