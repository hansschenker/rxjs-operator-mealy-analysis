# Test plan: `exhaust`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/transformation/exhaust.md](../../operators/transformation/exhaust.md).

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable higher-order operator |
| RxJS 7.x source | `src/internal/operators/exhaust.ts` |
| Signature | `exhaust(): OperatorFunction<ObservableInput<T>, T>` |
| Status | Stable. Alias of `exhaustAll`. |

## Behavior under test

Source emits inners. If no inner is active, subscribe to the new inner and forward it. If an inner is active, drop the new inner without subscribing. Complete when the source is done and no inner is active.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { idle, busy(inner), stopped }` plus `outerDone`.

Memory at subscribe: `idle`, outer not done.

Events: `{ outerNext(inner$), outerError, outerComplete, innerNext, innerError, innerComplete, unsubscribe }`.

Possible notifications: `{ next, error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``idle × outerNext → busy`.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``busy × outerNext → busy` (dropped).`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``busy × innerComplete → idle`, then `stopped` if outer is done.`
5. Transition 4: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Errors → `stopped`.`
6. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``innerNext → next`.`
7. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``outerNext → ε` in both idle and busy (subscribe action only when idle).`
8. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``innerComplete → complete` if outer done, else `ε`.`
9. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
10. Source edge 1: `Dropped inners are never subscribed.`
11. Source edge 2: `Source file is a one-line alias to `exhaustAll`.`

## Sequence from the analysis

Clicks projected to 1-second inners: a click during an open inner produces no subscription and no output.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
