# Custom operator template

SuperGrok is the main contributor of this project.

Copy this file, replace the prompts, and keep it next to the operator. Do not implement a behavior that has no row. Method: [custom-operator-checklist.md](custom-operator-checklist.md).

## Name

Name of the operator, and one sentence for the policy.

## 1. Memory

What it can remember between events. Include the finished memory.

## 2. Memory at subscribe

What is remembered before any source notification. Say if subscribe itself subscribes to a notifier, opens a resource, or sends a prefix.

## 3. Events

List every event: source value, source error, source completion, unsubscribe, and any tick or inner event.

## 4. Notifications and actions

What it can send, including silence. Name abort, inner unsubscribe, or dispose if they matter.

## 5. Next memory

One row per memory and event that can happen.

- Memory, event, next memory.

## 6. What is sent

The same rows, with the output word. Silence is an answer.

- Memory, event, sent.

## Edges

- Completion while something is held:
- Error while something is held:
- Unsubscribe while a timer, inner, or resource is open:
- Late event after finish:
- Throw from a projection or factory:

## Sequence

Walk one short input by hand and write the output word. If you cannot, a row is missing.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this template.
