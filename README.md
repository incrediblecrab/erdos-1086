# Erdős #1086: many triangles of one area

> Let $g(n)$ be minimal such that any set of $n$ points in $\mathbb{R}^2$ contains the vertices of at most $g(n)$ many triangles with the same area. Estimate $g(n)$.

**Status: open**, with $n^2\log\log n\ll g(n)\ll n^{20/9}$. Not solved here.

**Objective.** Sharpen the estimate where it is tractable, reproducing the published grid count first. **Inputs.** The problem page and the sources in [`refs/`](refs/README.md).

**Findings**, with proofs and caveats in [`NOTES.md`](NOTES.md):

- The square grid gives $g(n)\ge(6e^\gamma/\pi^2-o(1))\,n^2\log\log n$, where $6e^\gamma/\pi^2=1.08276\ldots$, and no area does asymptotically better there. The proof is informal and unreviewed, and no novelty is claimed.
- $g(5)=7$ by hand and by certificate, $g(6)=12$ computer-assisted, $18\le g(7)\le21$.
- Exact grid counts up to $800\times800$ give $g(640{,}000)\ge911{,}639{,}642{,}036$.
- The problem page disagrees with its sources in three places.

**Files.** [`src/`](src/README.md) holds the code and build steps, and [`results/`](results/README.md) the claims and data. `LICENSE` (MIT) covers this work only. The quoted problem is from [erdosproblems.com/1086](https://www.erdosproblems.com/1086), maintained by Thomas Bloom.
