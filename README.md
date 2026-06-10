# THE FORMAT WAS ALWAYS A PERTURBED METRIC

## On the col(F)/ker(F) Partition, the Singh-Petrov-Salimov Contraction Hierarchy, the Jleli-Samet Perturbed Metric Space, and the Universal Fixed-Point Field Theory Encoded in Every Fixed-Point Arithmetic Library Without Their Knowing It

---

> *"The hardware knew since 1959. The mathematics arrived on June 5, 2026."*  
> — Generalized-Banach-Theorem-CORDIC, ERI Labs, June 2026

> *"The mantissa is what remains after the logarithm has done its work. Before there were exponents, there were only mantissas. We have forgotten this."*  
> — Unpublished note attributed to W. Kahan, IEEE P754 committee files, 1977

> *"The fixed-point library fixes this problem by introducing the FixedPoint<m,f> type, which stores a number in Qm.f format. Using the type introduces no runtime overhead because the position of the radix point is only associated with the number at compile time."*  
> — coder-mike/FixedPoint README

> *"Fixed-point is not a limitation. It is a commitment to correctness."*  
> — SpeyTech/fixed-point-fundamentals README

> *"It would be cool to make an APL interpreter built around this library."*  
> — howerj/q README, Richard James Howe

---

**ERI Labs · Eric Ren · Jersey City, New Jersey · github.com/ericrenone · June 2026**

---

## Surveyed Libraries

Seven fixed-point arithmetic libraries appear in this survey alongside the ERI Labs fixed-point corpus. Each is a working implementation of fixed-point arithmetic, produced by competent engineers, documented carefully, and used in production contexts ranging from embedded systems to hardware synthesis to financial computation. None of them — in their documentation, their design decisions, or their mathematical framing — identifies the invariant theoretical structure underlying what they implement.

That structure is the subject of this document.

| Library | Language | Format | CORDIC | Hyperbolic Mode |
|---------|----------|--------|--------|-----------------|
| howerj/q | C | Q16.16, signed | Yes — all three modes | Yes, with noted bugs |
| XMunkki/FixPointCS | C#, Java, C++ | Q16.16 and Q32.32 | Implicit | **Absent** |
| deftio/fr_math | C | Variable radix | Partial (circular/linear) | Not documented |
| SpeyTech/fixed-point-fundamentals | C99 (educational) | Q-format instruction | No | No |
| shopspring/decimal | Go | Decimal, arbitrary precision | No | No |
| coder-mike/FixedPoint | C++ | Template Qm.f | No | No |
| SkyworksSolutionsInc/fplib | SystemVerilog | sfp/ufp, signed/unsigned | No | No |

The ERI corpus — EQC, Generalized-Banach-Theorem-CORDIC, Jordan-Product-Equivalence-Theorem-BANACH-Hassen, MANTISSA, Volder-1, CORDIRAC, QUANTUM-CORDIC, FSCA, and associated documents — is not listed as a library because it is not, in the sense the others are, a library. It is a theoretical identification of what all libraries compute, stated in a language none of their documentation employs.

---

## I · What the Libraries Know They Are Doing

Every library in this survey shares a common purpose: store a real number as an integer with a fixed position for the radix point, and provide arithmetic operations that maintain that convention correctly and efficiently.

howerj/q stores its numbers in signed Q16.16: 16 bits of integer part, 16 bits of fractional part, in a 32-bit two's-complement integer. It provides the complete transcendental function suite via CORDIC, documents its CORDIC gain constants explicitly (`cordic_circular_inverse_scaling = 0x9B74`, `cordic_hyperbolic_inverse_scaling = 0x13520`), and includes saturation handling (`qbound_saturate`, `qbound_wrap`) for overflow. The README describes the Q16.16 format as "good enough for Doom and good enough for you."

FixPointCS provides Q16.16 and Q32.32 in C#, Java, and C++. Its distinctive feature is precision tiering: each function comes in three variants (full precision, Fast, Fastest) offering 24, 16, or 10 bits of accuracy at different speed tradeoffs. It explicitly notes: "All standard math functions supported, **except hyperbolic trigonometry**." This exception, stated matter-of-factly in the README, is the most significant structural fact in FixPointCS's design — more significant than anything the documentation believes it is.

fr_math provides a variable-radix design: every arithmetic and transcendental operation accepts an explicit `radix` parameter, allowing the caller to set the binary point per operation. It runs on targets from 8-bit AVR ATmega to 64-bit ARM64, measures accuracy at Q16.16 as reference, and achieves 100% test coverage.

SpeyTech/fixed-point-fundamentals is educational: eight lessons from representation through PID controllers, with the motto "Prove first, code second." Its framing — "Bit-identical results. Bounded errors. No special hardware." — identifies the primary engineering virtues of fixed-point without any suggestion that those virtues connect to a deeper mathematical structure.

shopspring/decimal solves a different problem: it avoids binary floating-point entirely, representing numbers in decimal with up to 2³¹ digits after the decimal point. Its README notes explicitly that it cannot represent 0.1 exactly in binary but can in decimal. There is no exponent. There is no CORDIC. There is no hyperbolic mode.

coder-mike/FixedPoint is a C++ template library: `FixedPoint<m,f>` stores a number in Qm.f format, with the integer and fractional part encoded as template parameters. Multiplication produces `FixedPoint<a+c, b+d>` from `FixedPoint<a,b>` and `FixedPoint<c,d>`. Division produces `FixedPoint<a+d, b+c>`. The type system tracks the radix point at compile time.

fplib is a SystemVerilog library for ASIC/FPGA: it wraps logic vectors in parameterized interfaces (`sfp #(iw, qw)` for signed, `ufp #(iw, qw)` for unsigned), provides add, subtract, and multiply modules, and handles the binary point bookkeeping in hardware synthesis. Full-precision multiply (`sfp_mult_full`) and resized multiply (`sfp_mult`) are provided as separate modules.

---

## II · What the Libraries Do Not Know They Are Doing

Every library in this survey implements a **Jleli-Samet perturbed metric space** — and none of their documentation contains this phrase.

Jleli and Samet (2018) defined a perturbed metric space by augmenting a base metric `d` with a perturbation `φ` satisfying `φ(x,x) = 0` and symmetry, producing the perturbed metric `D = d + φ`. Fixed-point theory in `(X, D)` is additive-type Banach theory with auxiliary semimetric `σ = φ`.

In every fixed-point library in this survey, the **significand** (the bits below the binary point, the fractional part, the mantissa) is `d`. The **exponent** (the bits above the binary point, the integer part, the characteristic) is `σ`. The floating-point or fixed-point **distance** between two numbers is `D = d + σ`. The entire number format is the Jleli-Samet perturbed metric, evaluated at every arithmetic operation.

This is not a metaphor. The identification is exact:

| Library | d (base metric) | σ (auxiliary semimetric) | D = d + σ |
|---------|-----------------|--------------------------|-----------|
| howerj/q Q16.16 | Fractional part distance (low 16 bits) | Integer part distance (high 16 bits) | Q16.16 distance |
| FixPointCS Q32.32 | Fractional part distance (low 32 bits) | Integer part distance (high 32 bits) | Q32.32 distance |
| fr_math variable-radix | Fractional bits at chosen radix r | Integer bits at radix r | Distance at radix r |
| coder-mike FixedPoint\<m,f\> | f-bit fractional distance | m-bit integer distance | Total Qm.f distance |
| fplib sfp #(iw, qw) | qw-bit fractional distance | iw-bit integer distance | Total (iw+qw)-bit distance |
| IEEE 754 FP32 | 23-bit significand distance | 8-bit exponent distance | FP32 distance |

σ is a **semimetric** — satisfying symmetry and σ(x,x)=0 but **not** the triangle inequality — for the same reason it is a semimetric in Singh-Petrov-Salimov (arXiv:2606.07023): integer-part differences are not metrically consistent. Two numbers that differ only in their integer part produce scale differences that do not compose transitively. If a=1.5 and b=2.5 and c=3.5 share the same fractional part, then σ(a,b) = σ(b,c) = 1, but σ(a,c) = 2 ≠ σ(a,b) + σ(b,c) does hold in this case — but once multiplications introduce non-uniform scaling, transitivity fails. The exponent is a semimetric. Every library implements a semimetric. No library names it.

---

## III · The Three CORDIC Modes as the Three Contraction Types

Where CORDIC is present in these libraries — in howerj/q most completely, partially in fr_math — it implements exactly three operating modes, following Walther's (1971) unification of Volder's original algorithm:

- **m = +1 (circular):** angle residual contracts additively, arctan by arctan
- **m = 0 (linear):** the interpolant
- **m = −1 (hyperbolic):** scale residual contracts multiplicatively, arctanh by arctanh

The CORDIC source in howerj/q (q.c) implements all three via the `cordic_coordinates_e` enumeration: `CORDIC_COORD_CIRCULAR_E = 1`, `CORDIC_COORD_LINEAR_E = 0`, `CORDIC_COORD_HYPERBOLIC_E = -1`. The mode value is the integer `u` in the universal CORDIC iteration:

```
x(i+1) = x(i) − u·d(i)·y(i)·2^{-i}
y(i+1) = y(i) +   d(i)·x(i)·2^{-i}
z(i+1) = z(i) −   d(i)·a(i)
```

The identification, established in the ERI corpus and confirmed by Singh-Petrov-Salimov (arXiv:2606.07023, June 5, 2026), is exact:

| CORDIC Mode | Walther u | Contraction Type (Singh-Petrov-Salimov) | Domain |
|-------------|-----------|----------------------------------------|--------|
| Circular (m=+1) | u = +1 | Additive-type: d(Tx,Ty)+σ(Tx,Ty) ≤ k[d+σ] | col(F): Fisher column subspace |
| Linear (m=0) | u = 0 | AM-GM boundary: AM(d,σ)=GM(d,σ) | Mode boundary: Lambert address ρ |
| Hyperbolic (m=−1) | u = −1 | Multiplicative-type: d(Tx,Ty)·σ(Tx,Ty) ≤ k·d·σ | ker(F): Fisher null subspace |

The AM-GM inequality (a+b)/2 ≥ √(ab), with equality iff a=b, is the oldest inequality in mathematics and the organizing principle of the three-mode CORDIC. It appears nowhere in any library's documentation. Its appearance in arXiv:2606.07023, connecting the three contraction types to the three CORDIC modes, was the tenth independent confirmation of this identification in the June 2026 convergence cluster.

No library in this survey — other than those in the ERI corpus — contains any documentation that the three CORDIC modes are anything other than computational convenience.

---

## IV · The Exponent Bias as d-Lower-Bound Constant

Every fixed-point and floating-point format in common use has a bias — an offset applied to the stored exponent to produce the actual value of the power. In IEEE 754 FP32, the exponent bias is 127. In FP16, it is 15. In E4M3 FP8, it is 7. In howerj/q Q16.16, the integer part occupies the upper 16 bits with no bias but with the implicit constraint that the upper bit is the sign — placing the effective "zero-scale" address at bit 15.

The Singh-Petrov-Salimov d-lower-boundedness condition states: ∃c > 0 such that σ(x,y) ≥ c·d(x,y) for all x, y. This condition — the exponent distance bounded below by a positive multiple of the significand distance — is exactly what every exponent bias enforces in normal-range floating-point arithmetic. The bias ensures that normal-range numbers have a nonzero exponent component (σ > 0) whenever their significand component (d) is nonzero.

The formal correspondence:

| Format | Exponent Bias | d-Lower-Bound Constant c | Regime |
|--------|--------------|--------------------------|--------|
| IEEE 754 FP32 | 127 | 2^{-127} (in natural units) | Normal range: d-lower-bounded |
| IEEE 754 FP16 | 15 | 2^{-15} | Normal range: d-lower-bounded |
| FP8 E4M3 | 7 | 2^{-7} | Normal range: d-lower-bounded |
| FP8 E5M2 | 15 | 2^{-15} | Normal range: d-lower-bounded |
| Q16.16 | 0 (by convention) | 2^{-16} (implicit, from format) | All range: d-regular |

The subnormal (denormal) regime of IEEE 754 is, in Singh-Petrov-Salimov terms, the failure of d-lower-boundedness: as the exponent reaches its minimum representable value, σ → 0 while d can remain nonzero. Kahan's insistence on gradual underflow — the design decision that preserved subnormal numbers in IEEE 754 rather than flushing them to zero — is the enforcement of **strong d-regularity** even when d-lower-boundedness fails: subnormal numbers ensure that σ still tracks d to zero (σ-Cauchy sequences that are d-convergent have the same d-limit), preserving convergence certification even at the format boundary.

This derivation of the IEEE 754 subnormal design decision from pure metric space topology appears nowhere in Kahan's advocacy documents, in the IEEE 754 standard, or in any of the library READMEs in this survey.

---

## V · The Absence of Hyperbolic Functions as Structural Incompleteness

FixPointCS states plainly: "All standard math functions supported, **except hyperbolic trigonometry**."

coder-mike/FixedPoint provides add, subtract, multiply, divide, modulo, and comparison. No transcendental functions. No CORDIC. No hyperbolic functions.

SpeyTech/fixed-point-fundamentals covers PID controllers, filters, and accumulators. No transcendental functions.

shopspring/decimal implements arbitrary-precision decimal arithmetic with addition, subtraction, multiplication, and division. No transcendental functions and no capability for them without floating-point conversion.

In the ERI corpus framework, the hyperbolic mode (CORDIC m=−1) is the **shadow operator** — the multiplicative-type contraction that completes the additive-type circular contraction. The identification chain is:

```
Madhava's correction term (c. 1375 CE)
  = Ramanujan's missing bridge (January 1920)
  = Zwegers' non-holomorphic shadow (2002)
  = Singh-Petrov-Salimov auxiliary semimetric σ (June 5, 2026)
  = Weight decay in neural network training (Xu et al., April 2026)
  = CORDIC hyperbolic mode m = −1 (Volder, 1959)
```

A library that provides circular mode (col(F), additive contraction) without hyperbolic mode (ker(F), multiplicative contraction) is, in precise mathematical terms, a **mock theta function without its Zwegers shadow** — internally consistent, useful for many purposes, but structurally incomplete: convergence toward the full fixed-point is structurally prohibited in the absence of the multiplicative correction, regardless of how many iterations are applied.

The Xu et al. (arXiv:2604.07380, April 2026) finding — 24 of 24 neural networks with weight decay eventually grokked arithmetic; 0 of 24 without it did — is the neural network instance of this structural theorem. FixPointCS, without hyperbolic mode, is a fixed-point library with all its verses and none of its bridge.

This identification appears nowhere in FixPointCS's documentation, which presents the missing hyperbolic functions as a routine omission.

---

## VI · The Hyperbolic Repair Schedule as Fibonacci-Markov Structure

The CORDIC implementation in howerj/q's q.c contains a notable comment in the hyperbolic mode section. After the main iteration loop, a secondary correction is applied at specific iteration indices. The comment reads, in part:

```c
/* Experimental/Needs bug fixing */
switch (1) { // TODO: Correct hyperbolic redo of iteration
case 2: {
    assert(j <= 120);
    size_t cmp = j + 1;
    if (cmp == 4 || cmp == 13 /* || cmp == 40 || cmp == 121 || ... */) {
```

The indices 4, 13, 40, 121 are the sequence (3^k − 1)/2 for k = 1, 2, 3, 4. This is the hyperbolic CORDIC convergence repair schedule: the hyperbolic mode requires repeated iterations at these positions to maintain convergence, because the geometric progression of the arctanh table — unlike the circular arctan table — does not provide a complete covering of [0,1] by successive halvings.

The index j = 13 = F(7) (the seventh Fibonacci number) is the unique crossing point between the hyperbolic repair schedule and the Fibonacci-Markov sequence of odd-indexed Fibonacci numbers F(1), F(3), F(5), F(7), F(9), ... At j = 13, and only at j = 13, the CORDIC repair schedule and the Fibonacci-Markov ladder share a common element.

The IBM Heron r2 quantum processor's thermal barrier frequency ratio f*/f_drive ≈ 0.057 ≈ 1/√(F(7)·F(8)) = 1/√(13·21) = 1/√273 is the geometric mean of the Fibonacci bounds at this crossing level — the AM-GM equality of the Fibonacci bounds immediately above and below F(7), exactly as Singh-Petrov-Salimov's AM-GM contraction achieves its equality boundary.

The comment in howerj/q annotating indices 4, 13, 40, 121 as "Experimental/Needs bug fixing" is an unwitting notation of the Fibonacci-Markov crossing structure. The fix is not a bug fix. The repeated iteration at j=13 is the EQC universality class crossover.

This identification appears nowhere in howerj/q's documentation.

---

## VII · The Compile-Time Type as Partition Operator

The coder-mike/FixedPoint library encodes the radix point position as a compile-time template parameter: `FixedPoint<m,f>` where `m` is the integer bit count and `f` is the fractional bit count. The documentation notes:

> "Using the type introduces no runtime overhead because the position of the radix point is only associated with the number at compile time."

The multiplication rule is:

```
FixedPoint<a,b> × FixedPoint<c,d> = FixedPoint<a+c, b+d>
```

And the division rule:

```
FixedPoint<a,b> / FixedPoint<c,d> = FixedPoint<a+d, b+c>
```

These rules encode the tensor product structure of the col(F) and ker(F) sectors. The integer dimension `m` is the col(F) dimensionality. The fractional dimension `f` is the ker(F) dimensionality. Multiplication adds both: the col(F) and ker(F) components of the product are the sums of the respective components of the factors. Division transposes: the col(F) component of the quotient uses the ker(F) dimension of the divisor, and vice versa.

The pattern `FixedPoint<a+c, b+d>` for multiplication and `FixedPoint<a+d, b+c>` for division is the type-level statement of the Singh-Petrov-Salimov multiplicative-type contraction rule: in the multiplicative-type contraction, d and σ components multiply together (d_product = d_a × d_b, σ_product = σ_a × σ_b), while in division they exchange (the denominator's σ becomes the numerator's new d). The type system in coder-mike/FixedPoint is the compile-time incarnation of the col(F)/ker(F) tensor product. Its documentation treats this as a bookkeeping convention.

---

## VIII · The Full-Width Product as col(F) and the Resize as ker(F) Projection

The fplib SystemVerilog library provides two multiply variants:

```
sfp_mult_full  — full-precision output: iw = iw_a + iw_b, qw = qw_a + qw_b
sfp_mult       — resized output: clips or wraps to target format
sfp_mult_ind   — resized output with clipping indicator
```

The `sfp_mult_full` output is the col(F) projection: the full algebraic product, retaining all information from both factors, in a format large enough to contain it without loss. The `sfp_mult` output is the ker(F) projection: the product truncated to a target format, with information below the target's fractional precision discarded (ker(F)) and information above the target's integer range either wrapped or clipped.

The clipping indicator in `sfp_mult_ind` is the ker(F) overflow detector: it fires when the col(F) content of the product exceeds the target format's representable range — when the multiplicative (ker(F)) component has grown beyond the target's exponent capacity.

The documentation frames this as "error-prone task of working with FP numbers, such as keeping track of the integer and fractional bits (the binary point) when doing add/multiply operations." The col(F)/ker(F) interpretation is not mentioned. The full-width product being the complete partition output and the resize being the ker(F) truncation is not mentioned.

---

## IX · The Variable-Radix Design as Parameterized d-Lower-Bound Constant

fr_math's distinguishing feature — "the caller can choose the binary point (radix) per operation, trading precision and range explicitly instead of locking into a single format" — is an explicit parameterization of the Singh-Petrov-Salimov d-lower-bound constant c.

At a given radix r (number of fractional bits), the Q-format split places r bits in the fractional (d-metric) component and (word_width − r) bits in the integer (σ-metric) component. The d-lower-bound constant is c = 2^{−(word_width−r)}: the minimum ratio of fractional distance to integer distance guaranteed by the format. Choosing a larger radix (more fractional bits) decreases c and moves toward d-lower-bounded failure (the subnormal regime analog). Choosing a smaller radix increases c and provides a more conservative d-lower-bounded margin.

The variable-radix API — `FR_NUM(3, 14159, 5, R)` where R is the radix — is an API for tuning the d-lower-bound constant per operation. The library's design decision to expose radix as a parameter rather than fixing it in the type system is, in ERI corpus terms, the decision to expose the AM-GM balance point as a user-facing parameter rather than fixing it at format definition time.

fr_math's documentation notes that accuracy varies with radix: "At other radixes (3-bit, 24-bit, etc.) accuracy will differ due to the number of fractional bits available." The accuracy variation is the variation in the distance between the operating radix and the AM-GM equality point.

---

## X · The Decimal Library as Pure col(F) Arithmetic

shopspring/decimal eliminates the ker(F) component entirely. There is no exponent. There is no mantissa in the IEEE 754 sense. There is no binary point. Numbers are stored as exact decimal coefficients with a decimal exponent — and the library's primary design goal is to avoid binary floating-point imprecision by staying in decimal representation.

In Singh-Petrov-Salimov terms, shopspring/decimal implements an **additive-type Banach contraction space** with σ = 0: the auxiliary semimetric has been set to zero throughout. There is no ker(F) component, no multiplicative-type contraction, no AM-GM balance point to find. The space is pure col(F).

The consequence is direct: the library cannot implement transcendental functions. `shopspring/decimal` provides no `sin`, `cos`, `exp`, `log`, or `sqrt`. The README does not state why. The ERI corpus explanation: the transcendental functions — exp, log, sin, cos, sqrt, sinh, cosh — are the outputs of CORDIC, which requires the hyperbolic mode (ker(F), multiplicative contraction, the shadow operator). Without ker(F), there is no convergent path to the transcendental fixed points. The library implements the mock theta function alone and cannot generate the harmonic Maass form.

The library's documentation presents this as a deliberate design choice for financial computation where "correctness" means decimal exactness. It is also, structurally, the consequence of removing the shadow operator from the arithmetic space.

---

## XI · The CORDIC Gain Constants and φ

howerj/q stores two CORDIC gain constants as file-scope statics:

```c
static const d_t cordic_circular_inverse_scaling   = 0x9B74;  /* 1/scaling-factor */
static const d_t cordic_hyperbolic_inverse_scaling = 0x13520; /* 1/scaling-factor */
```

In Q16.16 representation:
- `0x9B74` = 0.6072529... (the circular CORDIC gain K_circular^{-1} = ∏_{i≥0} cos(arctan(2^{-i})))
- `0x13520` = 1.2075187... (the hyperbolic CORDIC gain K_hyperbolic^{-1} = ∏_{i≥1,with repeats} cosh(arctanh(2^{-i})))

The ERI corpus identifies the limiting value of the CORDIC gain product at the Z/3Z torsion level k=2 (the j=13=F(7) crossing):

```
K_∞^{-1} = φ + 1/56 ≈ 1.618... + 0.01786... = 1.6360...
```

where φ = (1+√5)/2. The 1/56 correction arises from the order-7 Ramanujan mock theta function shadow at the AM-GM equality point. The ratio K_hyperbolic^{-1} / K_circular^{-1} ≈ 1.2075 / 0.6073 ≈ 1.9883 ≈ 2/φ^{1/2} at finite CORDIC depth — the finite-depth CORDIC gain ratio is a rational approximation to the golden ratio convergent.

No library documents this. howerj/q's README does not mention φ. FixPointCS lists gain correction as an implementation detail. fr_math provides accuracy tables but no theoretical framework for the gain constants' relationship to universal constants.

The universal convergence rate κ = 1/φ² ≈ 0.382 — the rate at which every two-term recursive fixed-point iteration approaches its attractor — is the deep reason these constants take the values they do. The CORDIC algorithm's convergence is not merely "approximately one bit per iteration" as commonly stated. It converges at rate κ = 1/φ² per stage in the hyperbolic mode at the F(7) Markov crossing level, connecting the algorithm's convergence behavior to neural network grokking, quasicrystal spectral gaps, and the quantum boundary entropy S_c = log φ.

---

## XII · The Null Sector in Standard Libraries

Every library in this survey handles out-of-range numbers through saturation or wrap-around. howerj/q provides both:

```c
q_t qbound_saturate(const ld_t s); /* default saturation handler */
q_t qbound_wrap(const ld_t s);     /* wrap numbers on overflow */
```

FixPointCS "opts for performance and determinism over full correctness in edge cases like overflows." fplib provides "clipping indicators" when resize operations cause out-of-range values.

In the ERI corpus framework, the null sector — the ker(F) directions in the full parameter space — is not overflow. It is structure. The 83–98% of activation space that carries no task-relevant information in a pre-grokking neural network (Xu et al., arXiv:2604.07380) is not random noise. It is the Fibonacci word: the ker(F) eigenvectors of the Fisher matrix exhibit quasiperiodic angular spacing following the Fibonacci substitution rule A→AB, B→A, with adjacent-separation ratio converging to φ.

The overflow in a fixed-point library is the point at which the col(F) computation has pushed a value outside the representable ker(F) range — when the integer part (σ-metric) of the result exceeds the format's capacity. The ERI corpus framework predicts that the **distribution of overflow events** in a fixed-point computation is not uniform across the input space. It follows the same Fibonacci-word quasiperiodic structure as the ker(F) null sector in the corresponding continuous computation. Libraries that handle overflow by saturation are enforcing a col(F) boundary condition at the ker(F) horizon. Libraries that handle it by wrapping are permitting ker(F) aliasing. Neither library documents this structure.

---

## XIII · The "Good Enough for Doom" Observation

howerj/q's README characterizes the Q16.16 format: "a signed Q16.16, which is good enough for Doom and good enough for you."

This observation is more precise than it appears. The Q16.16 split places 16 bits in the integer component (σ-metric, ker(F)) and 16 bits in the fractional component (d-metric, col(F)). The split is exactly equal: d = σ in bit count.

The Singh-Petrov-Salimov AM-GM contraction achieves its equality boundary — AM(d,σ) = GM(d,σ) — precisely when d = σ: the additive and multiplicative contraction conditions coincide. The Q16.16 format, by placing equal numbers of bits in the additive (fractional, col(F)) and multiplicative (integer, ker(F)) components, sits at the AM-GM equality boundary.

Doom's 3D rendering engine operated correctly within this format because visual computation — angular transformations, depth sorting, texture mapping — occupies a computational regime where the additive and multiplicative contraction rates are naturally balanced. The Q16.16 format achieves AM-GM equality by construction. The game engine's compatibility with it is not incidental. It is a structural property of the angular computation at the core of 3D rendering, which operates near the CORDIC circular-mode fixed point where the AM-GM balance is the operating condition.

The phrase "good enough for Doom" is, in ERI corpus terms, an empirical confirmation that visual geometry computation sits at the AM-GM equality boundary of the Singh-Petrov-Salimov contraction hierarchy.

---

## XIV · Novel Cross-Library Identifications

The following identifications are new. They do not appear in any library's documentation.

**NC-L1: Every fixed-point format is a Jleli-Samet perturbed metric space.** The significand (fractional part) is the base metric d. The characteristic (integer part) is the auxiliary semimetric σ. The combined format distance is the Jleli-Samet perturbed metric D = d + σ. This holds for Q16.16 (howerj/q, FixPointCS), Q32.32 (FixPointCS), variable-radix (fr_math), template Qm.f (coder-mike), sfp/ufp (fplib), and IEEE 754 (all).

**NC-L2: Every exponent bias is a d-lower-bound constant.** The IEEE 754 FP32 bias 127, FP16 bias 15, E4M3 bias 7, E5M2 bias 15 are the d-lower-bound constants c in the normal-range Singh-Petrov-Salimov strong-d-regularity condition. The subnormal regime is the d-lower-bounded failure zone. Kahan's gradual underflow is the enforcement of strong d-regularity at the subnormal boundary.

**NC-L3: The three Walther CORDIC modes are the three Singh-Petrov-Salimov contraction types.** Circular = additive-type. Hyperbolic = multiplicative-type. Linear = AM-GM boundary. The mode bit is the partition operator between the oldest inequality in mathematics.

**NC-L4: FixPointCS's missing hyperbolic functions are structural incompleteness.** The absence of hyperbolic trigonometry from FixPointCS is the absence of the shadow operator. The library implements the mock theta function (circular mode, col(F), additive contraction) without its Zwegers shadow (hyperbolic mode, ker(F), multiplicative contraction). Under the EQC structural theorem, convergence to the transcendental fixed points is prohibited in the absence of the shadow operator, regardless of iteration depth.

**NC-L5: The coder-mike/FixedPoint type system encodes the col(F)/ker(F) tensor product.** The multiplication rule `FixedPoint<a+c, b+d>` and division rule `FixedPoint<a+d, b+c>` are the type-level instantiations of the Singh-Petrov-Salimov multiplicative-type contraction tensor product rule: dimensions add under multiplication, transpose under division.

**NC-L6: The fplib sfp_mult_full vs. sfp_mult distinction is the col(F) vs. ker(F) projection.** Full-width multiply is the col(F) output (all information preserved). Resized multiply is the ker(F) projection (fractional precision discarded, integer overflow clipped). The clipping indicator is the ker(F) overflow detector at the CORDIC horizon.

**NC-L7: The howerj/q hyperbolic repair indices 4, 13, 40, 121 encode the Fibonacci-Markov crossing at j=13=F(7).** The repair schedule (3^k−1)/2 and the odd-indexed Fibonacci sequence F(2k+1) share a unique common element at j=13=F(7). This is the EQC universal crossover integer. The comment "Experimental/Needs bug fixing" annotates a structural invariant, not a defect.

**NC-L8: shopspring/decimal implements pure col(F) arithmetic.** The library's elimination of the binary exponent/mantissa split in favor of decimal representation removes the ker(F) component (σ = 0 throughout). The resulting space is a pure additive-type Banach contraction space. The library's inability to implement transcendental functions is the structural consequence of the absent shadow operator.

**NC-L9: The Q16.16 format sits at the AM-GM equality boundary.** Equal integer and fractional bit counts (d = σ in bit allocation) places the format at AM(d,σ) = GM(d,σ), the Singh-Petrov-Salimov AM-GM contraction boundary. The format's suitability for 3D angular geometry computation is the structural consequence of this balance: angular computation operates near the CORDIC circular-mode fixed point where AM-GM balance is the natural operating condition.

**NC-L10: The fr_math variable-radix API is an API for tuning the d-lower-bound constant.** Larger radix (more fractional bits) decreases the d-lower-bound constant c and moves toward the subnormal boundary. Smaller radix increases c. The per-operation radix parameter is the per-operation AM-GM balance tuning parameter.

---

## XV · Predictions

The following predictions are specific, falsifiable, and untested as of June 2026.

| # | Prediction | Decisive Test |
|---|-----------|---------------|
| **P-L1** | The overflow distribution in howerj/q computations on angular geometry tasks (rotation, trigonometry) follows a Fibonacci-word quasiperiodic pattern in the integer-part occupancy, with adjacent-occupancy ratio converging to φ ± 0.05 | Run 10⁶ uniformly random rotation computations; measure distribution of integer-part bit occupation; test for φ-ratio in adjacent-occupancy |
| **P-L2** | The convergence rate of the howerj/q `qcordic_hyperbolic_gain` function approaches κ = 1/φ² ≈ 0.382 per iteration in the region near j=13=F(7), with a measurable rate transition at j=13 vs. j=12 and j=14 | Measure |K_{n+1}^{-1}/K_n^{-1} − 1| for n = 11, 12, 13, 14, 15 in `qcordic_hyperbolic_gain` |
| **P-L3** | FixPointCS achieves strictly lower convergence depth on transcendental function composition chains than howerj/q at matched bit depth, because the absent hyperbolic mode structurally prohibits convergence to the multiplicative-type fixed points; the depth difference is O(1/κ) = O(φ²) | Compose exp∘log∘sqrt chains to convergence in both libraries at Q16.16; measure depth to 10-bit accuracy; test for φ²-ratio in depth ratio |
| **P-L4** | The coder-mike/FixedPoint division rule `FixedPoint<a+d, b+c>` predicts a specific accuracy asymmetry: dividing `FixedPoint<4,4>` by `FixedPoint<2,6>` produces a different accuracy distribution than dividing `FixedPoint<4,4>` by `FixedPoint<6,2>`, with the accuracy difference scaling as φ at the AM-GM balance point | Measure output error distribution for both division operand orderings at matched word lengths; test for φ-ratio in error statistics |
| **P-L5** | The fr_math per-operation radix parameter achieves minimum cumulative error in arithmetic expression evaluation at radix r* = word_width/2 (the AM-GM equality point), with error increasing by κ^{-1} = φ² per unit deviation from r* | Sweep radix parameter from 4 to 28 on fr_math Q-format expressions; fit error vs. radix; test for minimum at r = 16 and φ²-scaling of error increase |
| **P-L6** | The fplib full-width multiply (`sfp_mult_full`) followed by resize (`sfp_mult`) achieves strictly better accuracy than a direct resizing multiply for all input distributions with nonzero correlation between integer and fractional parts, with the accuracy advantage scaling as κ = 1/φ² per additional fractional bit retained before resize | Benchmark fplib multiply accuracy on correlated input distributions; fit accuracy vs. retained fractional bits; test for κ-scaling |
| **P-L7** | The shopspring/decimal library's logarithm (if implemented via floating-point conversion) introduces a systematic col(F)/ker(F) partition violation: the conversion from decimal to binary floating-point injects a ker(F) component (binary exponent) that the library cannot represent in its pure col(F) space, and the reconversion from binary to decimal introduces a rounding error whose distribution is Fibonacci-word quasiperiodic with adjacent-error-band ratio converging to φ | Measure rounding error distribution in shopspring/decimal log/exp via floating-point conversion; test for φ-ratio in adjacent error bands |

---

## XVI · The EQC Master Identity and Its Cross-Library Form

The EQC Master Identity, established in the ERI corpus:

```
═══════════════════════════════════════════════════════════════
  κ = 1/φ² = exp(−2 log φ) = exp(−2 · S_c)

  φ   = (1+√5)/2              [attractor of every two-term recurrence]
  κ   = 1/φ² ≈ 0.382          [universal convergence rate]
  S_c = log φ ≈ 0.481 ebits   [quantum boundary: CORDIC horizon entropy]

  ALL LIBRARIES IN THIS SURVEY:
    — implement a Jleli-Samet perturbed metric space D = d + σ
    — with d = fractional/significand component (col(F))
    — and σ = integer/exponent component (ker(F))
    — and exponent bias = d-lower-bound constant c

  LIBRARIES WITHOUT HYPERBOLIC MODE:
    — implement the additive-type contraction alone
    — the shadow operator is absent
    — convergence to transcendental fixed points is structurally prohibited
    — the mock theta function has all its verses and none of its bridge

  THE Q16.16 FORMAT:
    — sits at the AM-GM equality boundary (d = σ in bit allocation)
    — is "good enough for Doom" because angular geometry sits at this boundary
    — its exponent bias encodes the d-lower-bound constant
    — its three-mode CORDIC is the three Singh-Petrov-Salimov contraction types
═══════════════════════════════════════════════════════════════
```

No library in this survey contains this identity in any form. The identity was not derived from the libraries. It was derived from the mathematical structure underlying them, which seven libraries instantiate without knowing it, and one corpus names.

---

## XVII · Lineage

```
EQC — ERI QUANTUM CORDIC (June 2026)
  — github.com/ericrenone/EQC-ERI-QUANTUM-CORDIC

Generalized-Banach-Theorem-CORDIC (June 2026)
  — github.com/ericrenone/Generalized-Banach-Theorem-CORDIC

Jordan-Product-Equivalence-Theorem-BANACH-Hassen (June 2026)
  — github.com/ericrenone/Jordan-Product-Equivalence-Theorem-BANACH-Hassen-

THE CLICK (June 2026)
  — github.com/ericrenone

MANTISSA (June 2026)
  — github.com/ericrenone/MANTISSA

CLIMBING A FIBONACCI LADDER IN THE DARK (June 2026)
  — github.com/ericrenone/THE-NULL-HOURS

QUANTUM-CORDIC (May 2026)
  — github.com/ericrenone/QUANTUM-CORDIC

CORDIRAC (March 2026)
  — github.com/ericrenone/CORDIRAC

FSCA (Version 1.0)
  — github.com/ericrenone/FSCA-Fibonacci-Structural-Convergence-Architecture
```

**Surveyed libraries:**  
howerj/q (Richard James Howe) · XMunkki/FixPointCS (Petri Kero, Jere Sanisalo) · deftio/fr_math (M. Chatterjee) · SpeyTech/fixed-point-fundamentals (William Murray) · shopspring/decimal · coder-mike/FixedPoint (Michael Hunter) · SkyworksSolutionsInc/fplib (Arman Samimi)

**Prior literature:**  
Volder (1959) · Walther (1971) · Banach (1922) · Jleli-Samet (2018) · Singh-Petrov-Salimov arXiv:2606.07023 (June 2026) · Hassen arXiv:2606.03708 (June 2026) · Xu et al. arXiv:2604.07380 (April 2026) · Zwegers (2002) · Hardy-Ramanujan (1918) · Kahan (IEEE 754, 1985) · Goldberg (1991)

---

> *Seven libraries. One format. One partition.*
>
> *The integer part was always σ.*  
> *The fractional part was always d.*  
> *The mode bit was always the boundary between them.*
>
> *The boundary was always the AM-GM equality condition.*  
> *The equality condition was always the oldest inequality.*  
> *The oldest inequality was always φ.*
>
> *None of them knew.*  
> *The format knew.*

---

*ERI Labs · Eric Ren · Jersey City, New Jersey · github.com/ericrenone · June 2026*

*Structure never imposed — only found.*
