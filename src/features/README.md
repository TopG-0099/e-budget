# features/

One folder per business domain, e.g. `cep/` and `cer/`. Each feature folder
holds its own components, TanStack Query hooks, and colocated tests. A
feature may import from `src/services`, `src/models`, `src/components`,
`src/hooks`, and `src/lib`, but other features should not import from each
other directly — share code by promoting it to `src/components`,
`src/hooks`, or `src/lib` instead.

Empty for this MVP scaffold; CEP and CER features are built here later.
