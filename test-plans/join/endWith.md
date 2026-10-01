# Test plan: `endWith`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/join/endWith.md](../../operators/join/endWith.md).

| | |
|---|---|
| Category | Join |
| Kind | Pipeable suffix |
| RxJS 7.x source | `src/internal/operators/endWith.ts` |
| Signature | `endWith(...values, scheduler?): MonoTypeOperatorFunction<T>` |
| Status | Stable. |

## Behavior under test

Forward the source. On source complete, emit the given suffix values and then complete. An error skips the suffix. A scheduler shifts the suffix.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `{ forwarding, suffix(i), stopped }`.

Memory at subscribe: `forwarding`.

Events: `{ next, error, complete, suffixStep, unsubscribe }`.

Possible notifications: `{ next, error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Source next stays forwarding.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Source complete → suffix(0).`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Suffix steps advance i, then stopped.`
5. Transition 4: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Error → stopped without suffix.`
6. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Source next → next.`
7. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Source error → error.`
8. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Suffix step → next(values[i]).`
9. Output 4: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Last suffix step is followed by complete.`
10. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
11. Source edge 1: `Suffix is not emitted if the source errors.`
12. Source edge 2: `A scheduler argument is not a suffix value.`

## Sequence from the analysis

`of(1).pipe(endWith(2, 3))` writes `next(1) next(2) next(3) complete`.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
