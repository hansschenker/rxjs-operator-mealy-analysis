# Test plan: `isEmpty`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/conditional/isEmpty.md](../../operators/conditional/isEmpty.md).

| | |
|---|---|
| Category | Conditional and Boolean |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/isEmpty.ts` |
| Signature | `isEmpty(): OperatorFunction<T, boolean>` |
| Status | Stable. |

## Behavior under test

If the source emits any next, emit `false` and complete, unsubscribing. If the source completes with no next, emit `true` and complete.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { empty, stopped }`.

Memory at subscribe: `empty`.

Events: `{ next, error, complete, unsubscribe }`.

Possible notifications: `{ next(boolean), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``next → stopped`.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``complete → stopped`.`
4. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``next → next(false) · complete`.`
5. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``complete → next(true) · complete`.`
6. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``error → error`.`
7. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
8. Source edge 1: `Does not wait for complete after the first value.`
9. Source edge 2: `Error is not converted to a boolean.`

## Sequence from the analysis

`EMPTY.pipe(isEmpty())` writes `next(true) complete`. `of(1).pipe(isEmpty())` writes `next(false) complete`.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
