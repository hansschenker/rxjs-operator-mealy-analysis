# Test plan: `zipAll`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/join/zipAll.md](../../operators/join/zipAll.md).

| | |
|---|---|
| Category | Join |
| Kind | Pipeable higher-order join |
| RxJS 7.x source | `src/internal/operators/zipAll.ts` |
| Signature | `zipAll(project?): OperatorFunction<ObservableInput<T>, T[]>` |
| Status | Stable. |

## Behavior under test

Collect inners from the source until it completes, then zip them by index. Emit a tuple whenever every inner can contribute one unused value. Complete when a tuple can no longer be formed.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: Collected inners, a queue per inner, outerDone, stopped.

Memory at subscribe: No inners.

Events: Higher-order alphabet plus inner next/error/complete.

Possible notifications: `{ next(tuple), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Outer next appends an inner.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Outer complete freezes the set and starts pairing.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Inner next appends to that queue; shift when all queues are non-empty.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Inner next → next(tuple) if it completed a row, else ε.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `A complete that leaves a queue empty and done → complete.`
7. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Any error → error.`
8. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
9. Source edge 1: `No inners: complete without a next.`
10. Source edge 2: `Project form is deprecated style.`

## Sequence from the analysis

Source `of(of(1, 2), of('a'))` then zipAll writes `next([1, 'a']) complete`. The leftover 2 is dropped.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
