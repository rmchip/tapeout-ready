# 01 — Flip-Flop Behavior: Setup, Hold, Skew & Jitter

> **Track:** Flip-Flops · **Reading time:** ~20 min · **Prereqs:** [Combinational Logic track](../01-combinational-logic/README.md)

Every timing question in an ASIC interview traces back to one thing: *what physically happens inside a flip-flop around the clock edge*. So instead of memorizing definitions, we build a flip-flop from pass transistors first — then setup and hold fall out naturally.

## 1. A D flip-flop from transmission gates

A standard positive-edge-triggered D flip-flop is two back-to-back latches — **master** and **slave** — built from transmission gates (TG, a pass-transistor pair) and inverters:

```
                MASTER LATCH                    SLAVE LATCH
           (transparent CLK=0)             (transparent CLK=1)

  D ──► TG1 ──●──► INV1 ──●────► TG2 ──●──► INV3 ──●──► Q
              │           │            │           │
              ▲           │            ▲           │
             INV2 ◄───────┘           INV4 ◄───────┘
          (clocked feedback)       (clocked feedback)

  TG1: ON when CLK=0, OFF when CLK=1
  TG2: OFF when CLK=0, ON when CLK=1
```

**Operation:**
- **CLK = 0:** TG1 is ON → master is *transparent*, D flows through TG1 → INV1. TG2 is OFF → slave holds the old value on Q.
- **CLK rises 0→1:** TG1 turns OFF (master *captures* — the feedback inverter INV2 latches the value), TG2 turns ON → the captured value propagates through the slave to Q.

That handoff at the edge is where setup and hold come from.

## 2. Setup time — from the circuit

For the master to latch the correct value, D must have propagated **through TG1 and INV1, and settled in the INV1–INV2 feedback loop, before TG1 shuts off** at the clock edge.

```
t_setup ≈ delay(TG1) + delay(INV1) + loop-settling margin
```

If D changes later than this, the feedback loop may latch the old value, the new value, or — worst case — hang between them (**metastability**).

**Definition:** setup time (t_su) = minimum time D must be stable *before* the active clock edge.

## 3. Hold time — from the circuit

TG1 doesn't turn off instantaneously — the internal clock (and its inverted copy driving the TG) takes a finite time to switch. If D changes *immediately after* the edge while TG1 is still partially conducting, the new value can corrupt the node being latched.

```
t_hold ≈ time for TG1 to fully turn OFF after the clock edge
         (≈ internal clock inversion delay)
```

**Definition:** hold time (t_h) = minimum time D must remain stable *after* the active clock edge.

```
                      active edge
                          │
  CLK   ──────────────────┌─────────────────
        ──────────────────┘
              ◄── t_su ──►│◄── t_h ──►
  D     ══════ STABLE ════╪═══ STABLE ══════
              (no transitions allowed in this window)
```

## 4. The timing equations (memorize these)

Take a launch flop FF1 driving a capture flop FF2 through combinational logic:

```
        ┌─────┐   comb logic    ┌─────┐
  ──► D │ FF1 │ Q ──► [logic] ──► D │ FF2 │ Q ──►
        └──▲──┘                 └──▲──┘
      launch clk             capture clk
```

**Setup check** (can the data make it in one clock period?):

```
T_cq + T_comb(max) + t_su  ≤  T_clk + T_skew
```

**Hold check** (does the new data arrive too fast and corrupt the current capture?):

```
T_cq + T_comb(min)  ≥  t_h + T_skew
```

where:
- `T_cq` = clock-to-Q delay of the launch flop
- `T_comb` = combinational path delay (max for setup, **min** for hold)
- `T_clk` = clock period
- `T_skew` = capture clock arrival − launch clock arrival (**positive** if capture clock arrives later)

Note what's *not* in the hold equation: **T_clk**. Hold violations are frequency-independent — you cannot fix them by slowing the clock. That's why a hold violation found after tapeout kills the chip, while a setup violation just means shipping a slower part.

## 5. How clock skew affects setup and hold

Skew = difference in clock arrival time between capture and launch flops.

| Skew | Setup | Hold |
|------|-------|------|
| **Positive** (capture clock arrives later) | ✅ Helps — data gets extra time (`+T_skew` of margin) | ❌ Hurts — data must stay stable longer at capture |
| **Negative** (capture clock arrives earlier) | ❌ Hurts — effective period shrinks | ✅ Helps — capture edge already passed when new data races in |

Intuition: positive skew delays the "deadline" (good for setup) but also delays the moment the capture flop stops listening (bad for hold). This is also why tools sometimes insert *useful skew* deliberately to fix setup on critical paths.

## 6. Clock jitter

Jitter = cycle-to-cycle variation in the clock edge position (from PLL noise, supply noise, etc.). Unlike skew (spatial, static), jitter is temporal and random.

- **Setup:** jitter eats directly into your margin — the capture edge might arrive *early* this cycle. Effective setup equation:

```
T_cq + T_comb(max) + t_su  ≤  T_clk + T_skew − T_jitter
```

- **Hold:** for a same-edge check driven by the same clock source, jitter largely cancels (both flops see the same displaced edge), so hold is typically analyzed with skew + OCV margins, not jitter.

## 7. Which edges are the checks run on?

For the default posedge→posedge flop pair:

- **Setup check:** launch edge → the **next** active edge at the capture flop (one full cycle later).
- **Hold check:** launch edge → the **same** active edge at the capture flop (zero cycles).

```
  launch clk  ──┌──┐____┌──┐____┌──┐__
                ▲       ▲
                │       └─ setup checked here (next edge)
                └─ hold checked here (same edge)
```

This is why STA reports show setup slack against the next edge and hold slack against the current edge.

## 8. 🔍 Deep dive — checks between *different* flop types

The edge relationships above change completely when launch and capture flops differ:

- posedge → negedge flop: **half-cycle** setup check
- negedge → posedge, latch → flop, flop → latch, gated clocks…

Each combination moves the setup/hold reference edges. This is a favorite STA interview escalation ("what if the capture flop is negative-edge?"). Work through every combination at the **VLSI Universe** blog — their series on setup/hold checks for different flop and latch pairs is the best free treatment of this: <https://vlsiuniverse.blogspot.com> (search "setup and hold checks" on the blog).

## Interview Check ✍️

1. Draw the master-slave TG flip-flop and point to the exact path that determines setup time. Now point to what determines hold time.
2. A path has T_cq = 0.2 ns, T_comb(max) = 3.1 ns, t_su = 0.15 ns, skew = +0.1 ns. What's the minimum clock period? *(3.35 ns)*
3. Same path, T_comb(min) = 0.05 ns, t_h = 0.2 ns, skew = +0.1 ns. Hold violation? *(Yes: 0.25 < 0.30 — and note slowing the clock won't fix it.)*
4. Your design has setup violations on 5 paths. Name three fixes. Now hold violations on 5 paths — why is buffering the data path the go-to fix there?
5. Why doesn't jitter typically appear in the hold check?

---

**Next:** 02 — Timing Between Different Flop Types *(coming soon)*
