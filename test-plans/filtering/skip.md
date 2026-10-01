# Test plan: `skip`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/filtering/skip.md](../../operators/filtering/skip.md).

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/skip.ts` |
| Signature | `skip(count: number): MonoTypeOperatorFunction<T>` |
| Status | Stable. |

## Behavior under test

Drop the first `count` source nexts, then mirror the source. Error and complete always pass through.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { skipping(k), forwarding, stopped }` for `0 ≤ k ≤ count`.

Memory at subscribe: `skipping(0)` if count > 0, else `forwarding`.

Events: `{ next(v), error, complete, unsubscribe }`.

Possible notifications: `{ next(v), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``skipping(k) × next → skipping(k+1)` while `k+1 < count`, else `forwarding`.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Terminal → `stopped`.`
4. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``skipping × next → ε`.`
5. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``forwarding × next → next(v)`.`
6. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `The next that reaches count is the first forwarded, or is still skipped depending on the off-by-one: after `count` skips, subsequent values emit. The value that increments k to count is skipped.`
7. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
8. Source edge 1: ``skip(0)` is identity.`
9. Source edge 2: `Does not unsubscribe; it only drops.`

## Sequence from the analysis

`skip(2)` on `a b c d` writes `next(c) next(d) complete`.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
