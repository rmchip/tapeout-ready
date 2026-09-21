# 01 — Read This Paper: Cummings on Clock Domain Crossing

> **Track:** CDC · **Reading time:** the paper is ~50 pages; budget 3–4 sessions · **Prereqs:** [Flip-Flops track](../02-flip-flops/README.md)

There is one document that covers CDC better than any blog, course, or textbook chapter:

> **Clifford E. Cummings, "Clock Domain Crossing (CDC) Design & Verification Techniques Using SystemVerilog," SNUG Boston 2008, Sunburst Design, Inc.**
> 📄 [sunburst-design.com/papers/CummingsSNUG2008Boston_CDC.pdf](https://www.sunburst-design.com/papers/CummingsSNUG2008Boston_CDC.pdf)

This post is deliberately *not* a summary — the paper is self-explanatory and re-explaining it would only dilute it. This is a reading guide: what to make sure you get out of it, because these exact items are what CDC interview questions are built from.

## Must-get checklist

Work through the paper and don't move on until you can explain each of these from memory:

**1. Why CDC fails at all**
The metastability mechanism and why *any* signal crossing asynchronous domains can violate setup/hold. Understand the MTBF equation qualitatively — what makes MTBF better (more settling time, faster flops) and worse (higher clock/data rates).

**2. The 2-FF synchronizer**
The workhorse. Why two flops (first may go metastable; second samples after a full period of settling), why the two flops must be placed close together, and when a third stage is warranted (very high-speed designs where one period of settling isn't enough MTBF).

**3. What a synchronizer does NOT do**
It doesn't remove metastability — it gives it time to resolve, with high probability. It also adds 1–2 destination cycles of latency and can legally output the old *or* new value. Every downstream design decision follows from this uncertainty.

**4. Fast → slow crossings and pulse width**
Why a one-cycle pulse from a fast domain can vanish entirely in a slow domain, the minimum-width requirement (the "three-edge" / ~1.5× receiving-clock-period rule), and the two fixes: stretch the pulse, or use a toggle-based synchronizer with feedback acknowledge.

**5. The multi-bit rule — the single most-asked CDC question**
You cannot put a 2-FF synchronizer on each bit of a bus. Know *why*: per-bit skew means different bits resolve on different edges, so the receiving domain can sample an incoherent mix of old and new values. Then know Cummings' three consolidation strategies for multi-bit crossings: **multi-cycle path (MCP) formulation**, **gray-code encoding** (only one bit changes per transition — safe to sync), and **asynchronous FIFOs**.

**6. Handshake protocols**
Request/acknowledge feedback for crossings in both directions, and the latency cost that comes with them.

**7. Verification angle**
Why simulation alone misses CDC bugs (metastability doesn't happen in RTL sim), and the role of structural CDC lint plus assertions.

## How this maps to interviews

Screens rarely go past items 1–5. Onsite rounds push into 5–6 ("design me a way to move an 8-bit bus between domains" — the expected answer walks through why bit-wise sync fails, then picks MCP, gray code, or FIFO based on throughput). Item 5's FIFO answer is the deepest rabbit hole, which is exactly the next post.

## Interview Check ✍️

1. Your colleague put 2-FF synchronizers on all 16 bits of a bus. What breaks, and what are the three correct alternatives?
2. A 1-cycle pulse crosses from 400 MHz to 100 MHz. What happens, and what are two fixes?
3. Why does a 2-FF synchronizer have unpredictable 1–2 cycle latency, and when does that matter?
4. When would you add a third synchronizer flop?
5. Why can't RTL simulation catch metastability bugs?

---

**Next:** 02 — Asynchronous FIFOs *(coming soon)*
