# Test plan: `distinctUntilKeyChanged`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/filtering/distinctUntilKeyChanged.md](../../operators/filtering/distinctUntilKeyChanged.md).

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/distinctUntilKeyChanged.ts` |
| Signature | `distinctUntilKeyChanged(key, compare?): MonoTypeOperatorFunction<T>` |
| Status | Stable. |

## Behavior under test

`distinctUntilChanged` with `keySelector = value => value[key]`. Same one-key memory.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { empty, holding(keyValue), stopped }`.

Memory at subscribe: `empty`.

Events: `{ next(v), error, complete, unsubscribe }`.

Possible notifications: `{ next(v), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Same as `distinctUntilChanged`, key is `v[key]`.`
3. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Emit on first value and whenever `v[key]` compares unequal to memory.`
4. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
5. Source edge 1: `The whole object is emitted, not the key.`
6. Source edge 2: `Missing key yields `undefined` and participates in comparison.`

## Sequence from the analysis

Objects `{id:1, n:0}`, `{id:1, n:1}`, `{id:2, n:2}` with key `id` write the first and the third objects.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
