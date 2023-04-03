# AirspeedVelocity

[![Stable](https://img.shields.io/badge/docs-stable-blue.svg)](https://MilesCranmer.github.io/AirspeedVelocity.jl/stable/)
[![Dev](https://img.shields.io/badge/docs-dev-blue.svg)](https://MilesCranmer.github.io/AirspeedVelocity.jl/dev/)
[![Build Status](https://github.com/MilesCranmer/AirspeedVelocity.jl/actions/workflows/CI.yml/badge.svg?branch=master)](https://github.com/MilesCranmer/AirspeedVelocity.jl/actions/workflows/CI.yml?query=branch%3Amaster)
[![Coverage](https://coveralls.io/repos/github/MilesCranmer/AirspeedVelocity.jl/badge.svg?branch=master)](https://coveralls.io/github/MilesCranmer/AirspeedVelocity.jl?branch=master)

AirspeedVelocity.jl makes it easy to benchmark Julia packages over their lifetime.
It is inspired by [asv](https://asv.readthedocs.io/en/stable/).

## Installation

You can install the CLI with:

```bash
julia -e '\
    using Pkg; \
    Pkg.add(url="https://github.com/MilesCranmer/AirspeedVelocity.jl.git"); \
    Pkg.build("AirspeedVelocity")'
```

You may then use the CLI with, e.g.,

```bash
benchpkg Transducers v0.4.65 v0.4.70 master -a BangBang,ArgCheck,Referenceables,SplitApplyCombine
```

which will download `benchmark/benchmarks.jl` of `Transducers.jl`,
run the benchmarks for all revisions given (`v0.4.65`, `v0.4.70`, and `master`),
and then save the JSON results in the current directory.
Here, we also specify `BangBang.jl`, `ArgCheck.jl`, `Referenceables.jl`, and
`SplitApplyCombine.jl` as additional dependencies of the benchmarks.

You can also provide a custom benchmark.
For example, let's say you have a file `script.jl`, defining
a benchmark for `SymbolicRegression.jl`:

```julia
using BenchmarkTools, SymbolicRegression
const SUITE = BenchmarkGroup()
SUITE["eval_tree_array"] = begin
    tree = Node(; feature=1) + cos(3.2f0 * Node(; feature=2))
    X = randn(Float32, 2, 1_000)
    options = Options(; binary_operators=[+, -, *], unary_operators=[cos])
    f() = eval_tree_array(tree, X, options)
    @benchmarkable f() evals=1 samples=100
end
```

we can run this benchmark over the history of `SymbolicRegression.jl` with:

```bash
benchpkg SymbolicRegression v0.15.3 v0.16.2 -s script.jl -o results/ --exeflags="--threads=4 -O3"
```

where we have also specified the output directory and extra flags to pass to the
`julia` executable.
