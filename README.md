# RxJS operator Mealy analysis

SuperGrok is the main contributor of this project.

This repository analyzes every operator on the official RxJS operator list as a Mealy machine, using the 6-tuple

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

See [docs/methodology.md](docs/methodology.md).

Notifications are the alphabet. Operator memory is the state. The output word may be empty (`ε`) or several letters long (`next · complete`). A stopped machine is absorbing.

## Coverage

111 analyses. `partition` is listed twice on the official page (join creation and transformation) and is analyzed once, under join creation.

| Category | Count | Folder |
|---|---|---|
| Creation | 15 | [operators/creation](operators/creation) |
| Join creation | 7 | [operators/join-creation](operators/join-creation) |
| Transformation | 27 | [operators/transformation](operators/transformation) |
| Filtering | 25 | [operators/filtering](operators/filtering) |
| Join | 7 | [operators/join](operators/join) |
| Multicasting | 6 | [operators/multicasting](operators/multicasting) |
| Error handling | 3 | [operators/error-handling](operators/error-handling) |
| Utility | 12 | [operators/utility](operators/utility) |
| Conditional and Boolean | 5 | [operators/conditional](operators/conditional) |
| Mathematical and Aggregate | 4 | [operators/mathematical](operators/mathematical) |

## Source of the operator list

The official RxJS operator catalog (creation, join creation, transformation, filtering, join, multicasting, error handling, utility, conditional, mathematical and aggregate), matched against the 7.x implementation rather than the docs marble diagrams alone.
