# Test plan: `distinctUntilChanged`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/filtering/distinctUntilChanged.md](../../operators/filtering/distinctUntilChanged.md).

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/distinctUntilChanged.ts` |
| Signature | `distinctUntilChanged(comparator?, keySelector?): MonoTypeOperatorFunction<T>` |
| Status | Stable. |

## Behavior under test

Emit when the current key is not equal to the previous key. Comparator defaults to `===`. Only the last key is stored, not the full history.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { empty, holding(key), stopped }`.

Memory at subscribe: `empty`.

Events: `{ next(v), error, complete, unsubscribe }`.

Possible notifications: `{ next(v), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``empty × next(v) → holding(key(v))`.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``holding(k) × next(v) → holding(key(v))` whether or not it emits.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Terminal → `stopped`.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``empty × next(v) → next(v)`.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``holding(k) × next(v) → next(v)` if `compare(k, key(v))` is false, else `ε`.`
7. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Throw in comparator or key selector → `error`.`
8. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
9. Source edge 1: `First value always passes.`
10. Source edge 2: `Comparator receives keys after `keySelector`.`

## Sequence from the analysis

`1, 1, 2, 2, 1` writes `next(1) next(2) next(1)`. The last 1 passes because it differs from 2.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
