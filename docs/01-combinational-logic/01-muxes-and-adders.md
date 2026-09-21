# 01 — Combinational Logic: Think in Muxes, Build Toward Adders

> **Track:** Combinational Logic · **Reading time:** ~15 min · **Prereqs:** none — this is the starting point

Interviewers don't ask you to recite truth tables. They ask you to *build things from restricted parts* — "make X using only 2:1 muxes" — because it tests whether you understand structure, not memory. This post covers the two skills that matter: mux-based construction and adder fundamentals.

## 1. The 2:1 mux is a universal element

A 2:1 mux computes `Y = S ? I1 : I0`. Every basic gate falls out by wiring constants and signals to the inputs:

```
  NOT A:   I0=1, I1=0, S=A          →  Y = ~A
  AND:     I0=0, I1=B, S=A          →  Y = A·B
  OR:      I0=B, I1=1, S=A          →  Y = A+B
  XOR:     I0=B, I1=~B, S=A         →  Y = A⊕B
```

If you can build NOT and AND (or NOT and OR), you can build anything — the 2:1 mux is universal. Practice deriving each of these on paper; "implement XOR with 2:1 muxes" is a stock screen question.

## 2. Building bigger muxes from 2:1 muxes

An 8:1 mux from 2:1 muxes is a 3-level tree, one select bit per level:

```
  I0 ─┐
      ├─[2:1]─┐  S0
  I1 ─┘       │
  I2 ─┐       ├─[2:1]─┐  S1
      ├─[2:1]─┘       │
  I3 ─┘               ├─[2:1]── Y   S2
  I4 ─┐               │
      ├─[2:1]─┐       │
  I5 ─┘       ├─[2:1]─┘
  I6 ─┐       │
      ├─[2:1]─┘
  I7 ─┘
```

**Count:** 4 + 2 + 1 = **7 muxes**. In general, an N:1 mux needs **N−1** 2:1 muxes across **log₂N** levels.

### The area lesson

Mux count scales linearly with inputs (N−1), but *inputs scale exponentially with select bits*: every extra select bit **doubles** the mux tree. A 4-variable function on a mux needs an 8:1 tree (7 muxes); a 10-variable function would need a 512:1 tree (511 muxes). Mux-based implementation of arbitrary logic blows up exponentially with variable count — that's why synthesis maps to optimized gates, and why FPGAs cap LUTs at 4–6 inputs. Say the word "exponential" in the interview; that's the insight being probed.

## 3. Priority encoder from a mux

A priority encoder outputs the index of the highest-priority active input. For a 4-input encoder (I3 highest), the classic trick is using the inputs themselves as select lines / data:

```
  Y1 = I3 + I2                    (upper bit)
  Y0 = I3 + (~I2 · I1)            (lower bit)
```

Implemented with a 4:1 mux: put `{I3, I2}` on the select lines and hardwire the data inputs to encode the priority outcomes. Work this out on paper — the exercise ("build a priority encoder using a 4:1 mux") forces you to see mux select lines as *decision variables*, not just address bits. And note the same scaling problem: a wide priority encoder built this way inherits the exponential mux-tree growth from §2.

## 4. The full adder

The atom of arithmetic. Three inputs (A, B, Cin), two outputs:

```
  Sum  = A ⊕ B ⊕ Cin
  Cout = A·B + Cin·(A ⊕ B)      [= A·B + B·Cin + A·Cin]
```

Chain N of them, Cout→Cin, and you have an N-bit **ripple-carry adder**:

```
  A0 B0      A1 B1      A2 B2      A3 B3
   │ │        │ │        │ │        │ │
  [FA]──C1──[FA]──C2──[FA]──C3──[FA]──C4
   │          │          │          │
   S0         S1         S2         S3
```

**Worst-case delay:** the carry must ripple through every stage — delay grows **linearly with bit width** (O(N)). A 32-bit ripple adder's critical path is the full carry chain, which is why wide ripple adders are the timing bottleneck in datapaths.

### 🔍 Dig deeper — faster adders

If the ripple carry chain is too slow, you trade area for speed with smarter carry handling: **carry-lookahead** (compute carries in parallel from generate/propagate terms), **carry-select**, **carry-skip**, and parallel-prefix trees like **Kogge-Stone** and **Brent-Kung**. Knowing the *names* and the trade-off (O(N) → O(log N) delay, at higher area) is enough for a first-round screen; deriving them is a follow-on topic. For clear explanations of each architecture, see Wikipedia's [Adder (electronics)](https://en.wikipedia.org/wiki/Adder_(electronics)) and [Carry-lookahead adder](https://en.wikipedia.org/wiki/Carry-lookahead_adder) pages.

## Interview Check ✍️

1. Implement XOR using only 2:1 muxes. Minimum count? *(2 — one to make ~B, one to select.)*
2. How many 2:1 muxes for a 16:1 mux? How many levels? *(15, in 4 levels.)*
3. Why does implementing arbitrary N-input logic in muxes not scale? What's the growth rate?
4. Write Sum and Cout for a full adder from memory. What's the critical path of a 64-bit ripple-carry adder in terms of N?
5. Name two adder architectures that beat ripple carry on worst-case delay, and state the trade-off.

---

**Next track:** [Flip-Flops — Setup, Hold, Skew & Jitter](../02-flip-flops/01-setup-hold-skew-jitter.md)
