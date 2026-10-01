# Test plan: `mergeAll`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/join/mergeAll.md](../../operators/join/mergeAll.md).

| | |
|---|---|
| Category | Join |
| Kind | Pipeable higher-order join |
| RxJS 7.x source | `src/internal/operators/mergeAll.ts` |
| Signature | `mergeAll(concurrent = Infinity): OperatorFunction<ObservableInput<T>, T>` |
| Status | Stable. |

## Behavior under test

Subscribe to inners as the source emits them, up to `concurrent`, and forward their values as they arrive. Extra inners queue.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { active count, queue, outerDone, stopped }`.

Memory at subscribe: Active 0, empty queue.

Events: Higher-order alphabet.

Possible notifications: `{ next, error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Start inners up to concurrency.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``innerComplete` pulls the queue.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `All done → `stopped`.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``innerNext → next`.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Completion word only when active and queue are empty and outer is done.`
7. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
8. Source edge 1: ``mergeAll(1)` matches `concatAll`.`
9. Source edge 2: `Implemented through `mergeInternals`.`

## Sequence from the analysis

Two delayed inners can interleave their nexts on the output.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
