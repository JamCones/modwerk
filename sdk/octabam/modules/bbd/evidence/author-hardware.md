# BBD Delay author hardware report

Tester: JamCones (Cam Jones), the author. Octatrack MKII, OS 1.40C.
Dates: 11 and 12 October 2026. These are the author's reported listening
results; nobody else observed the unit.

Release image: BBD14, MAIN OS SHA-256
`ffe65ea1f10926c0ffde18d9afe38b346826136d31673d5cc2823781ebca90f1`, the site's
own build of BBD beside the stock effects, equal to the native record in
`sdk/native-comparisons/bbd.json`. The released folder rebuilds it byte for
byte. Source: `536fb51b`. BBD14's DSP code is BBD13's; only its placement moved (below).

## What was played

- **BBD01–BBD02:** on T5, BBD worked, sounded like a BBD and followed scenes.
  On T1 it crackled, and STGS changes made artefacts and then silenced tracks
  1–4 (core B). BBD on T2, T3 and T4 at once did not crackle. Diagnosis in
  `REP_DIVIDES.md`: `rep` divides holding interrupts off.
- **BBD03:** every `rep` replaced by a `do` loop. No crackle on T1 with or
  without a kick, none with PHASER or FILTER on FX1, STGS changes clean on T1
  and T5 with tracks 1–4 playing, and a heavier existing project fine.
- **BBD04–BBD09:** FRZ, FREQ, the ±9 dB TONE, LIM and the MIX curve played as
  intended; LIM's orientation, range and default were set by ear. No problem
  clipping at MIX 64.
- **BBD10:** PRE and POST sound as intended, with no switch-on thump and no
  crackle.
- **BBD11:** sounds fine; no change from BBD10 (bit-identical in software).
- **BBD13** (includes BBD12, not played separately): "no issues and freeze
  works great". It removes the 8192-stage STGS option, keeps a ringing loop's
  level through a crossfade into FRZ 127, and draws PRE and POST as stock
  DELAY's switches.
- **BBD14**, the release image: BBD behaves as on BBD13, with no crackle on
  T1; DARK REV works on FX2; the FX2 chooser has no SPRING REV or PLATE REV and
  keeps DELAY, DARK REV and BBD Delay; two BBD instances ran at once; a
  Part or project save and reload worked; eight BBD instances, FX2 on all eight tracks (four per DSP core), ran for 30 minutes with every track's feedback and modulation running, without a problem; a saved
  project came back after a power cycle with BBD on its tracks and its
  settings intact; an LFO swept FREQ across its whole range; and FRZ and every
  other control were played through the crossfader (scenes).
- **BBD14E**, the site's build of BBD with E-Verb beside the stock effects
  (MAIN OS `faf935628af849eca30cb3e3f73c061c1d28d557213eee67bfea36518ec898a2`,
  equal to its native record): "all good", both effects working. An extra
  combination check, not part of BBD's own qualification.

Images BBD01–BBD13 placed BBD in SPRING REV's space alone and kept DARK REV,
which calls a routine inside that space, so DARK REV was broken in them (not
noticed on the unit; found by the site's builder and confirmed by a render
gate). None of them was published. BBD14 gives up PLATE REV too and places
BBD clear of the routine.

## Limitations

- Attributed author-reported functional results, not a detailed stress
  qualification. The 30-minute eight-instance run is a listening result; no
  hardware canaries or wall-clock measurements were taken.
- Only FREQ was swept end to end by an LFO; the other controls were moved by
  hand, locks and scenes, not by LFO across their full range. MIDI/USB load
  and recovery are not reported. MKI is untested.
- Cycle values are software bounds and emulator measurements; chip contention
  and wall-clock timing are not measured.
