# Test plan: `iif`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/creation/iif.md](../../operators/creation/iif.md).

| | |
|---|---|
| Category | Creation |
| Kind | Cold conditional factory |
| RxJS 7.x source | `src/internal/observable/iif.ts` |
| Signature |  |
| Status | Stable. |

## Behavior under test

Subscribe evaluates `condition()` and subscribes to either the true or the false observable. Missing branch is `EMPTY` (immediate complete). After the choice, the machine is a forwarder. The condition is not re-checked.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { idle, forwarding(branch), stopped }`.

Memory at subscribe: `idle`.

Events: `{ subscribe, innerNext, innerError, innerComplete, unsubscribe }`.

Possible notifications: `{ next(v), error(e), complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``idle × subscribe → forwarding(true)` or `forwarding(false)` based on `condition()`.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `A throw in `condition` → `stopped`.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Inner terminal or unsubscribe → `stopped`.`
5. Transition 4: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Inner next stays in `forwarding`.`
6. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``subscribe` throw → `error(e)`, else `ε` (branch subscription is an action).`
7. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Inner notifications are copied to the output.`
8. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Missing branch writes `complete` on subscribe.`
9. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
10. Source edge 1: `Condition runs per subscription.`
11. Source edge 2: `Default branch completes empty.`

## Sequence from the analysis

`iif(() => flag, of(1), of(2))` with `flag = true` writes `next(1) complete` and never subscribes to `of(2)`.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
