# BBD Delay 0.1.0-experimental release resource bounds

Native source: `291cb3bb` (the released folder).
MAIN OS SHA-256: `ffe65ea1f10926c0ffde18d9afe38b346826136d31673d5cc2823781ebca90f1`
(BBD14), the site's build of BBD beside the stock effects and the image the
author played; it equals the native record in `sdk/native-comparisons/bbd.json`.
`bbd.asm` and `bbd_ref.py` are BBD13's; only the placement and metadata changed.

## Software cycle upper bound

Basis: 44.1 kHz, 16 frames/block, eight FX2 instances, four per core.

**Sample work.** `tools/build/cycle_count.py` prices the per-sample loop at
401 modeled cycles with the bucket tick taken and PRE and POST on (both
`bsr bbhsec` sites charged as inlined, plus the call pair). A block holds at
most 16 samples, split or not: `16 * 401 = 6,416`.

**Per-call work**, a union bound over the assembled program (1,044 words):

| part | modeled cycles |
| --- | ---: |
| every word outside the sample loop and the helpers, once: init (70), the per-call prologue with both the clear path and the run path (489), the tail (8) | 567 |
| extra iterations of the fixed loops: init zeroing (90), the clear's three loops (255 + 56 + 10), the three per-call 24-step divides (3 x 23) | 480 |
| the coefficient rotation, up to nine passes after a reset: 8 extra x 45 words | 360 |
| helper bodies: nine `bbsect` calls (46 words + 23 extra divide steps each; `bbpole` is smaller) and one `bbexp2` (23) | 644 |
| control overheads beyond the word count: 17 `do` x 4, 10 `bsr`/`rts` pairs x 6, 15 branches x 4 | 188 |
| **total** | **2,239** |

Rounded up to **2,400 modeled cycles/call**. Charging the clear path and the
run path together, and the reset's nine-section rotation on every call, keeps
it a ceiling for any call.

Two calls per block cover a trigger split. The per-instance block bound is
`6,416 + 2 * 2,400 = 11,216`; four instances cost **44,864/core/block**. The SDK
module allowance is `3,120 * 16 = 49,920`, leaving 5,056 modeled cycles. The
original stock/scheduling reserve is kept outside this allowance. This is a
conservative code-word model, not a whole-instrument contention or wall-clock
guarantee; companion custom effects must fit the remaining budget separately.

For comparison, the matched emulator benchmark (TESTING.md) measures at most
433.31 executed instructions per sample with a trig split at every position.

Commands: `python3 modules/bbd/verify.py` (including the no-op loop marker
proof) and the source census `rg -n '\b(do|rep|bsr|jsr)\b' modules/bbd/bbd.asm`:
no `rep`, no recursion, no helper calls a helper, and no backward loop outside
the priced sample loop other than those counted above.

## Exact reserved memory

Per instance: the stock FX2 allocation, `256 * 3 + 16,384 * 3 = 49,920` bytes.
BBD uses X state `r7+$00..$5a` (91 words) of the 256-word X block, and of the
16,384-word Y buffer the ring (at most 4,095 words) and 58 coefficient words at
`base+$2000..$2039`; the rest is reserved, unused padding.

Shared: two 1,044-word P programs (6,264 bytes; `P:0x01000..0x01414` and
`P:0x00dc0..0x011d4`, in the space of SPRING REV and PLATE REV), the system stack on both
cores (72 bytes, below), the cloned 416-byte FX2 descriptor at `0x400D6B20` and
the 54-byte STGS label formatter at `0x400D6CC0`: **6,806 bytes**. Eight
instances total **406,166 bytes**.

Stack: the deepest nesting is the dispatcher's call, the rotation's `do`, a
`bsr bbsect` and its divide's `do`: six system-stack entries (two 24-bit words
each) per core. The sample loop nests less (call, `do`, `bsr bbhsec`).

DSP numbers are logical 24-bit words; descriptor and formatter counts use byte
accounting. BBD reuses the stock instance buffers and dispatcher stack. There
is no authored heap, SDRAM allocation, ColdFire executable or callback. Its
constants are inline program immediates. `init` leaves r1/n1/m1 unchanged. The
eight-instance guard and dirty-start gates cover software isolation; they do
not claim hardware canaries.
