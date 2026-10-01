# RxJS operator Mealy analysis

SuperGrok is the main contributor of this project.

This repository analyzes every public RxJS 7.x operator as a Mealy machine, using the 6-tuple

1. State space `(S)`
2. Initial state `(S0)`
3. Input alphabet `(Z)`
4. Output alphabet `(A)`
5. Transition function `(T : S × Z → S)`
6. Output function `(G : S × Z → A*)`

Each operator has its own artifact under [`operators/`](operators/README.md). The models are grounded in the RxJS 7.x sources at <https://github.com/ReactiveX/rxjs/tree/7.x/src/internal/operators>, revision `e5351d02e225e275ac0e497c7b66eaa5f0c88791`. Creation functions that do not live in that directory are read from their real 7.x files (`src/internal/observable/*`, `src/internal/ajax/ajax.ts`) and the file path is recorded in the artifact.

## Contributor

SuperGrok is the main contributor of this project.

- Main contributor: SuperGrok (supergrok@x.ai)
- Repository owner: Hans Schenker (hansschenker)

Commits include the trailer `Co-authored-by: SuperGrok <supergrok@x.ai>` so SuperGrok is recorded as a contributor.

## Method

See [docs/methodology.md](docs/methodology.md). The matching test plan is [docs/test-plan.md](docs/test-plan.md), with one case list per operator under [test-plans/](test-plans/README.md).

Notifications are the alphabet. Operator memory is the state. The output word may be empty (`ε`) or several letters long (`next · complete`). A stopped machine is absorbing.

## Coverage

136 analyses. That is the official catalog, plus every other public operator file in `src/internal/operators` (`combineLatestWith`, `shareReplay`, `repeat`, `connect`, `sequenceEqual`, and the deprecated aliases) and the public creation functions that live beside that tree (`never`, `pairs`, `using`, `onErrorResumeNext`, `animationFrames`, `fromFetch`, `webSocket`). `partition` is listed twice on the official page and is analyzed once, under join creation. Internal helpers (`OperatorSubscriber`, `mergeInternals`, `scanInternals`) are not operators and have no artifact.

| Category | Count | Folder |
|---|---|---|
| Creation | 21 | [operators/creation](operators/creation) |
| Join creation | 7 | [operators/join-creation](operators/join-creation) |
| Transformation | 28 | [operators/transformation](operators/transformation) |
| Filtering | 25 | [operators/filtering](operators/filtering) |
| Join | 15 | [operators/join](operators/join) |
| Multicasting | 9 | [operators/multicasting](operators/multicasting) |
| Error handling | 6 | [operators/error-handling](operators/error-handling) |
| Utility | 15 | [operators/utility](operators/utility) |
| Conditional and Boolean | 6 | [operators/conditional](operators/conditional) |
| Mathematical and Aggregate | 4 | [operators/mathematical](operators/mathematical) |

## Source of the operator list

The official RxJS operator catalog, matched against the 7.x implementation rather than the docs marble diagrams alone, then completed from the public files in `src/internal/operators`, `src/internal/observable`, and `src/internal/observable/dom`.
