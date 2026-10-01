# Test plan: `raceWith`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/join/raceWith.md](../../operators/join/raceWith.md).

| | |
|---|---|
| Category | Join |
| Kind | Pipeable join |
| RxJS 7.x source | `src/internal/operators/raceWith.ts` |
| Signature | `raceWith(...otherSources): OperatorFunction<T, T>` |
| Status | Stable. |

## Behavior under test

Subscribe to the source and the others. The first to emit a next, error, or complete wins. Losers are unsubscribed. The winner is forwarded.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `{ racing, forwarding(winner), stopped }`.

Memory at subscribe: `racing`.

Events: `{ next_i, error_i, complete_i, unsubscribe }`.

Possible notifications: `{ next, error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `First signal from i → forwarding(i) or stopped if that signal is terminal.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Later signals from losers are not delivered.`
4. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Winning next → next, then later winner notifications copy through.`
5. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Winning error → error.`
6. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Winning complete → complete.`
7. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
8. Source edge 1: `Subscription order matters for synchronous sources.`
9. Source edge 2: `Same machine as creation `race`, with the piped source included.`

## Sequence from the analysis

A synchronous source wins against a later timer and the timer is unsubscribed.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
