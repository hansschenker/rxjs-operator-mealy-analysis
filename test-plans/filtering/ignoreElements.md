# Test plan: `ignoreElements`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/filtering/ignoreElements.md](../../operators/filtering/ignoreElements.md).

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/ignoreElements.ts` |
| Signature | `ignoreElements(): OperatorFunction<T, never>` |
| Status | Stable. |

## Behavior under test

Drop every next. Forward error and complete only.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { active, stopped }`.

Memory at subscribe: `active`.

Events: `{ next(v), error, complete, unsubscribe }`.

Possible notifications: `{ error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``next` stays `active`.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Terminal → `stopped`.`
4. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``next → ε`.`
5. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``error → error`, `complete → complete`.`
6. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
7. Source edge 1: `Useful to keep a stream's terminal signals while discarding values.`
8. Source edge 2: `Does not delay complete.`

## Sequence from the analysis

`of(1, 2, 3).pipe(ignoreElements())` writes `complete`.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
