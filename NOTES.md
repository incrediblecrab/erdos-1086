# Erdős #1086: working notes

Everything here was done on September 23, 2026, on an Apple M3 Pro (11 cores) under macOS 27.0, with Apple clang 21.0.0, Python 3.14.7, sympy 1.14.0 and numpy 2.5.3. Sources are cited by the tags in [`refs/README.md`](refs/README.md).

**Notation.** The *doubled area* of a triangle $pqr$ is $|\det(q-p,\,r-p)|$, twice its area; for lattice points it is a positive integer. $G_m=\{0,\dots,m-1\}^2$ is the $m\times m$ grid, $n=m^2$, and $T_N(m)$ is the number of triangles of $G_m$ with doubled area $N$. The code and `results/claims.json` write the doubled area as $D$. $\sigma(N)$ is the sum and $d(N)$ the number of divisors of $N$; $\gamma$ is Euler's constant; $\log$ is natural.

## 1. The record

The problem page, fetched September 23, 2026 (it reports "last edited 16 October 2025"), states: "Let $g(n)$ be minimal such that any set of $n$ points in $\mathbb{R}^2$ contains the vertices of at most $g(n)$ many triangles with the same area. Estimate $g(n)$." It is labelled **open**, "This is open, and cannot be resolved with a finite computation", with no comments, no proof claims and "Formalised statement? No". The google-deepmind/formal-conjectures repository had no file for 1086 when checked the same day; the commit was not recorded.

The recorded bounds are $n^2\log\log n \ll g(n) \ll n^{20/9}$. The lower bound is [ErPu71, Theorem 2], read from the page image of p. 249 because the scan's OCR layer is unusable. It takes $a=[\sqrt{\log n}]$ rows of about $n/a$ integral points each and counts triangles of area $\tfrac12 a!$, i.e. doubled area $a!$. The result is $g^{(2)}_2(n)\geqslant cn^2\log\log n$ with an unspecified $c$. The upper bound is [RaSh17, Theorem 1]. The chain before it is $O(n^{5/2})$ [ErPu71], $O(n^{7/3})$ [PaSh92], $O(n^{44/19})$ [DST09], $O(n^{9/4+\varepsilon})$ [ApSh10] and $O(n^{9/4})$ [Ap13]. [PaSh92] and [Ap13] were not read; their bounds are as [DST09] and [RaSh17] report them.

Three discrepancies between the page and its sources:

1. **Area zero.** Read literally, "triangles with the same area" includes degenerate ones: $n$ collinear points give $\binom n3$ triples of area $0$, so $g(n)=\binom n3$. The page's next sentence ("how many triangles of area $1$") and [Er75f, p. 104], which says "non zero", fix the intended reading. Everything here counts nonzero areas only.
2. **"Apfaulbaum [Ap13]".** The author is Roel Apfelbaum, as on [ApSh10] and in [RaSh17]'s reference [1].
3. **[ErPu71, p. 248].** The page image reads "The same method shows that $G^{(k)}_{2k}(kn)\geqslant n^k$ and $g^{(k)}_{2k+2}(kn+1)\geqslant n^{k+1}$", followed by the conjecture $g^{(k)}_{2k+2}(n)=\frac{n^{k+1}}{(k+1)^{k+1}}(1+o(1))$. As printed, the first bound gives $g^{(k)}_{2k+2}(N)\gtrsim N^{k+1}/k^{k+1}$, which exceeds the conjectured value, so the two cannot both hold. The problem page states the bound as $\bigl(\frac1{(k+1)^{k+1}}+o(1)\bigr)n^{k+1}$, consistent with the conjecture. Which line is misprinted is inferred, not stated in the source.

**Literature checked for the constant in §3.** [ErPu71], [Er75f], [DST09], [ApSh10] and [RaSh17] each state the lower bound only as $\Omega(n^2\log\log n)$ or $c\,n^2\log\log n$ with an unspecified constant (a loose grep of each text for "log log", with every hit read). Two web searches on September 23, 2026 found no explicit constant either. The first answer only restated the query and cited nothing that supports it; the second returned only the bound history above. OEIS and Semantic Scholar sweeps earlier the same day found nothing that improves either bound. Not read: [PaSh92], [Pu74], [Ap13], and Brass, Moser and Pach's *Research Problems in Discrete Geometry* (2005). Absence from what was read is not evidence that the constant is new, and no novelty is claimed.

## 2. Reproducing the published grid count first

[DST09, Theorem 3] states that the $\lfloor\sqrt n\rfloor\times\lfloor\sqrt n\rfloor$ integer grid spans $(6/\pi^2-o(1))n^2$ minimum-area triangles, about $0.6079n^2$. This is a paraphrase: the arXiv text layer scrambles the formula, and the page was not rendered. In $\mathbb Z^2$ the minimum nonzero doubled area is $1$, so this count is $T_1(m)$. `src/grid_count` gives:

| $m$ | 25 | 50 | 100 | 200 | 400 | 800 |
|---|---|---|---|---|---|---|
| $T_1/n^2$ | 0.60334 | 0.60705 | 0.60758 | 0.60787 | 0.60791 | 0.60793 |

$6/\pi^2=0.607927$, and at $m=800$, $T_1=249{,}006{,}655{,}884$ and $T_1/n^2=0.607926$. This is the evidence that the engine measures the published quantity. The six histogram totals also equal OEIS A045996, the number of triangles in the $m\times m$ grid, and $\binom{m^2}3$ minus the collinear triples.

## 3. The square grid: Theorems A and B

**Status: informal proofs, written for this directory. Not machine-checked and not reviewed by anyone else.** The upper-bound step reuses the charging of [DST09, Lemma 1], which assigns each triangle to a longest side.

For $u\in\mathbb Z^2\setminus\{0\}$ let $\gcd(u)=\gcd(|u_x|,|u_y|)$. Let $P_a(m)$ be the number of unordered pairs $\{p,q\}\subset G_m$ with $\gcd(q-p)=a$.

**Lemma 1 (apexes).** Let $p\neq q$ in $\mathbb Z^2$, $u=q-p$, $a=\gcd(u)$ and $N\ge1$. The $w\in\mathbb Z^2$ with $|\det(u,w-p)|=N$ exist iff $a\mid N$, and then they fill two lattice lines parallel to $u$. Put $t(w)=(w-p)\cdot u/|u|^2$, so $p+t(w)u$ is the foot of the perpendicular from $w$. On each line exactly $a$ points have $t\in[0,1)$, and at most $a+1$ have $t\in[0,1]$.

*Proof.* Write $u=au'$ with $u'$ primitive. Then $\det(u,w-p)=a\det(u',w-p)$, and $w\mapsto\det(u',w-p)$ maps $\mathbb Z^2$ onto $\mathbb Z$ with fibres that are translates of $\mathbb Zu'$. So the solutions are the fibres over $\pm N/a$, and these exist iff $a\mid N$. Moving along a fibre by $u'$ changes $t$ by $u'\cdot u/|u|^2=1/a$, so on each line the $t$-values form a coset of $\frac1a\mathbb Z$. ∎

**Lemma 2 (pairs).** $P_a(m)=\frac3{\pi^2}\frac{m^4}{a^2}+O(m^3/a)$ uniformly for $1\le a<m$, and $P_a(m)=0$ for $a\ge m$.

*Proof.* Let $Q_b$ count pairs with $b\mid\gcd(q-p)$, that is, pairs in the same residue class mod $b$. Write $m=kb+\rho$ with $0\le\rho<b$. Each coordinate class then has $k$ or $k+1$ members, and $s_b=\sum_j c_j^2=m^2/b+\theta$ with $0\le\theta\le b/4$. So $Q_b=\frac12(s_b^2-m^2)=\frac{m^4}{2b^2}+E_b$, with $|E_b|\le m^2/2$ for $b\le m$, and $Q_b=0$ for $b\ge m$. Möbius inversion over multiples gives $P_a=\sum_{k<m/a}\mu(k)Q_{ak}=\frac{m^4}{2a^2}\sum_{k<m/a}\frac{\mu(k)}{k^2}+O(m^3/a)$, and the sum is $\frac6{\pi^2}+O(a/m)$. ∎

**Proposition 3 (upper).** For all $m,N\ge1$: $T_N(m)\le\sum_{a\mid N}2a\,P_a(m)=\frac6{\pi^2}\,m^4\,\frac{\sigma(N)}N+O(m^3d(N))$.

*Proof.* Charge each triangle to a longest side $\{p,q\}$. The angles at $p$ and $q$ are acute, so the third vertex has $t\in(0,1)$. By Lemma 1 there are at most $2a$ such vertices, with $a=\gcd(q-p)\mid N$. Summing Lemma 2 over $a\mid N$ gives $\frac6{\pi^2}m^4\sum_{a\mid N}\frac1a+O(m^3d(N))$, and $\sum_{a\mid N}\frac1a=\sigma(N)/N$. ∎

**Proposition 4 (lower).** If $3N<m$, then $T_N(m)\ge\frac6{\pi^2}\,m^4\,\frac{\sigma(N)}N-O(m^3\sigma(N))$.

*Proof.* Let $G'=\{N,\dots,m-1-N\}^2$, a copy of $G_{m'}$ with $m'=m-2N>N$. Orient each pair of $G'$ whose gcd $a$ divides $N$. By Lemma 1, exactly $2a$ lattice points $w$ have doubled area $N$ and $t(w)\in[0,1)$. Each lies within $N/|u|\le N$ of the segment $pq\subset[N,m-1-N]^2$, so it is in $G_m$. The number of such (pair, $w$) is $I=\sum_{a\mid N}2aP_a(m')$. Every divisor of $N$ is below $m'$, so Lemma 2 gives $I=\frac6{\pi^2}m'^4\frac{\sigma(N)}N+O(m^3d(N))$.

A triangle arises at most three times, once per side. An obtuse triangle arises at most once: for either side at the obtuse vertex, the foot of the third vertex falls outside the closed side. So $T_N\ge I-2\,\mathrm{NO}_N$, where $\mathrm{NO}_N$ counts non-obtuse triangles of doubled area $N$.

In such a triangle, let $\ell$ be the shortest side, opposite angle $A$, and let $B\ge C$ be the other two angles. The height onto $\ell$ is $h=\ell\sin B\sin C/\sin A\ge\ell\sin B$, because $A\le C\le90^\circ$. Since $60^\circ\le B\le90^\circ$, this gives $h\ge\frac{\sqrt3}2\ell$ and so $\ell^2\le2N/\sqrt3$. Orient the shortest side. Both base angles are at most $90^\circ$, so $t\in[0,1]$, and Lemma 1 allows at most $2(\gcd(s)+1)$ third vertices. Hence $\mathrm{NO}_N\le m^2\sum_{0<|s|^2\le2N/\sqrt3}2(\gcd(s)+1)=O(m^2N(1+\log N))$.

Finally $m'^4\ge m^4-8Nm^3$, $d(N)\le\sigma(N)$, and $N(1+\log N)\le m\,\sigma(N)$. ∎

**Theorem A.** For fixed $N$, $T_N(m)/m^4\to\frac6{\pi^2}\frac{\sigma(N)}N$ as $m\to\infty$. This follows from Propositions 3 and 4. For $N=1$ it is the limit behind [DST09, Theorem 3].

**Theorem B.** As $m\to\infty$, $\max_{N\ge1}T_N(m)=\bigl(\frac{6e^\gamma}{\pi^2}+o(1)\bigr)\,n^2\log\log n$ with $n=m^2$. Hence

$$g(n)\ \ge\ \Bigl(\frac{6e^\gamma}{\pi^2}-o(1)\Bigr)\,n^2\log\log n,\qquad \frac{6e^\gamma}{\pi^2}=1.08276\ldots$$

*Proof.* **Lower.** Take $N=\operatorname{lcm}(1,\dots,y)$ with $y=\lfloor\frac12\log m\rfloor$. By the prime number theorem, $\log N=\psi(y)=(1+o(1))y$, so $N=m^{1/2+o(1)}$ and $3N<m$ for large $m$. Since $\sigma(N)/N\le1+\log N$, the error in Proposition 4 is $o(m^4)$. Each prime $p\le y$ divides $N$ to the largest power $p^e\le y$. So $\frac{\sigma(N)}N=\prod_{p\le y}\frac{1-p^{-e-1}}{1-1/p}\ge(1-\frac1y)^{\pi(y)}\prod_{p\le y}(1-\frac1p)^{-1}$. Mertens' product [Gr13, p. 117] gives $\prod_{p\le y}(1-1/p)^{-1}\sim e^\gamma\log y$, and $(1-\frac1y)^{\pi(y)}\to1$. With $\log y=\log\log n-\log4+o(1)$ this gives the lower bound. **Upper.** $T_N(m)=0$ for $N>(m-1)^2$. Gronwall's theorem [Gr13, eq. (25)], $\limsup\sigma(N)/(N\log\log N)=e^\gamma$, gives $\max_{N<n}\sigma(N)/N\le(e^\gamma+o(1))\log\log n$. Also $d(N)=O_\varepsilon(N^\varepsilon)$, so the error in Proposition 3 is $o(m^4)$. **Non-square $n$.** $g$ is nondecreasing, since adding a point destroys no triangle. With $m=\lfloor\sqrt n\rfloor$, $m^4=n^2(1-O(n^{-1/2}))$ and $\log\log m^2=\log\log n+o(1)$. ∎

**What this does and does not say.** The order $n^2\log\log n$ is Erdős and Purdy's. Theorem B supplies an explicit constant for the square grid. Its upper half says that no single area does better in the square grid, so beating $1.0828\,n^2\log\log n$ needs a different point set. Nothing here touches the upper bound $n^{20/9}$.

**Convergence is very slow (measured, §6).** $T/(n^2\log\log n)$ at the best $N$ rises from 0.70 to 0.86 over $m=25,\dots,800$, still well below 1.0828. There are two visible reasons. First, at the best $N$ the count is 5 to 12 % below $\frac6{\pi^2}\frac{\sigma(N)}Nn^2$; in all 41 cells of the fixed-$N$ table with $N\ge2$ and $N/n\le0.1$, $1-\text{ratio}$ lies between $3.3\,N/n$ and $5.9\,N/n$. That is an empirical fit, not proved, and it fails for larger $N/n$. Second, $\sigma(N)/N$ at the best $N$ lags $e^\gamma\log\log n$: at $m=800$ it is 3.838 at $N=5040$, against 4.618.

## 4. $g(5)=7$ and $g(6)=12$

**$g(5)=7$, by hand.** *Lower bound:* $R=\{(0,0),(1,0),(0,1),(1,1),(1,-1)\}$ spans 7 triangles of doubled area 1. Its full profile is 7 of doubled area 1, 2 of doubled area 2 and 1 collinear triple.

*Upper bound:* suppose 5 points span 8 triangles of one nonzero area. Each triangle lies in 2 of the 5 four-point subsets, and $2\cdot8>5\cdot3$. So some four points $A,B,C,D$ span 4 triangles of that area. No three of them are collinear, and none lies inside the triangle of the other three, whose area would then be a sum of three others. So they are in convex position, say in the order $ABCD$. Then $[ABC]=[ABD]$ with $C,D$ on the same side of the edge $AB$ forces $CD\parallel AB$, and likewise $AD\parallel BC$, so $ABCD$ is a parallelogram.

An affine map takes it to $(0,0),(1,0),(1,1),(0,1)$ with doubled area 1. A fifth point $(x,y)$ needs 4 of the six values $|y|,|y-1|,|x|,|x-1|,|x-y|,|x+y-1|$ to equal 1. No two in the pairs $\{|y|,|y-1|\}$ and $\{|x|,|x-1|\}$ can both be 1, so $x-y=\pm1$ and $x+y-1=\pm1$ must both hold. The four solutions all have $x=\frac12$ or $y=\frac12$, which rules out a whole axis pair. So at most 3 of the six equal 1, giving at most $4+3=7$, a contradiction. ∎

*Independent certificate:* `src/g5_certificate.py 8`. It maps one of the triangles to $(0,0),(1,0),(0,1)$. That leaves $\binom97=36$ choices of seven more triangles and $2^7$ sign vectors: 4608 polynomial systems in the other two points. Every Gröbner basis over $\mathbb Q$ is $[1]$, so none has a solution even over $\mathbb C$. The control `g5_certificate.py 7` finds consistent systems and a real configuration spanning 7.

**$g(6)=12$, computer-assisted** (`src/g6_exact.py`; exact arithmetic, 3 s).

1. *Classification.* The $K=7$ systems number $\binom96\cdot2^6=5376$, and 63 are consistent. For each, sympy's `solve` and `solve_poly_system` must return the same finite set. Its size must also equal $\dim_{\mathbb Q}\mathbb Q[x_3,y_3,x_4,y_4]/I$, read off a grevlex Gröbner basis, so no complex solution is missed. The 42 real solutions with five distinct points are each mapped onto $R$ by an explicit affine bijection. So every 5-point set with 7 triangles of one nonzero area is an affine copy of $R$.
2. *Reduction.* Take six points with $t\ge13$ such triangles. Each triangle lies in 3 of the 6 five-subsets, and $3t\ge39>36$, so some five-subset has 7 and is an affine copy of $R$ with doubled area 1. At least 6 of the 10 triangles through the sixth point $X$ then have $\det=\pm1$. No direction occurs among more than 4 pairs of $R$, so $X$ lies on two non-parallel such lines.
3. *Enumeration.* All 50 intersection points are tried in exact rationals. The largest count of doubled-area-1 triangles in $R\cup\{X\}$ is 12, at $X=(0,-1)$ and $(0,2)$; both make an affine copy of the $2\times3$ grid. That grid has 12, so $g(6)=12$.

`src/g6_brute.py` shares no code with `g6_exact.py`. It lists the possible denominators $\{1,2,3\}$ and scans every $X\in\frac16\mathbb Z^2$ with $|x|,|y|\le16$ (37,244 points) in integer arithmetic. It finds the same maximum at the same two points. During development three planted defects each made the check fail: a wrong $R$, one dropped solution (caught by the quotient-dimension test), and a shrunken brute-force box (maximum 10). The result rests on sympy's Gröbner bases and solvers. The cross-checks guard against missed solutions, but this is not a formal proof.

**$g(7)$ is not settled: $18\le g(7)\le21$.** Seven points with 19 would need a six-subset with 11, since $4\cdot19>7\cdot10$. Classifying those needs five-point sets with 6, whose systems can be positive-dimensional. This was not attempted.

## 5. Small $n$

`src/small_table.py` ran in 2 min 18 s. It uses two methods:

- `src/small_search` searched exhaustively over every $n$-subset of the $5\times5$ box ($n\le10$) and the $6\times6$ box ($n\le12$).
- Simulated annealing ran in the $10\times10$ box for $n=4,\dots,16$: 1M iterations, 4 restarts, seeds 1 to 3, and target doubled areas $\{1,2,3,4,6,8,12\}$. A second annealing pass used the $12\times12$ box with 4M iterations and seeds 4 to 6 for $n=11,\dots,16$. That makes 399 annealing runs.

Every run's configuration was recounted by `src/recount.py` before it could be kept. As a positive control, both exhaustive boxes rediscover $g(4)=4$, $g(5)=7$ and $g(6)=12$. Annealing reaches every exhaustive box optimum, and the second pass reproduced every first-pass value without improving any.

The upper bounds come from averaging: each triangle of an $n$-set lies in $n-3$ of its $(n-1)$-subsets, so $g(n)\le\lfloor g(n-1)\,n/(n-3)\rfloor$, started from $g(6)=12$. They are weak.

| $n$ | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 | 15 | 16 |
|---|---|---|---|---|---|---|---|---|---|---|
| lower bound | 18 | 28 | 36 | 48 | 60 | 74 | 91 | 108 | 128 | 148 |
| its doubled area | 1 | 2 | 2 | 2 | 2 | 2 | 2 | 2 | 2 | 2 |
| found by | both | both | both | both | both | both | anneal | anneal | anneal | anneal |
| upper bound | 21 | 33 | 49 | 70 | 96 | 128 | 166 | 211 | 263 | 323 |

For $n=7$ the configuration is the $2\times3$ grid plus $(1,2)$; for $n=8$ it is the $3\times3$ grid minus its centre. The rest are in `results/small_n.json`. The exhaustive values are optimal only within their box. An optimal set may need a larger box or non-lattice coordinates, so every lower bound for $n\ge7$ is only that. No published table of small values was found; an OEIS search for 1, 4, 7, 12, 18 returned four unrelated sequences.

## 6. Finite lattice data

`src/lattice_table.py` writes `results/lattice.json`, and `src/verify_lattice.py` rechecks it. One full run took 12 min 41 s wall-clock on 10 threads on a shared machine. Every count is exact, and each best count is a certified lower bound on $g(n)$ at that $n$. For example, $g(640{,}000)\ge911{,}639{,}642{,}036$.

**Square grids at the best doubled area $N$:**

| grid | $n$ | best $N$ | $\sigma(N)/N$ | $T_N/n^2$ | ratio to $\frac6{\pi^2}\frac{\sigma(N)}N$ | $T_N/(n^2\log\log n)$ |
|---|---|---|---|---|---|---|
| 25×25 | 625 | 12 | 2.333 | 1.3072 | 0.922 | 0.7020 |
| 50×50 | 2,500 | 60 | 2.800 | 1.5014 | 0.882 | 0.7298 |
| 100×100 | 10,000 | 120 | 3.000 | 1.7110 | 0.938 | 0.7706 |
| 200×200 | 40,000 | 360 | 3.250 | 1.8797 | 0.951 | 0.7963 |
| 400×400 | 160,000 | 2,520 | 3.714 | 2.0572 | 0.911 | 0.8284 |
| 800×800 | 640,000 | 5,040 | 3.838 | 2.2257 | 0.954 | 0.8584 |

**Theorem A at fixed $N$.** The table gives $T_N/(\frac6{\pi^2}\frac{\sigma(N)}Nn^2)$, which should tend to 1:

| $N$ | $m=25$ | 50 | 100 | 200 | 400 | 800 |
|---|---|---|---|---|---|---|
| 1 | 0.9925 | 0.9986 | 0.9994 | 0.9999 | 1.0000 | 1.0000 |
| 2 | 0.9864 | 0.9965 | 0.9991 | 0.9997 | 0.9999 | 1.0000 |
| 6 | 0.9617 | 0.9898 | 0.9974 | 0.9994 | 0.9998 | 1.0000 |
| 12 | 0.9215 | 0.9799 | 0.9949 | 0.9987 | 0.9997 | 0.9999 |
| 60 | 0.6027 | 0.8820 | 0.9721 | 0.9934 | 0.9984 | 0.9996 |
| 120 | 0.3473 | 0.7720 | 0.9382 | 0.9857 | 0.9967 | 0.9992 |
| 360 | 0.0241 | 0.4522 | 0.8172 | 0.9514 | 0.9882 | 0.9974 |
| 840 | 0.0000 | 0.1468 | 0.6228 | 0.8857 | 0.9706 | 0.9928 |
| 2520 | 0.0000 | 0.0000 | 0.2414 | 0.6970 | 0.9111 | 0.9773 |

**Shape at $n\approx40{,}000$, and the [ErPu71] strip.** Best $T_N/n^2$ for rectangles:

| grid | 200×200 | 100×400 | 40×1000 | 20×2000 | 10×4000 | 5×8000 | 3×13333 |
|---|---|---|---|---|---|---|---|
| best $N$ | 360 | 360 | 360 | 360 | 120 | 12 | 2 |
| $T_N/n^2$ | 1.8797 | 1.8714 | 1.8046 | 1.6791 | 1.4401 | 1.1059 | 0.7777 |

At every $n$ computed here, $\lfloor\sqrt{\log n}\rfloor=3$, so the [ErPu71] construction is a 3-row strip with doubled area $3!=6$. Its $T_6/n^2$ is 0.7768 at $n=9{,}999$, 0.7775 at $39{,}999$ and 0.7777 at $159{,}999$. The best area in the strip, $N=2$, is marginally higher. The square beats the strip by 2.42× at $n\approx40{,}000$ and 2.65× at $n\approx160{,}000$. This is a finite-$n$ comparison only. The strip's asymptotic constant was not derived, and its row count grows with $n$. Reading $\log$ as $\log_2$ would give 4 rows at $n=159{,}999$. No 4-row strip was computed. The 3-, 5- and 10-row rectangles at $n\approx40{,}000$ all do far worse than the square: 0.78, 1.11 and 1.44 against 1.88.

## 7. How the numbers are checked

`src/final_check.py` re-derives every entry of `results/claims.json`, runs 49 checks in about a minute, and exits 0 only if all pass:

- `grid_count` must match the independent triple enumeration `src/brute_check.py` bin by bin at 12 sizes. These go up to 30×30 (120,464,616 triangles) and 17×53 (120,727,840).
- `src/brute_count.c`, a second triple enumeration, written in C and sharing no code or method with `grid_count`, must reproduce the stored 25×25, 50×50 and 100×100 histograms byte for byte.
- `grid_count` must rebuild five of the histograms to the SHA-256 that `lattice.json` records. `verify_lattice.py` must pass on all 16 runs, which includes checking each raw file against that hash.
- The $g(5)$ certificate and its control must pass, as must `g6_exact.py` and `g6_brute.py` and the recount of every small-$n$ configuration.
- Each claimed value is then compared with what those steps produced. Theorems A and B are reported as NOT MACHINE-CHECKED.

The gate plants four defects of its own, and each must make the component exit 1:

- 6 extra triangles in one bin, against `brute_check`;
- 6 extra triangles at $D=12$ in the 25×25 histogram with its hash updated, against `verify_lattice`;
- a wrong reference configuration, against `g6_exact`;
- an overstated small-$n$ count, against `recount`.

Separately, a copy of the directory was given three false claims in `claims.json`: $g(6)=13$, $19\le g(7)$, and one extra triangle at $n=640{,}000$. The same copy had 6 triangles added at $D=7$ in the stored 100×100 histogram. The gate exited 1, with 44 of 49 checks passing. It named the three claims, the `brute_count` comparison, and `verify_lattice`, which found that file's hash, its total, and its total against A045996 all wrong. The rebuild check passed, correctly: it compares `grid_count`'s output with the recorded hash, not with the file.

**Known blind spot.** `verify_lattice.py` checks totals, the best and fixed-$N$ bins, and hashes. A planted move of 6 triangles between two unchecked bins (7 and 11), with the hash updated, **passed**. Five of the 14 stored grids have every bin confirmed independently. The gate has `brute_count` reproduce the 25×25, 50×50 and 100×100 grids. One-off runs logged in `results/brute_count.log`, and not gate-checked, reproduced the 200×200 grid (28 min) and the 3×3333 strip byte for byte. The other nine were not brute-forced: 400×400, 800×800, the five other rectangles at $n=40{,}000$, and the strips at $n=39{,}999$ and $159{,}999$. The cost grows as $n^3$, so 400×400 would take an estimated 21 to 30 hours at the rates measured here. For those nine, per-bin correctness rests on `grid_count`'s agreement at the smaller sizes, on every raw count being divisible by 6, which `grid_count` enforces, and on the totals.

## 8. Not done

- No Lean formalization. formal-conjectures has no statement to target, and none was written.
- Theorems A and B are informal and unreviewed. The constant has not been checked against the unread literature in §1.
- The asymptotics of the [ErPu71] strip were not derived.
- $g(7)$ remains between 18 and 21. The small-$n$ lower bounds come from searches in small boxes.
- The problem itself remains open.
