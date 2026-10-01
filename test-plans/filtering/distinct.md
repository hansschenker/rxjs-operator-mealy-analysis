# Test plan: `distinct`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/filtering/distinct.md](../../operators/filtering/distinct.md).

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/distinct.ts` |
| Signature | `distinct(keySelector?, flushes?): MonoTypeOperatorFunction<T>` |
| Status | Stable. |

## Behavior under test

Emit a value only the first time its key is seen. Key defaults to the value itself. Memory is a set. Optional `flushes` observable clears the set when it emits.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { seen: Set, stopped }`.

Memory at subscribe: Empty set.

Events: `{ next(v), error, complete, flush, flushError, unsubscribe }`.

Possible notifications: `{ next(v), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``next(v)` adds `key(v)` if absent.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``flush` clears the set.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Terminal → `stopped`.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``next(v) → next(v)` if key was absent, else `ε`.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``flush → ε`.`
7. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Error and complete copy through.`
8. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
9. Source edge 1: `Set membership uses the key selector result.`
10. Source edge 2: `Unbounded memory if the key domain is unbounded and no flush is given.`

## Sequence from the analysis

`of(1, 1, 2, 1).pipe(distinct())` writes `next(1) next(2) complete`.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
