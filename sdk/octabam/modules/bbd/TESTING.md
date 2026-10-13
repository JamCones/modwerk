# BBD Delay testing

The software evidence here comes from the octabam native toolchain
(`dsp_asm`, `dsp_host`) and a private image built from my own OS 1.40C MAIN
OS. The author also played images BBD01 to BBD14 on an Octatrack MKII; see
[Hardware](#hardware). Firmware, images, dumps and renders stayed outside the
repository.

## Commands and exact revision

Gate source: commit `536fb51b`. Every
run below used that `bbd.asm`, `manifest.py` and `bbd_ref.py`. The run tree is
a disposable copy of this repository's `sdk/octabam` with the module at
`modules/bbd` and `vendor/` plus `out/raw/section_3_MAIN_OS.bin` linked from a
native octabam checkout. Before each run I checked that its `dsp_asm` assembles
`move x:(r7+$15),a` as the one-word `0257de`.

```sh
python3 modules/bbd/verify.py                         # render gates on a private image
python3 tools/harness/benchmark_stock_dsp.py          # stock baseline, 4,096 blocks
python3 modules/bbd/perf.py bench  <out>               # matched benchmark
python3 modules/bbd/perf.py stress <out> --seconds 32  # stress
npm run fx:audit -- plan <dir>; npm run fx:audit -- check <dir> <renders>
```

The private image is the site's own build of BBD beside the stock effects: BBD
in the space of SPRING REV and PLATE REV, every other stock effect kept
(SHA-256 `ffe65ea1f10926c0ffde18d9afe38b346826136d31673d5cc2823781ebca90f1`,
the BBD14 image, equal to `sdk/native-comparisons/bbd.json`; stock MAIN OS
`164f31224bf61181e3f50e7dec40df9afcae5b16dbf6e4c0d0cc5e986af0a84e`).
BBD sits at `P:0x01000..0x01414` on payload A and `P:0x00dc0..0x011d4` on B:
1,044 words with 613 to spare. SPRING REV alone would hold it, but DARK REV
calls a 35-word routine at SPRING REV+820; BBD starts at PLATE REV's base and
ends before it. Images BBD01–BBD13 used SPRING REV alone and kept DARK REV
listed: there DARK REV ran into BBD's code (the `stock` gate fails on that
layout, about 95 % of output words different). None of them was published.

## Render gates (`verify.py`)

All pass:

```
[PASS] source: init leaves r1/n1/m1 alone
[PASS] source: static cycles: 401 cycles/sample per instance (the tick path), 4 per core = 1604 of 3120; marker no-op True
[PASS] placement: payload A: BBD P:0x01000..0x01414 (1044 words) in P:0x01000..0x01679
[PASS] placement: payload B: BBD P:0x00dc0..0x011d4 (1044 words) in P:0x00dc0..0x01439
[PASS] dry: MIX 0 is the input sample for sample (static): 0 of 48000 words differ
[PASS] dry: MIX 0 is the input sample for sample (moving knobs): 0 of 48000 words differ
[PASS] dry: MIX 0 is the input sample for sample (moving knobs, split 5): 0 of 48000 words differ
[PASS] reference: TIME 0: -73.0 dB against bbd_ref (limit -60), peak -8.1 dBFS
[PASS] reference: TIME 64: -70.3 dB against bbd_ref (limit -60), peak -8.2 dBFS
[PASS] reference: TIME 127: -57.1 dB against bbd_ref (limit -50), peak -8.3 dBFS
[PASS] reference: TIME 32, STGS 1024: -70.9 dB against bbd_ref (limit -60), peak -8.1 dBFS
[PASS] reference: TIME 96, STGS 1024: -59.0 dB against bbd_ref (limit -50), peak -8.2 dBFS
[PASS] reference: MIX 64: -74.5 dB against bbd_ref (limit -60), peak -5.5 dBFS
[PASS] reference: FDBK 80: -72.7 dB against bbd_ref (limit -40), peak -8.1 dBFS
[PASS] reference: FDBK 90, TONE 127: -72.5 dB against bbd_ref (limit -40), peak -8.1 dBFS
[PASS] reference: FDBK 70, TONE 0: -73.0 dB against bbd_ref (limit -40), peak -8.1 dBFS
[PASS] reference: FDBK 40, TONE 0, FREQ 0: -73.2 dB against bbd_ref (limit -40), peak -8.1 dBFS
[PASS] reference: FDBK 40, TONE 0, FREQ 127: -72.8 dB against bbd_ref (limit -40), peak -8.1 dBFS
[PASS] reference: FDBK 40, FRZ 64: -69.9 dB against bbd_ref (limit -40), peak -13.5 dBFS
[PASS] reference: FDBK 100, TONE 0, LIM 120: -73.0 dB against bbd_ref (limit -40), peak -8.1 dBFS
[PASS] reference: MIX 32: -78.8 dB against bbd_ref (limit -60), peak -6.9 dBFS
[PASS] reference: MIX 96: -72.2 dB against bbd_ref (limit -60), peak -6.7 dBFS
[PASS] reference: PRE ON: -65.8 dB against bbd_ref (limit -60), peak -8.3 dBFS
[PASS] reference: POST ON: -72.5 dB against bbd_ref (limit -60), peak -8.0 dBFS
[PASS] reference: PRE and POST ON: -70.4 dB against bbd_ref (limit -60), peak -8.1 dBFS
[PASS] splits: split 1 matches the model split alike: -72.4 dB
[PASS] splits: split 7 matches the model split alike: -72.4 dB
[PASS] splits: split 15 matches the model split alike: -72.4 dB
[PASS] stages: STGS 4096 -> 1024 -> 4096 mid-signal matches the model: -69.8 dB through two fades and clears
[PASS] freeze: FRZ 127 mid-signal matches the model: -71.1 dB
[PASS] freeze: the frozen loop sustains: peak -11.1 dBFS 1 s after the freeze, -11.1 dBFS at the end
[PASS] freeze: a 0.5 s crossfade to FRZ 127 keeps a self-oscillating loop's level: -35.3 dBFS RMS before, -35.0 frozen (+0.4 dB; LIM 127, TONE 0)
[PASS] freeze: FRZ 100 with TONE 0 and FDBK 20 lengthens the trails without droning: -97.7 dBFS RMS 8.5-10 s after a 0.25 s burst (limit -70)
[PASS] idle: FDBK 90, the last 0.5 s of a 9.5 s tail: -114.4 dBFS
[PASS] idle: FDBK 10, TONE 0, the last 0.5 s of a 9.5 s tail: -132.5 dBFS
[PASS] idle: FDBK 127, TONE 0, LIM 0 self-oscillates, bounded below full scale: peak -0.33 dBFS, last 0.5 s -2.8 dBFS RMS
[PASS] idle: FDBK 127, TONE 0, LIM 127 self-oscillates, bounded below full scale: peak -8.11 dBFS, last 0.5 s -32.5 dBFS RMS
[PASS] idle: LIM 127 settles the oscillation at least 8 dB under LIM 0 (off): 29.6 dB quieter
[PASS] isolation: eight instances under -guard 0x4000 -guard-shared -dirty 0x5a: 8 instances guarded, 0 stray regions, 0 clobbers
[PASS] isolation: each instance alone renders bit-identically: 8 of 8 identical
[PASS] garbage: on silence, a garbage block renders exactly as a zeroed one, below -100 dBFS: 4 fills x 2 knob sets, 600 blocks, Y dirty too; loudest sample 2 LSB (-132.5 dBFS): the bucket noise
[PASS] garbage: a tone from a garbage block starts like a zeroed one: first 256 blocks peak within 1 dB
[PASS] zipper: no block-rate step on any continuous knob: 7 knobs, then TONE inside the loop; jump up / jump down / turn, move/static dBFS above; flagged none
[PASS] stock: DARK REV on core A (fixed) renders as on the stock OS: 0 of 176384 words differ
[PASS] stock: DARK REV on core A (moving) renders as on the stock OS: 0 of 176384 words differ
[PASS] stock: DARK REV on core B (fixed) renders as on the stock OS: 0 of 176384 words differ
[PASS] stock: DARK REV on core B (moving) renders as on the stock OS: 0 of 176384 words differ
```

Notes on the limits:

- **Slow clocks.** The cutoff is 1.7 kHz there, and 24-bit DF1 state noise in
  the anti-alias and reconstruction cascades sits about 56 dB under the
  signal. The residual is broadband below the cutoff, and its in-phase gain
  error is −66 dB, so it is noise, not a response difference.
- **Feedback cases.** These get −40 dB because feedback recirculates every
  difference; they measured −72 to −74 dB.
- **Idle.** The FDBK 10 case runs at TONE 0. That boost is +9 dB; measured on
  `bbd_ref.py`, a full TONE (either end, any FREQ) rings on from FDBK ~39-43,
  and TONE centred keeps a 60 dB tail past 6 s from FDBK ~103.
- **Garbage start.** It is judged as `verify_dirtystate.py` judges it: a
  garbage start must render exactly as a zeroed start and stay below −100 dBFS.
  For about 0.2 s after a reset, the bucket noise, lifted by the
  compander's warm start, leaves at most 2 LSB from either start.

## Sound quality (`fx:audit`)

Rendered wet only (MIX 127, FDBK 0, TONE 64, STGS 4096) on BBD12; BBD13 renders
these bit-identically (FRZ 0). The audit assembles BBD into the stock image
itself, so BBD14's placement does not change it. Against the
same renders on BBD11 the in-band rows move by up to 1 dB, and the rows for
tones the filters remove (100 dB and more down) by a few dB: the shorter clear
starts the bucket clock 16 blocks sooner, so its edges sample each test tone at
different points. The idle row is unchanged. Not at the "dearest"
knobs: FDBK 127 with TONE 0 self-oscillates by design and would swamp the
tones. The CPU cost does not depend on the knobs.

BBD, TIME 0 (clock 20.5 kHz, cutoff 6.8 kHz):

```
signal                  out dBFS  harmonics  aliasing  residual   DC dBFS   rails  verdict
tone-1100hz-12db           -12.4      -44.8     -44.1     -25.2     -90.4       0  FAIL
tone-1100hz-1db             -3.1      -24.3     -44.0     -25.1     -85.0       0  FAIL
tone-2701hz-12db           -13.3      -55.5     -24.2     -17.5     -91.6       0  FAIL
tone-2701hz-1db             -4.1      -45.0     -24.5     -17.6     -96.1       0  FAIL
tone-5301hz-12db           -18.1      -68.0     -20.3      -8.1    -103.2       0  FAIL
tone-5301hz-1db             -8.8      -68.1     -20.3      -8.1    -100.8       0  FAIL
tone-9102hz-12db          -103.5      -69.1      20.1      23.3    -124.1       0  FAIL
tone-9102hz-1db            -93.5      -79.8      19.9      22.9    -120.4       0  FAIL
idle                    after the input stops: idle activity of 2 LSB value(s), peak -132.5 dBFS  NOTE
```

BBD, TIME 127 (clock 5.1 kHz, cutoff 1.7 kHz):

```
signal                  out dBFS  harmonics  aliasing  residual   DC dBFS   rails  verdict
tone-1100hz-12db           -24.4      -52.5     -37.0      -9.5     -92.6       0  FAIL
tone-1100hz-1db            -14.7      -52.4     -36.5      -9.3     -85.4       0  FAIL
tone-2701hz-12db          -114.4      -52.9     -12.8       2.9    -139.5       0  FAIL
tone-2701hz-1db           -103.5      -63.9      -7.2       7.5     <-140       0  FAIL
tone-5301hz-12db          -114.3      -64.0     -31.7     -25.3     <-140       0  FAIL
tone-5301hz-1db           -103.4      -75.8     -32.8     -21.4     <-140       0  FAIL
tone-9102hz-12db          -114.3      -76.1     -32.5     -32.7     <-140       0  FAIL
tone-9102hz-1db           -103.4      -75.5     -15.9     -23.5    -139.7       0  FAIL
idle                    after the input stops: idle activity of 2 LSB value(s), peak -132.5 dBFS  NOTE
```

The aliasing limit (−60 dBc) fails, and that is intended: a bucket brigade is
a sampler clocked at 5–20 kHz, and the charge-transfer curve bends the held
samples, so its harmonics fold at the clock rate. Tones above the clock-third
cutoff are removed: their "dBc" compares leftovers with a fundamental 80–140 dB
down. DC passes everywhere (−80 to −139 dBFS against −60), nothing touches
the rails, and idle is 2 LSB of bucket noise.

## Performance

`perf.py bench` runs E-Verb's method: `benchmark_reverbs.run` as
`benchmark_stock_dsp.py` runs it, one instance per core at positions 0 and 4,
audio at X:0, 4,096 blocks, the null stub (67) subtracted. It covers BBD on the
private image and SPRING REV, CHORUS and DJ EQ on the stock image, through the
same cases: fixed, the moving-knob pattern (it reaches every STGS) and a trig
split at every position 1–15. BBD adds the dearest settings with splits and 18
TIME/STGS/FDBK corners. The unsplit stock figures reproduce
`benchmark_stock_dsp.py` exactly.

| effect | worst unsplit | worst with splits |
| --- | ---: | ---: |
| **BBD** | **406.38** | **433.31** (moving, split 1) |
| SPRING REV | 257.50 | 314.19 |
| CHORUS (closest stock in function: a BBD-style line) | 253.62 | 293.06 |
| DJ EQ (the dearest stock effect) | 293.38 | 330.94 |

Executed instructions per sample, not chip cycles. The static loop is 401
cycles (`cycle_count.py`, the tick path, PRE and POST on), so four per core
take 1,604 of the 3,120 usable. BBD is 1.38× SPRING REV and 1.48× CHORUS with
splits (1.58× and 1.60× unsplit), and dearer than every stock effect. At the
defaults (PRE and POST OFF, fixed knobs) it measures 370.38. BBD13's FRZ adds a
24-step divide a call (sustain for TONE's peak gain): about 4 instructions a
sample over BBD12, and one fewer static cycle (the per-bucket mix is gone).

The extra buys full filter orders: 8th-order anti-alias and 9th-order
reconstruction, both tracking the clock. They are about 90 words of the
401-cycle loop, counted from the source. It also buys a 24-step divide a sample
for the compander, with the Padé tanh folded into it.

Three changes cut the cost:

- FDBK and TONE now glide once a call instead of every sample. That saved 23
  cycles (368 to 345, measured before FRZ, LIM, the MIX curve and PRE/POST
  brought the loop to 415).
- One filter section per call follows the clock instead of all nine. That takes the per-call coefficient
  work from ten divisions to two.
- A bookkeeping pass with bit-identical output took the static loop from 415
  to 402 and the worst case from 437.88 to 427.19: TONE's bandpass now runs in
  the filters' parallel-move form from a per-call copy of its coefficients in
  Y, seven constant loads folded into immediate adds and compares, and a
  shorter sign fix-up after the compressor's divide. Before and after rendered
  bit-identically in `dsp_host` on all 41 compared cases (every reference case
  at two MIX settings, moving knobs at splits 0, 1 and 7, a freeze and a dirty
  start).

Going further means lower filter orders, which would change the sound.

**Stress** (`perf.py stress --seconds 32`, 15 minutes, on BBD14). This is E-Verb's load,
driven straight into `dsp_host` on the private image, because
`stress_project.py` needs a template Octatrack project that isn't available
here:

- BBD on FX2 of all eight tracks (four per core), behind CHORUS on every FX1.
  CHORUS has eight named slots; DJ EQ, the dearest stock effect, has five.
- Per block, three LFOs (sine, triangle, square) on TIME, FDBK and MIX.
- Each sixteenth at 120 BPM, a new random value on all 18 named slots: BBD's
  ten and CHORUS's eight. 12 can be locked on a unit; STGS, LIM, PRE, POST,
  TAPS and FBLP are SETUP knobs. This also forces BBD's fade-and-clear again
  and again and switches PRE and POST on and off.
- Trig splits of 1, 3, 7, 8, 12 and 15 frames on six tracks.
- Eight stems: drums, a chord, noise bursts, a sweep, a full-scale square,
  impulses, DC steps and plucks.
- `-guard 0x4000 -guard-shared -dirty 0x5a`.

Result:

- No hang.
- BBD's eight instances: 0 stray writes and 0 clobbers. No instance clobbers
  another.
- The guard also counts 13,318,840 stray-write regions for the stock CHORUS
  instances. I didn't examine them further.
- Per-core peak 2,774.8 and 2,802.4 executed instructions per sample with all
  sixteen effects.

The previous record (2,544.8 and 2,549.2, 13 slots a step) came from an image
older than FRZ, FREQ, LIM, PRE and POST; this run replaces it.

It exercises the DSP side only: the firmware's own LFO and lock delivery, Part
switches and the ColdFire side are not in it.

`npm run perf:audit -- check evidence/performance.json` (Node 24):

```
✓ record          dsp record for bbd@0.1.0-experimental
· cycles          4 x 401 = 1604 of 3120 per sample per core (51%), measured 433.31 per instance; over half a core: say what stays free for stock effects and other modules
· stock benchmark 433.31 vs CHORUS 293.06 (1.48x); the dearest stock effect is 330.94; justified in the record
✓ stress          4 per core, eight tracks, 3 LFOs and 18 locked slots, 32 s, no clobber, no hang
```

What stays free: four BBDs on one core leave 1,516 of the 3,120 usable static
cycles for the four FX1 effects on those tracks and any other module on that
core. The stress run above, with CHORUS on every FX1, peaked at 2,802.4
executed instructions per sample per core. Real-chip headroom is unmeasured.

`evidence/bench.json` and `evidence/stress.json` are the raw records. They
hold image hashes only: no firmware, dump or audio.

## Stock flows

Not run under the ColdFire port. From the build, and on the unit BBD Delay
was chosen from the end of the FX2 chooser:

- The FX2 chooser gains BBD Delay at the end and loses SPRING REV and PLATE REV.
- DARK REV stays and renders bit-identically to the stock OS on both cores,
  fixed and moving (the `stock` gates).
- The FX1 chooser is unchanged.
- No other page, gesture or stock behaviour is touched.

## Hardware

Functional report, not a formal qualification: the author (JamCones) played
images BBD01 to BBD14 on an Octatrack MKII, OS 1.40C, on 11 and 12 October
2026. BBD11 (`ddcb0cde…a9166e`) is the bookkeeping pass described under
Performance, bit-identical to BBD10 (`f611a45a…de61735`) in `dsp_host`. BBD12
(`065fcd56…5590fa`) removes the 8192-stage STGS option and halves the clear
after a STGS change. BBD13 (`ca80bbd3…e0c020`) adds the FRZ fix (the input
fades and the loop sustains below 127, so a crossfade into 127 keeps a ringing
loop's level) and draws PRE and POST as stock DELAY's two-state switches
(display only: the DSP receives the same values). BBD14 (`ffe65ea1…`) is
BBD13's code placed as the site builds it, clear of DARK REV's routine; it is
the release image, the last one played and the one these gates ran on.

- **BBD01–BBD02:** BBD on T5 worked, sounded like a BBD and followed scenes.
  On T1 it crackled, and STGS changes made artefacts and then silenced tracks
  1–4 (core B). BBD on T2, T3 and T4 at once did not crackle. Cause and
  diagnosis are in `REP_DIVIDES.md`.
- **BBD03:** with every `rep` replaced by a `do` loop, no crackle on T1 with or
  without a kick playing, none with PHASER on FX1, STGS changes clean on T1 and
  T5 with tracks 1–4 still playing, and a heavier existing project fine.
- **BBD04–BBD09:** FRZ, FREQ, the ±9 dB TONE, LIM and the MIX curve played as
  intended; LIM's orientation, range and default were set by ear on the unit.
  No problem clipping at MIX 64.
- **BBD10:** PRE and POST sound as intended, with no switch-on thump and no
  crackle.
- **BBD11:** sounds fine; no change heard from BBD10.
- **BBD13** (includes BBD12, not played separately): no issues; FRZ works as
  intended.
- **BBD14**, the release image (`ffe65ea1…`): BBD as on BBD13 with no T1
  crackle; DARK REV works on FX2; the FX2 chooser has no SPRING REV or PLATE
  REV; two BBD instances at once; a Part or project save and reload; eight
  instances (FX2 on all eight tracks, four per core) for 30 minutes with
  feedback and modulation running on every track; a saved project restored
  after a power cycle, settings intact; FREQ swept end to end by an LFO; FRZ
  and every other control played through the crossfader.
- **BBD14E**, BBD with E-Verb as the site builds it (`faf93562…`, equal to its
  native record; all three stock reverbs given up): both effects working. An
  extra combination check.

Not reported: LFO sweeps of controls other than FREQ, MIDI/USB load, Part
switches, hardware canaries, wall-clock timing and an MKI.

## OT UI capture evidence

`scripts/capture-module-ui.py` used the headless `ot_emu` (MKII panel, empty
scratch card, stopped transport) on the private image above. It captured
`ot-setup.png`, `ot-main.png` and `ot-feedback.png`; hashes and the panel plan
are in `media/capture.json`. The STGS knob draws with the OS's select icon;
PRE and POST draw as stock DELAY's two-state switches (its ON/OFF formatter
and switch widget, read from DELAY's descriptor). That is a display change
only: the DSP receives the same 0/1 values as before. UI captures say nothing
about audio or hardware.
