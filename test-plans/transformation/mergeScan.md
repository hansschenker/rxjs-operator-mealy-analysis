# Test plan: `mergeScan`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/transformation/mergeScan.md](../../operators/transformation/mergeScan.md).

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable higher-order accumulator |
| RxJS 7.x source | `src/internal/operators/mergeScan.ts` |
| Signature | `mergeScan(accumulator, seed, concurrent = Infinity): OperatorFunction<T, R>` |
| Status | Stable. |

## Behavior under test

Like `scan`, but the accumulator returns an observable. Seed is the initial acc. Each outer value starts `accumulator(acc, value)` and the inner's emissions are both outputs and the latest acc. Concurrency defaults to Infinity. Complete when outer is done and inners are idle.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { acc, active, queue, outerDone, stopped }`.

Memory at subscribe: `acc = seed`, active 0, empty queue.

Events: Higher-order alphabet.

Possible notifications: `{ next(acc'), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``outerNext` queues work that closes over the acc at start time according to `mergeInternals` scan semantics.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Inner next replaces `acc`.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Idle and outer done → `stopped`.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``innerNext(r) → next(r)` and `acc := r`.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Seed is not emitted by itself.`
7. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Errors copy through.`
8. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
9. Source edge 1: `High concurrency can start accumulators with a stale acc if the implementation does not serialize the seed read. 7.x `mergeScan` uses `mergeInternals` and passes the latest acc when the inner is subscribed; overlapping inners can still interleave emissions.`
10. Source edge 2: `Seed is required.`

## Sequence from the analysis

`of(1, 1).pipe(mergeScan((acc, v) => of(acc + v), 0))` writes `next(1) next(2) complete` when serialized by the accumulator dependency.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
