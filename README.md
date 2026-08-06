# ProbMods

Working through [Mental Models for Probabilistic Programming](https://andrewshadeblevins.com/probability-primer/), a reading companion by Andrew Blevins for writing generative models in Pluck.

The primer is a syllabus roadmap for [ProbMods](https://probmods.org), with coding tasks in [Pluck](https://pluck-lang.github.io) instead of WebPPL (introduced and used in ProbMods). This repo holds my working files: Pluck programs, notes, and answers to the module deliverables.

## Why Pluck

ProbMods is written for WebPPL, which samples and therefore approximates the distributions it reports. Pluck takes the opposite approach: it compiles a program's random choices into a binary decision diagram and computes distributions **exactly**, wherever the query has finite or lazily bounded support.

That difference is mostly invisible early on but it is important later. An exact answer of `1.1e^-13` is the kind of thing sampling wouldn't surface. Conditioning on unlikely observations is the normal case in cognitive modelling, not an edge case. A method that reports zero probability doesn't work for the situations we might care most about. 

## Contents

```lessons.pluck```       Working file for Learn Pluck by Writing Pluck (Lessons 1–10)

```pluck-cheatsheet.md``` Quick reference — forms, arities, gotchas

```deliverables/```       Module answers, drafted here rather than in the browser

The primer's answer boxes save to browser `localStorage` only. Hence keeping the working docs here.

## Setup

The primer's setup section assumes Julia and Rust are already installed. If they aren't, that's two prerequisites you need to work through the lessons.

```bash
# Rust (Pluck's inference engine is a Rust library, compiled locally)
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source "$HOME/.cargo/env"

# Julia
curl -fsSL https://install.julialang.org | sh
```

Then clone Pluck and build the engine:

```bash
git clone --recurse-submodules https://github.com/pluck-lang/Pluck.jl.git
cd Pluck.jl/src/RSDD/rsdd
cargo build --release --features ffi
cd ../../..
```

The `cargo build` takes several minutes and ends on `Finished`.

### If the build fails on `SmallRng`

```
error[E0432]: unresolved import `rand::rngs::SmallRng`
note: the item is gated behind the `small_rng` feature
```

`SmallRng` sits behind an optional feature flag in rand 0.8 that's off by default. Check `src/RSDD/rsdd/Cargo.toml` for:

```toml
rand = { version = "0.8", features = ["small_rng"] }
```

Add the feature if it's missing (`cargo add rand@0.8 --features small_rng`) and rebuild. There's no `Cargo.lock` pinning versions, so a fresh checkout can resolve a dependency set where nothing else switches that flag on.

## Running

Per session, from the `Pluck.jl` folder:

```bash
julia
```

```julia
using Pkg; Pkg.activate("."); using Pluck
load_pluck_file("path/to/lessons.pluck");
```

- The trailing `;` suppresses Julia's echo of every parsed form. Without it you get your query results plus a lot of noise.
- Then it's edit, save, re-run `load_pluck_file`. No need to restart Julia between edits. 
- Definitions accumulate across loads, so restart for a clean slate if a redefinition behaves oddly.
- Pluck code never goes at the `julia>` prompt. Only `load_pluck_file` does.

## Notes

VS Code doesn't recognise `.pluck`. Setting the language to **Clojure** will get you bracket matching and correct `;;` comment handling. I use Scheme but it isn't built in to VSC. Clojure is. 

## Links

- [The primer](https://andrewshadeblevins.com/probability-primer/)
- [Pluck](https://pluck-lang.github.io) · [docs](https://pluck-lang.github.io/docs) · [Pluck.jl](https://github.com/pluck-lang/Pluck.jl)
- [ProbMods](https://probmods.org)
- [Lazy knowledge compilation paper](https://dl.acm.org/doi/10.1145/3729325)
