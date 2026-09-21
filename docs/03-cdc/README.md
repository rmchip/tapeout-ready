# Track 3 — Clock Domain Crossing

CDC has a canonical reference: Cummings' SNUG 2008 paper. This track is built around actually reading it, then going deep on the one structure it leads to — the async FIFO.

## Posts

| # | Post | Topics | Status |
|---|------|--------|--------|
| 01 | [Read This Paper: Cummings on CDC](./01-cummings-cdc-paper.md) | Guided reading of the Sunburst Design CDC paper — metastability, 2-FF sync, fast→slow pulses, multi-bit strategies, MCP, handshakes | ✅ |
| 02 | Asynchronous FIFOs | Dual-clock FIFO design, gray-code pointers, full/empty generation | 📝 |

**Legend:** ✅ published · 📝 planned

## The reference

- [Cummings, "Clock Domain Crossing (CDC) Design & Verification Techniques Using SystemVerilog," SNUG Boston 2008](https://www.sunburst-design.com/papers/CummingsSNUG2008Boston_CDC.pdf) — read it in full; the track's post 01 tells you what to make sure you extract.

**Prev track:** [← Flip-Flops & Timing](../02-flip-flops/README.md)
