# Test plan: `pluck`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/transformation/pluck.md](../../operators/transformation/pluck.md).

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/pluck.ts` |
| Signature | `pluck(...properties): OperatorFunction<T, R>` |
| Status | Deprecated. Use `map(x => x.a.b)`. |

## Behavior under test

Project each next through a path of property names. Missing segments yield `undefined`. Error and complete pass through.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { active, stopped }`. No index.

Memory at subscribe: `active`.

Events: `{ next(v), error, complete, unsubscribe }`.

Possible notifications: `{ next(path(v)), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``active × next → active`.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Terminal → `stopped`.`
4. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``active × next(v) → next(v[k1][k2]...)`.`
5. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Terminal inputs copy through.`
6. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
7. Source edge 1: `Does not throw on missing properties; it emits `undefined`.`
8. Source edge 2: `Deprecated.`

## Sequence from the analysis

`pluck('a', 'b')` on `{a:{b:1}}` writes `next(1)`. On `{}` writes `next(undefined)`.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
