# Test plan: `repeat`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/utility/repeat.md](../../operators/utility/repeat.md).

| | |
|---|---|
| Category | Utility |
| Kind | Pipeable resubscribe |
| RxJS 7.x source | `src/internal/operators/repeat.ts` |
| Signature | `repeat(countOrConfig?): MonoTypeOperatorFunction<T>` |
| Status | Stable. Config form `{ count, delay }` on 7.x. |

## Behavior under test

Forward the source. On complete, resubscribe if repeats remain. `count` is the number of times the source is subscribed in total in the numeric form used by 7.x docs (repeat(1) means one subscription, no extra repeat). Delay waits before the resubscribe. Error is not repeated; it is forwarded. Infinite count repeats forever.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `{ forwarding(n), waitingDelay, stopped }`.

Memory at subscribe: `forwarding(1)`.

Events: `{ next, error, complete, delayTick, unsubscribe }`.

Possible notifications: `{ next, error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Next stays.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Complete → waitingDelay if another subscription is allowed, else stopped.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `delayTick → forwarding(n+1).`
5. Transition 4: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Error → stopped.`
6. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Next → next.`
7. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Complete → ε if a repeat will happen, else complete.`
8. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Error → error.`
9. Output 4: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Resubscribe is an action.`
10. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
11. Source edge 1: `Error does not repeat.`
12. Source edge 2: `Delay notifier error becomes the output error.`
13. Source edge 3: `count Infinity never writes the final complete.`

## Sequence from the analysis

`of(1).pipe(repeat(2))` writes `next(1) next(1) complete`. The complete of the first subscription is swallowed.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
