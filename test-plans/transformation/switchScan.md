# Test plan: `switchScan`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/transformation/switchScan.md](../../operators/transformation/switchScan.md).

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable higher-order accumulator |
| RxJS 7.x source | `src/internal/operators/switchScan.ts` |
| Signature | `switchScan(accumulator, seed): OperatorFunction<T, R>` |
| Status | Stable. |

## Behavior under test

Like `mergeScan` with switch semantics. A new outer value unsubscribes the active accumulator inner and subscribes to `accumulator(latestAcc, value)`. Inner emissions update `acc` and are forwarded.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { acc, active inner | ⊥, outerDone, stopped }`.

Memory at subscribe: `acc = seed`, no inner.

Events: Higher-order alphabet.

Possible notifications: `{ next(r), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``outerNext` unsubscribes the active inner and subscribes to the new accumulation.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``innerNext` replaces `acc`.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``innerComplete` clears active; if outer done → `stopped`.`
5. Transition 4: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Errors → `stopped`.`
6. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``innerNext(r) → next(r)`.`
7. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``outerNext → ε` (switch action only).`
8. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``innerComplete → complete` if outer done, else `ε`.`
9. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
10. Source edge 1: `Seed is required and not emitted up front.`
11. Source edge 2: `Unsubscribed inner emissions are not outputs.`

## Sequence from the analysis

A fast outer with a slow accumulator inner: only the latest accumulation survives; previous inner nexts stop.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
