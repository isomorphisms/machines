# Edric / Idris 2 port

This branch starts a semantic port of `machines` to Idris 2 using the currently implemented Edric `.idric` surface syntax.

The first slice deliberately ports the representation and behavior before the Haskell typeclass scaffolding:

- `Machine k o` keeps the typed request family `k`, output `o`, `Stop`, `Yield`, and `Await` with a starvation continuation.
- `Plan k o a` is the direct `Done | Emit | Need | Fail` representation described by the Haskell `PlanT` documentation instead of reproducing its CPS encoding.
- `Process a b` is the one-input `Is a` specialization.
- `source`, `~>`, `run`, `echo`, filtering, taking/dropping, buffering, scans, folds, mapping, flattening, and final-value processes are present.
- `T a b t` preserves the dependent two-input request family. The port keeps Kmett's distinction: `tee` precomposes two processes, while `zipWith`/`zipping` perform deterministic two-input zipping.
- Pure `Mealy` and `Moore` state machines are ported directly, with explicit lazy continuations and adapters back into `Process`.

The Edric spellings used in source are the aliases implemented by the Idriç frontend: `→`, `⇒`, and `←`. Planned notation is not treated as if it were already accepted syntax.

## Behavior gate

`edric/tests/Main.idric` checks characteristic examples from the old library plus the dependent two-input boundary:

```text
source [1,2,3] ~> taking 2 ~> mapping (+1)  ⇒  [2,3]
buffered 3 [1,2,3,4,5]                    ⇒  [[1,2,3],[4,5]]
scan (+) 0 [1,2,3,4,5]                    ⇒  [0,1,3,6,10,15]
fold (+) 0 [1,2,3,4,5]                    ⇒  [15]
zipping [1,2] ["a","b","c"]              ⇒  [(1,"a"),(2,"b")]
tee (mapping (+10)) (filtered (/="b")) ... ⇒  [(11,"a"),(12,"c")]
Mealy counting "word"                       ⇒  [(0,'w'),(1,'o'),(2,'r'),(3,'d')]
Moore running total [1,2,3]                 ⇒  [0,1,3,6]
```

Counts that are `Int` in Haskell are `Nat` here. Negative counts do not describe a useful stream operation, so the Idris port removes them from the API rather than carrying them as runtime edge cases.

## Not ported yet

This is not yet a claim of whole-library parity. The main remaining layers are:

1. `MachineT`, `MealyT`, `MooreT`, and effectful/effect-polymorphic plans.
2. `Wye` and its nondeterministic/concurrent input policy.
3. `Group`, `Fanout`, `Stack`, `Pipe`, `Lift`, and the rest of the higher combinator surface.
4. An Edric-aware package/CI gate that runs these `.idric` files through the Idriç compiler and then executes the regression program.

The pure source in these commits has been checked against the Haskell definitions and doctest behavior, but has not yet been accepted by an Idris 2/Edric compiler in CI. The test program is intended to be that next gate.
