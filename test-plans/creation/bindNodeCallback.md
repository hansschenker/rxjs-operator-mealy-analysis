# Test plan: `bindNodeCallback`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/creation/bindNodeCallback.md](../../operators/creation/bindNodeCallback.md).

| | |
|---|---|
| Category | Creation |
| Kind | Cold creation function (Node error-first adapter) |
| RxJS 7.x source | `src/internal/observable/bindNodeCallback.ts` |
| Signature | `bindNodeCallback(callbackFunc, resultSelector?, scheduler?): (...args) => Observable<T>` |
| Status | Stable. `resultSelector` deprecated. |

## Behavior under test

Same adapter shape as `bindCallback`, but the callback is Node-style `(err, ...results)`. A truthy first argument is an error output and there is no `next`. A null/undefined error yields `next` of the remaining args (a single result unwrapped, several results as an array) and then `complete`.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { idle, waiting, stopped }`.

Memory at subscribe: `idle`.

Events: `{ subscribe(args), callback(err, results), unsubscribe }`.

Possible notifications: `{ next(result), error(err), complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``idle × subscribe → waiting` (or `stopped` if the function throws).`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``waiting × callback(err, _) → stopped` when `err` is truthy.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``waiting × callback(null, results) → stopped`.`
5. Transition 4: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``waiting × unsubscribe → stopped`.`
6. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``waiting × callback(err, _) → error(err)` if `err` is truthy.`
7. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``waiting × callback(null, results) → next(unwrapped) · complete`.`
8. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``subscribe` that throws → `error(e)`.`
9. Output 4: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``unsubscribe → ε`.`
10. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
11. Source edge 1: `Only the first callback counts.`
12. Source edge 2: `A falsy error (`null`/`undefined`) is success. A truthy error short-circuits results.`

## Sequence from the analysis

Subscribe to `bindNodeCallback(fs.readFile)(path)`. Callback `(null, buf)` writes `next(buf) · complete`. Callback `(enoent, _)` writes `error(enoent)` and stops.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
