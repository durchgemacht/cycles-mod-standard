# Cycles Mod Standard — draft 0.5

**Status:** draft 0.5, 2026-10-26, public. Draft 0.1 (2026-10-21) described a target only. Draft 0.2 described the
`machine` point of Level B and the manifest of Level C as [model27](https://github.com/durchgemacht/model27) built
them: a composer that places committed objects and registers them through a runtime, with no toolchain at build
time, tested on a Model:Cycles on 2026-10-22 (§8). Draft 0.3 added what model27 built next:
- the machine shortcuts, a fixed list of key + knob combinations a machine may use on its own tracks (§3.1);
- dispatcher tables for every point SAMPLE uses: `sound.record`, `view.main`, `view.knob`, `screen.tick`,
  `browser.open` (§3.3);
- the same composition in the web flasher, in JavaScript, byte for byte (§8).

Since then model27 has built the claims (2026-10-23):
- what each mod takes beyond its writes, as declared data in one shape: saved slots, key combinations, descriptors,
  formatters, MIDI numbers, RAM, views (§4.5);
- the conflicts of rules 2 and 3 computed from them, by both composers and the overlap check (§4.4);
- the registry's tables generated from them (§5).

And then two more machine shortcuts (2026-10-26):
- RETRIG + PUNCH and RETRIG + GATE, each a switch, on or off, saved with the sound (§3.1);
- on the machine's tracks the PUNCH and GATE LEDs show the switches while RETRIG is held;
- a machine reads them in the sound its note plays, because the parameters it gets are smoothed (§2.3).

The machine parts are proven in emulation with a second, internal reference machine, REF2, whose manifest is the
template for porting a machine (§7.4); REF2 was played on a Model:Cycles next to SAMPLE on 2026-10-26, switches
included. The claims are proven by the tools' own checks. The shared header is still a proposal.
**Authors:** durchgemacht and Claude.
**License:** MIT ([LICENSE](LICENSE)).
**Scope:** mods for the Elektron Model:Cycles OS 1.13 that are built from the user's own official file with
Modded-Cycles' tools. Everything here is read from the code of Modded-Cycles (release 1.56, fork commit `95b63b7`),
Cycles One-Shot 0.6 to 0.8, model27 and the technical notes the projects keep about the OS. Where a fact was observed
but not followed through in the OS, it says so. Addresses are virtual addresses of the unpacked MAIN OS (base
`0x40000400`).

**Applying it to an existing mod:** [APPLYING.md](APPLYING.md) is the checklist for authors and their AI
assistants. It produces an inventory, a manifest, the collisions and the deviations of a mod, not a rewrite.

**Changes from 0.4:**
- §2.3: the parameters a machine gets are smoothed; a value it must read exactly is read in the note's sound;
- §3.1: the `shortcut` point gains two switches, RETRIG + PUNCH and RETRIG + GATE; the `gesture` row;
- §3.3: the shortcut layer's two more sites and its byte of presses per machine;
- §4.2: the `shortcut` entry for a switch;
- §4.5: GATE is key 7; the stock list gains a press on PUNCH or GATE, and FUNC + PUNCH/GATE;
- §5.2 to §5.4: k 32, record slot 18, the switches' row;
- §7.4: REF2's manifest with Square and Tone open;
- §8, §10 and §11: brought up to date.

**Changes from 0.3:**
- §3.1: the `gesture` and `param.sound` points as model27 computes them, and the gestures each mod takes;
- §4.2: the shapes of `claims`; new §4.5, the claims as model27 reads them (scopes, situations, the words of a
  combination, the stock combinations);
- §4.3 and §4.4: rules 2 and 3 computed, and what building them settled;
- §5: model27 keeps no number file, its registry's tables come from the claims; §5.2 to §5.5 brought up to date;
- §7.1, §7.3 and §7.4: the claims and the counts as computed now;
- §8 to §11: brought up to date.

**Changes from 0.2:**
- §3.1: `sound.record`, `view.main`, `view.knob`, `screen.tick` and `browser.open` as model27 dispatches them, and the
  new `shortcut` point;
- §3.2: the "by machine" rule;
- §3.3: the dispatcher tables as built;
- §4.2: the `shortcut` entry of `uses`;
- §5.1: option C taken;
- §5.2 to §5.4: the shortcuts' slots and keys;
- §7.4: REF2's manifest;
- §8 to §11: brought up to date.

**Changes from 0.1:** the status above; §3.3 the machine table as model27 runs it; §4.2 to §4.4 the manifest as
model27's composer reads it (the object's recipe, patch points as symbols, hooks inside `uses`) and what it computes;
§5.1 and §5.6 model27's numbering and order; §7.1 SAMPLE's manifest as built (One-Shot 0.8); §8 the builder as
built and the web flasher's choice; §9 what is done; §10 that choice taken; One-Shot's sample pool above the payload
since 0.7 (§2.7, §3.1). The rest is as in 0.1.

- [0. Why, and what this is not](#0-why-and-what-this-is-not)
- [1. Terms](#1-terms)
- [2. Level A: the machine contract](#2-level-a-the-machine-contract)
- [3. Level B: extension points](#3-level-b-extension-points)
- [4. Level C: the manifest](#4-level-c-the-manifest)
- [5. The number registry](#5-the-number-registry)
- [6. The cost model](#6-the-cost-model)
- [7. Examples: SAMPLE, MACRO, track-filter, REF2](#7-examples-sample-macro-track-filter-ref2)
- [8. Builder and web flasher](#8-builder-and-web-flasher)
- [9. Migration path](#9-migration-path)
- [10. Open decisions](#10-open-decisions)
- [11. Sources](#11-sources)

## 0. Why, and what this is not

Three things are re-invented by every mod today, and each one is visible in the repositories:

- **The OS is re-derived.** The addresses of the decimator, the amp envelope, the VCA, PUNCH, the decay table and
  the voice base are defined again in `acid.c`, `macro.cc`, `sdvintage.c` and One-Shot's `os.h`. The comment that
  describes the voice-loop contract is the same text in `acid.c` and in One-Shot's `sampler.c`.
- **Slot, descriptors and payload address are fixed when a tweak is generated.** That is why
  `tweaks/model-cycles_OS1.13/` holds 283 files, 252 of them combinations of Syntakt engines, and why SAMPLE is
  machine 6 and nothing else.
- **Compatibility is a hand-written list of ids.** One-Shot's list does not know ACID, which arrived after the
  commit it was built against. The reasons for a conflict (same hook, same sound slot, same gesture, same memory)
  are in the bytes and in the notes, not in a declaration a tool can read.

This document is **not** a new loader and not a new build pipeline. A mod that follows it is still a tweak JSON that
`tools/build.py` applies and the web flasher embeds. It builds on what exists: `check_overlaps.py` (notes/49),
`registry.py` and `REGISTRY.md` (notes/52), the flasher cards (`gen_flasher_tweaks.py`) and the added-machine
mechanics (`gen_macro.added_machine`). It adds three things:

- **Level A, in the firmware:** one written contract for machines and one shared header, instead of copies.
- **Level B, at build time:** named *extension points* with a composition rule each, owned by the standard, so that
  the builder, not the author, assigns slots, descriptors, payload addresses and dispatch order.
- **Level C, for distribution:** a manifest per mod from which the tweak JSON, the flasher card, the registry and
  the conflicts are *derived*, and a registry of numbers that end up in users' data.

## 1. Terms

| Term | Meaning |
|---|---|
| **machine** | a sound engine the OS selects on the MACHINES page: two functions in the voice loop plus five parameter descriptors |
| **package** | a mod as shipped: one or more relocatable objects (m68k ELF `.o`) and a manifest |
| **extension point** | a named place in the OS a package may use: the machine table, a stage of the track chain, a menu, a gesture, a vtable entry, a range of sound slots |
| **composition rule** | how several packages share one extension point: table, chain, append, exclusive, allocation, registry |
| **claim** | a resource a package needs: an extension point, a hook address, a sound slot, a gesture, a memory range |
| **number registry** | the permanent assignment of numbers that are stored in users' projects, kits, sounds or settings |
| **dispatcher table** | a fixed-size table of callbacks in the standard's own payload, filled by the builder, walked by code written once |
| **build** | one set of packages chosen by the user, turned into one image |

## 2. Level A: the machine contract

This level describes what the OS already imposes. Nothing here is new; it is written down once so that it stops
being copied. The facts were read in the OS by Modded-Cycles (notes/14, 18, 43, 54) and by One-Shot
(TECHNICAL.md §5, §6) and agree with each other.

### 2.1 The voice loop

The voice loop (`0x400a7d4a`) calls, for each of the six tracks and once per block of 32 frames at 48 kHz, the two
functions of the track's machine, taken from the `update` and `render` tables:

```c
void update(int32_t pmod, char *voice, const char *params);   /* then */
void render(int32_t *out, char *voice);                        /* 32 frames, Q31 */
```

- `pmod` is the trig's note in semitones `<< 16`.
- `params` are the note's parameters as 8.8 words at fixed offsets: `+0x12` MACHINE, `+0x14` PITCH, `+0x16` COLOR,
  `+0x18` SHAPE, `+0x1a` SWEEP, `+0x1c` CONTOUR, `+0x1e` PUNCH, `+0x20` GATE, `+0x22` FINE, `+0x24` DECAY.
- The note a machine plays is PITCH − 64 + the trig's note + FINE (±2 semitones), clamped to 0–127. Every stock
  machine computes it that way (notes/43 §2, One-Shot TECHNICAL §5).
- `out` takes 32 `int32_t`; the sound of a trig starts in the block *after* the trig: the VCA keeps the trig's own
  block silent (One-Shot TECHNICAL §5).

### 2.2 The voice

A voice is `0x31c` bytes at `0x42308828 + t · 0x31c`:

| Offset | Meaning |
|---|---|
| `+0x00`–`+0x3c` | cleared by the voice reset `0x400a7ab8` when a track changes machine; the reset also sets the amp envelope's defaults |
| `+0x2c` | free for a marker the machine uses to notice that reset (ACID writes `'ACD1'`, One-Shot `'SMP1'`) |
| `+0x34` | set in the block of a trig |
| `+0x38` | set in the block after it (the OS copies `+0x34` there at the end of a block); machines set their envelopes up when it is set |
| `+0x3c` | set when the note ends |
| `+0x230` | amp envelope level, Q31 (`0x80000000` full, 0 silence) |
| `+0x234` | the level of the last block (the VCA ramps from it) |
| `+0x23c` | envelope stage |
| `+0x248` | attack: `0x80000000` means full at once; anything else is added each block |
| `+0x24c` | gate flag, set by the voice loop for every machine |
| `+0x250` | decay factor per block, from the table at `0x4011d84c` indexed by DECAY |
| `+0x318` | pointer to the decimator's state frames in SRAM |

### 2.3 The parameters and the sound

- When a note's sound differs from the track's last one, the audio side copies the sound's 66 bytes of parameters
  (k 0–32) to `0x800015ae + 66 · t` (`0x40058a0a`), before any smoothing. The sound a track played last is at
  `0x800017ec[t]`; 0 there makes the next note copy again (the OS does that after a preview note, `0x4005910e`).
- The `params` a machine gets are not that copy. The audio block function calls the voice loop (`0x4005981e`) with
  the buffer `0x40058474` returns, a one-pole smoother over the parameter words (Q15 coefficients `0x7c29` and
  `0x03d7`), after `0x40056e24` and the LFO (`0x40091ab2`) have changed it; track t's `params` is that buffer plus
  14 + 66 · t
  ([model27 notes/63](https://github.com/durchgemacht/model27/blob/main/notes/63-level-b4-shortcuts-dispatchers.md)
  §17.2). A knob's value glides, which suits a knob. A value a machine must read exactly and at once, a switch or a
  table number, is read in the sound the note plays instead: `0x800017ec[t]` + 20 + 2k, or the copy
  `0x800015ae + 66 · t + 2k` when `0x800017ec[t]` is 0 (a preview note). CHORD Tables and REF2 do (model27 notes/66
  §2.1, notes/63 §17.2). Which words the smoother covers was not measured word by word.
- A sound is 100 bytes: `+4` name (16 bytes), `+20 + 2k` parameter k as 8.8, `+38` the machine (high byte of k 9).
  k 6 is SWEEP, 7 CONTOUR, 11 COLOR, 12 SHAPE. The slots k 23–32 exist in the sound but the stock OS does not use
  them; section 5.2 says who does.
- Kits sit at `0x409c9c80 + 7094 · n` for the 96 patterns: six track sounds from `+28`, the pool of 64 sounds from
  `+694`. The audio side plays from a working copy of the current kit (`*0x800017e4`).

### 2.4 The stock chain

TONE, MACRO, ACID and SAMPLE end `render` with the same four functions, in this order:

| Address | Function | Contract |
|---|---|---|
| `0x400a9f58` | 2:1 decimator | 64 frames at 96 kHz in, 32 at 48 kHz out; IIR half-band; the Model:Samples uses the same code |
| `0x400a9252` | amp envelope | once per block; stages attack, decay, gate (section 2.2) |
| `0x400a9430` | VCA | applies the envelope to the 32 frames, ramping within the block from `+0x234` to `+0x230` |
| `0x400a967a` | PUNCH | gain with saturation and a compressor-like envelope |

Machines that run at 48 kHz skip the decimator. TONE's setup of the chain is at `0x400aa7b8`; MACRO and SAMPLE copy
it. The decay table `0x4011d84c` maps DECAY to the per-block factor; One-Shot's TECHNICAL §5 lists the times.

Other facts of the audio side that machines use today:

- velocity gain of this block `0x40fe4c90`, of the next blocks `0x40fe4cd0` (ACID's slide);
- the rest of a trig's length `0x800017b4`, and what the countdown loses per block `0x40149310` (ACID);
- with the Syntakt engines' load governor, `update` and `render` of a quiet voice are not called until its next
  trig (`GOV_SKIP`); a machine with state must treat a gap of more than three blocks as a muted voice. ACID reads
  DMA timer 0 at `0xfc07000c` (135.168 MHz, 90 112 ticks per block) for that.

### 2.5 Descriptors and knobs

- A parameter descriptor is `0x38` bytes: `+0x00` the machine it belongs to, `+0x08` minimum, `+0x0c` maximum,
  `+0x10` default (8.8), `+0x2c` long name, `+0x34` short name. The stock table has 76 entries at `0x4010dce0`.
- **An added machine gets exactly five descriptors:** COLOR, SHAPE, SWEEP, CONTOUR and Amp Decay, modelled on
  SNARE's. That is the OS's layout, not a design choice. "Four knobs" is therefore the baseline of every machine;
  the standard only says what may be added on top (section 2.7).
- A machine borrows one of the six stock pictures for the MACHINES page (MACRO, ACID and SAMPLE take TONE's, 4).
- The knob's text comes from the OS's parameter objects (`0x40a71754 + 100 · i`); SAMPLE's descriptors share
  SNARE's objects through a detour of the accessor `0x4004df40` (notes/18). An object's `+0x20` is the formatter:
  `0x400456c8` prints 0–127, `0x400456ea` −64…+63, `0x40045bd6` OFF. A machine may replace the formatter of its own
  knobs (One-Shot does for Cutoff); it is a claim (section 3).

### 2.6 Rules for a core machine

A **core machine** uses the machine extension point and nothing else. It:

- MUST implement `update` and `render` as in 2.1 and keep the pitch law of 2.1 (PITCH, FINE and the trig's note);
- MUST end `render` with the stock chain of 2.4 in TONE's order, or document in `stock_behaviour` where it differs
  (One-Shot's filter runs after the chain, before Volume + Dist: documented);
- MUST do nothing while the voice is idle (no trig pending, envelope below −102 dB, own sound ended), so that six
  idle tracks cost nothing (notes/23 §7: −77 % instructions when the six voices are quiet);
- MUST survive the voice reset of 2.2 and the governor's gap of 2.4 without a click or a stuck note;
- MUST keep its state in its own BSS and its code and tables in its own payload: no RAM outside what the manifest
  claims, no OS function detoured;
- MUST leave every other track bit for bit as the stock OS plays it (the track-filter proves that at OFF against
  the real code, One-Shot's filter at 0; the same proof is expected of a machine's "off" positions);
- SHOULD give every knob a meaningful range end to end and a sensible default, and SHOULD not click under a square
  LFO on any knob (One-Shot 0.6 fades the filter for that reason);
- MUST declare its cost by the method of section 6.

### 2.7 Extensions a machine may use

Anything beyond 2.6 is an extension and goes through an extension point of section 3: a gesture (MACRO's
FUNC + PITCH lo-fi, ACID's PUNCH as waveform switch), a view (One-Shot's waveform screen), the browser (One-Shot
opens sample files), sound records (One-Shot's sample key), the file system, a memory range (One-Shot's sample
pool, 47 MB since 0.7). A machine that re-reads a stock parameter (ACID: DECAY also drives the filter envelope,
PUNCH is a switch) declares that in `stock_behaviour.changes`, because the stock meaning is lost on its tracks.

### 2.8 The shared header

`cycles_os.h`, one file, replaces the four copies. It holds, with the comment on where each was read:

- the voice loop contract (2.1) and the voice layout (2.2) as macros and offsets;
- the stock chain (2.4) as inline wrappers, and a `stock_chain_tone()` helper that sets the amp chain up TONE's way;
- the sound, kit and parameter-copy addresses (2.3);
- the descriptor layout and the formatters (2.5);
- the idle test and the governor gap test (2.6);
- the file system (lock, read, index), the screen (redraw, views, text, fonts) and the browser, for extensions,
  as One-Shot's `os.h` has them today.

The header carries the OS version in its name or a macro; a package built against it records that version in its
manifest (`compat.os`).

## 3. Level B: extension points

An extension point is a place in the OS that the standard *owns*: it has one hook there, one dispatcher table, and a
rule for several packages at once. A package never detours such a place itself; it registers a callback in its
manifest and the builder fills the table. Places the standard does not own are plain hooks, declared as claims
and checked byte for byte by `check_overlaps.py`, as today.

### 3.1 The points

| Id | Where in the OS | Composition rule | Limit | Used today by |
|---|---|---|---|---|
| `machine` | the `update`/`render` tables, the names, the record map, the descriptors and knob rows, moved into the payload (notes/18) | **table**: the builder assigns the index and the descriptors | 6 added machines in the current layout (109 descriptors fit between `DESCN` and `ROWSN`); 14 marks on the MACHINES page (2 rows of 7); 8 added records | MACRO, ACID, Syntakt engines, Model-TG's Sampler, SAMPLE, sdvintage-7th |
| `track.post_voldist` | the call of Volume + Dist in the audio block function: `0x40059852` → `0x40056f58(0x80001858, a2)` ("H3", notes/47 §2) | **chain**, ordered by the packages' declared `order`; each stage sees the six track buffers `0x80001858 + 128 · t` after Volume + Dist and before level, pan and sends | register contract: keep d2–d7/a2–a6, restore MACSR (`0x20` there), leave acc0–acc3 at zero | track-filter |
| `track.post_voice` | after the voice loop (`0x4005981e`), before the velocity gain (`0x4005982a`) | **chain** | candidate: no mod uses it; contract to be read | — |
| `fx.bus` | the delay and reverb | **chain** | **not mapped**: the mixer `0x400567ba` does level, pan and sends; where the effects run has not been read | — |
| `sound.record` | the three sound record functions: save v2 `0x4005afa0`, save v1 `0x4005b054`, load `0x4005aecc` | **chain**, as built in model27: the builder's shortcut layer takes the functions' first 8 bytes (`lea -12(sp),sp ; movem.l a2-a4,(sp)`) and runs one package's wrapper as the next of its chain (One-Shot's "around" form: a copy of the arguments, the 8 bytes replayed, the stock body, then the package's `after` function, `d0` kept), then copies k 28–31 into record slots 28–31 or back; without a shortcut in the build, the package writes its own wrapper | one wrapper per function behind the layer; the functions return 0 when they did nothing (a null pointer; for the load also a record of another version than 2, left untouched), and a wrapper copies its slots only when the function returned 1 | Model-TG, track-filter, One-Shot, model27's shortcut layer |
| `param.tables` | at boot the OS builds its parameter tables (k → descriptor per machine, CC, LFO) from the descriptors: `0x4005a274`, called once from `0x4004486c`; the track-filter's `tf_boot` hooks that call and copies its four descriptors into the active table (notes/47 §4, `tf_ui.S`) | **append** at boot | the active table is the one the build moved (the Syntakt engines move it into their payload); a package finds it where the build says | track-filter |
| `sound.copy` | the sound's copy of its parameters by record slot (`0x4001461c`, slot 17 of `Sound`'s vtable), whose bound k ≤ 22 at `0x4005aa34`/`0x4005aa54` drops everything above (notes/47 §4.3) | **chain** | contract to be read from notes/47 §4 | track-filter |
| `browser.open` | the PRESET browser reads the opened file: `0x400a6602` (`jsr 0x400a3052`) | **by machine**, as built in model27: the hook of the machine of the track the browser loads into (the browser object's `+664`), otherwise the stock read; a machine's hook takes a0 = &result, `4(sp)` = &handle, a2 = the browser's pointer and the caller's d2–d7 (d4: the entry's index), and ends in the stock read unless it takes the file | one hook per machine | One-Shot, REF2 (sample-preview, with Model-TG, plays files from the same browser) |
| `screen.tick` | the screen's redraw asks whether to draw: `0x400075f0` (`jsr 0x40076be4`), about 30 Hz, UI task | **append**, as built in model27: every machine's hook in machine order, then the stock `0x40076be4(view)` once; each hook also ends with that getter, which only reads the view's `+32`, and returns its answer | the redraw also runs from another task (screen copy); a hook takes its own lock if it needs one (One-Shot's tick does) | One-Shot, REF2 |
| `view.main`, `view.knob` | entry `+16` of MainScreenView's vtable (`0x400ffe30`, stock `0x4001b22a`) and of the knob view's (`0x4010053c`, stock `0x4001e40a`): the draw method | **by machine**, as built in model27: the draw of the selected track's machine, otherwise the stock draw; each draw still checks the track itself and calls the stock draw in every case it does not take, so that it works as the point's only taker too | one draw per machine and view; level-pan-values draws on the main screen through other hooks (`0x400081f2`, `0x4001aa4e`) and is not a byte conflict, but a functional one this point would make visible | One-Shot, REF2 |
| `menu.track_setup` | FUNC + TRACK; TrackMenuView is rebuilt at every opening by its one caller `0x4001c510` (`jsr 0x4003e020`); entries are 84-byte MenuItems (`0x400734b0`, vtable `0x40103470`) with four `std::function` callbacks, appended with `0x400732d6(menu, item, 12)`; strings via `0x400f980c` | **append**, in `order`; a package gives label, value text, change callback and a scope (all tracks, or tracks of its machine) | the MenuItem ABI is the risk; the shared header wraps it once | — (read for One-Shot's loop/reverse, not built) |
| `shortcut` | on a track of the machine that declares it, RETRIG held + COLOR, SHAPE, SWEEP or CONTOUR (a value): the knob lookup's vtable entry `0x401005a8` (stock `0x4001e814`); RETRIG held + a press on PUNCH or GATE (a switch): the function both presses call, `0x4001ac4a` (its first 6 bytes; its only callers are PUNCH's and GATE's cases of MainScreenView's key handler, `0x4001c2d2` and `0x4001caa2`), and MainScreenView's LED method at `0x4001cc86` (6 bytes, LED 19 PUNCH, LED 20 GATE) | **by machine**: one shortcut per knob or key and machine, from the builder's fixed list (model27: the four knobs, and PUNCH and GATE since 0.5). The builder's shortcut layer shows a value through the OS's unused descriptors 1–4 and saves it in k 28–31 and record slots 28–31; it flips a switch in k 32 (bit 8 PUNCH, bit 9 GATE) instead of PUNCH or GATE, saves it in record slot 18 (§5.2, §5.3), and lets PUNCH's and GATE's LEDs show the switches while RETRIG is held. Everything else about a switch, whether and where the screen shows it, is the machine's | four values and two switches per machine. A value never with FUNC (fine adjust), TRACK (Control All), LFO (its destination) or a held trig (p-locks); a switch never with TRACK (Control All stays), and a held trig changes nothing. No p-lock and no LFO destination; 0 is every value's and switch's default (a new sound and an old one read 0) | REF2 (Octave; Square, Tone open) |
| `gesture` | key combinations and their meaning | **exclusive** per combination, situation and scope; no dispatcher. As model27 computes it: a combination's atoms, one set of held keys and one action each, clash in situations and scopes that meet, with another meaning (§4.5) | — | MACHINES held + COLOR/SHAPE/SWEEP/CONTOUR (track-filter), MACHINES held + DECAY/SWEEP/CONTOUR (Model-TG's Attack, Filter, Resonance), FUNC + PITCH on MACRO tracks (lo-fi), TRACK + TRIG 1–6 (control-all-config), TRACK held + FUNC (latching-mute), a TRIG held + PAGE (trig-preview), RETRIG held + notes (arp), TRIG 1–16 in Keys mode (chord-keys), LFO on the LFO page (lfo-plus), RETRIG + knob and RETRIG + PUNCH/GATE (the `shortcut` point); the stock combinations of §4.5 |
| `param.sound` | the sound slots k 23–32 and the record slots | **registry** (section 5); computed from the claims in model27 (§4.4) | no k is free any more (notes/56 §3.6) | Model-TG, track-filter, lfo-plus, One-Shot, model27's shortcut layer |
| `param.descriptor` | the OS's unused descriptors 1–4 ("Error"), which the track-filter turns into real parameters of every machine (p-locks, LFO, CC, NRPN) | **exclusive** | four | track-filter; model27's shortcut layer, machine-scoped (their machine stays −1, so no row, CC, NRPN or LFO list has them) |
| `midi` | CC and NRPN numbers a package listens to | **registry** | — | track-filter (CC 74, 71; NRPN 1:20, 1:21) |
| `memory.mask` | freed sprite masks (notes/52): 164 masks, 35 960 bytes; 113 free (18 152 bytes) at release 1.56 | **allocation** by `registry.py` and `sprites.redirect_write` | a mask is redirected in the same tweak that writes it | arp, chord-keys, lfo-plus, track-filter, trig-hold, … |
| `memory.payload` | code copied at boot from the end of the image to RAM: `0x43000000` (alone) or `0x46700000` (with Model-TG), boot hook at `0x400004b2` (the BSS clear) through a stub in the cave `0x4016cae8` | **allocation**: the builder lays packages out one after the other and fills the copy list | the image must end below `0x40200000`; the stock image ends at `0x401aa140` | every added machine, One-Shot |
| `memory.ram` | RAM the stock OS leaves unused: `0x42800000`–`0x46700000` (the Model:Samples keeps its sample pool there) | **allocation** | the alone payload window `0x43000000` lies inside it: a claim must stay clear of the build's payload (model27's composer refuses a layout that runs into one) | One-Shot's sample pool, `0x43800000`–`0x46700000` since 0.7, above the alone payload |

### 3.2 Composition rules

| Rule | Meaning | Builder's job |
|---|---|---|
| table | N numbered entries | assign numbers (or take them from the registry, section 5.1), refuse above the limit |
| chain | ordered list of callbacks at one place | sort by `order`, refuse two packages with the same `order` at the same point, fill the table |
| append | unordered list, every callback runs | fill the table |
| by machine | one callback per machine; the one of the track's machine runs, otherwise the stock code | fill the table by machine number; each callback still checks its track |
| exclusive | one owner | refuse a second claimant with a different meaning; allow two identical claims |
| allocation | space handed out at build time | lay out, record in `REGISTRY.md` |
| registry | numbers fixed for ever | refuse two packages with the same number; refuse a number not in the registry |

### 3.3 Dispatcher tables

For every chain or append point the standard's own payload part holds a table of, say, eight 32-bit entries ending
with zero, and one dispatcher written once in assembly that saves what the point's contract requires, walks the
table and restores. The builder only writes addresses into the table. Consequences:

- no code is generated per combination, so the web flasher can fill the tables in JavaScript;
- the register contract of a point (the H3 contract above) is kept in one place, not in every mod;
- a package that is not installed is simply not in the table.

The machine table already works like this, and model27's registrar runtime does it literally for the `machine`
point: code assembled once, a parameter block the builder fills per build (§8).

**As built in model27** ([notes/63](https://github.com/durchgemacht/model27/blob/main/notes/63-level-b4-shortcuts-dispatchers.md)
§11 to §13). Two *parts* of the builder are compiled once and committed: objects with roots and patch points like a
package, no machine of their own. The composer adds a part to a build only when the build needs it:
- **the shortcut layer** (`tools/machines/shortcuts/`, 0.2 since the switches), when a machine declares a
  `shortcut`:
  - it takes the knob lookup's vtable entry and the three sound record functions; when a machine declares a switch,
    also the start of `0x4001ac4a` and the 6 bytes at `0x4001cc86` in MainScreenView's LED method;
  - its data block after its code holds the next of each record function's chain, the descriptor table the OS uses,
    per machine a byte of the presses it declared (bit 0 PUNCH, bit 1 GATE; eight machines at most), and per machine
    and knob the shortcut's names, range, default and formatter;
- **the dispatchers** (`tools/machines/dispatch/`), when two machines take the same point:
  - `view.main` and `view.knob` go by the selected track's machine;
  - `screen.tick` goes to every machine;
  - `browser.open` goes by the machine of the track the browser loads into;
  - a 100-byte data block holds the first machine, their number, and a table of six methods per point.

A point one machine takes keeps that machine's own hook or entry. So SAMPLE alone, and SAMPLE with a machine that
takes none of its points, are byte for byte what they were. The web flasher fills the same tables in JavaScript
(§8).

### 3.4 Limits a builder enforces

- at most 6 added machines (descriptors), 14 machines on the MACHINES page, 8 added records, in the current layout
  of the moved tables (`gen_syntakt_engines.check_room`); a larger layout is possible because the tables are ours,
  and model27 will raise the number of added machines soon;
- the image ends below `0x40200000`;
- the payload window of a build: `0x43000000` upward alone, `0x46700000`–`0x46800000` with Model-TG; a package's
  `memory.ram` claim must not overlap the window;
- the sum of the cost of six voices of the heaviest chosen machine plus every chain stage (section 6).

## 4. Level C: the manifest

### 4.1 Principles

- **Kind-neutral core.** A package does not say what it *is*. It lists the extension points it *uses*. One-Shot
  uses `machine`, `browser.open`, `sound.record`, `screen.tick`, `view.main`, `view.knob` and `memory.ram`; the
  track-filter uses `track.post_voldist`, `sound.record`, `param.tables`, `sound.copy`, `param.descriptor`,
  `gesture`, `midi`, `memory.mask`. A format with one kind per mod fails on both.
- **Derived versus declared.** Hooks, writes, payload, sizes, masks and the computed conflicts are *derived* by tools
  from the objects and the registry, as `REGISTRY.md` is derived today, and are never edited by hand. Gestures,
  scopes, stock-behaviour changes, cost measurements and tests are *declared* by the author.
- **Additive schema.** `schema` names the version. Unknown fields are ignored. An unknown *claim type* is not
  ignored: the builder marks the package "unchecked against X" and the flasher shows it.
- **JSON**, like the tweaks and the flasher, so that one toolchain reads all of it.

### 4.2 Fields

| Field | Declared / derived | Content |
|---|---|---|
| `schema` | declared | `"cycles-mod/0.1"`: the name model27's composer reads; draft 0.2 describes that schema as built, and a change to it gets a new name |
| `id`, `version`, `name`, `description`, `source` | declared | as on the flasher card; `id` is the package's folder name, `source` the URL of the sources at the pinned commit |
| `license`, `credit` | declared | as on the flasher card (`credit: {kind, who, repo}`), the license file next to the sources |
| `device`, `os` | declared | `"Model:Cycles"`, `"1.13"` |
| `order` | declared | the package's place among chains (today's tweak `order`); section 5.6 |
| `build` | declared | the object's recipe (`sources`, `include`, `cflags`, `defines`); `objects`: one relocatable ELF file, the `ld -r` of the sources, committed; `symbols`: `update` and `render`; `keep`: the other symbols the writes need (hooks, view methods); `patch_points`: `{symbol, kind}`, an undefined symbol whose address the builder sets (kinds `machine_index`, `descriptor_base`) |
| `uses` | declared | one object per extension point, with that point's own fields; a hook as `hook: {at, stock, op, symbol}` (`op` is `jsr` or `jmp`), a vtable entry as `vtable: {at, stock_ptr, symbol}`, each with `why`. The builder checks the stock bytes against the user's file and writes the hook, or the point's dispatcher when another machine of the build takes it too (§3.3). A shortcut is `{"point": "shortcut", "key": "RETRIG", "knob": "COLOR", "param": {"long", "short", "min", "max", "default": 0, "format"}}`: names in printable ASCII, the short one of 4 characters at most; integers with min ≤ 0 ≤ max within −128…127; `format`, optional, a kept symbol of the package written like the OS's formatters (`(object + 0x14, value, buffer)`), the integer one otherwise; the machine reads the value in k 28 to 31 (`params + 0x38 + 2 · i`, i the knob's place in the list). A switch is `{"point": "shortcut", "key": "RETRIG", "press": "PUNCH"}` (or `"GATE"`): no knob, no `param`, off is 0; the machine reads it as bit 8 (PUNCH) or bit 9 (GATE) of k 32 in the sound its note plays (§2.3) and shows it, if it wants, where and when it likes |
| `claims` | declared | `sound_slots`, `record_slots`, `saved`, `descriptors`, `formatters`, `plock_rows`, `midi`, `gestures`, `ram`, `views`, in the shapes of §4.5: a number or a string takes the holder's own scope (a package's machine); an object names another scope, a range, a situation, a meaning, or what a stock combination it takes did (`replaces`). Hooks are in `uses`; masks, payload and sizes are derived into the build's tweak and `REGISTRY.md`, not written in the manifest |
| `cost` | declared, measured | section 6 |
| `stock_behaviour` | declared | `changes`: what a stock function or gesture no longer does on this package's tracks; `fixes`: stock bugs it corrects; `unchanged`: what was proven bit for bit |
| `compat` | declared | `built_against` (Modded-Cycles commits the build was verified with), `requires` (packages it needs), `conflicts_declared` (reasons a tool cannot compute, with text) |
| `tests` | declared | `emulation` (tool or script, date, which build), `hardware` (date, by whom, which build), `status` (`tested` or `experimental`, with the flasher's rule: tested only after the maintainer's own device) |

**The object and the patch points, as built.** A package ships one relocatable m68k ELF object, the `ld -r` of its
sources compiled with `-fno-pic -fno-common -ffunction-sections -fdata-sections`. Such an object carries only
absolute 32-bit relocations (`R_68K_32`), so a builder places it without a linker: model27's `tools/relocate.py`
keeps what `update`, `render` and `keep` reach, lays the sections out in the linker script's order and writes each
relocation, byte for byte what `ld` gives ([model27 notes/60](https://github.com/durchgemacht/model27/blob/main/notes/60-level-b2-relocator-runtime.md)
§3, §9). A value that depends on the build is an undefined symbol whose *address* is the value, resolved like
`ld --defsym`, so that one committed object serves every build:

```c
extern char cm_machine_index[];                     /* the composer gives it the machine's index */
#define SAMPLE_MACHINE ((int)(long)cm_machine_index)
```

The package's generator rebuilds the object with the pinned compiler and compares it with the committed one
(`--check`).

### 4.3 What the builder derives from a manifest

- the tweak JSON (`writes`, `append`, `symbols`, computed `conflicts`), as `machine/build.py` writes it today;
- the flasher card (`label`, `desc`, `status`, `credit`, `license`, `excludes` from the computed conflicts);
- the registry entries (`REGISTRY.md` §1–§10; in model27, §6 and §8 to §10 come from the claims) and the checks
  of the claims;
- the build report: index assigned to each machine, descriptor numbers, payload addresses, cost sum.

In model27 today the composer derives the tweak JSON and the build report; the flasher card and the registry entries
still come from that tweak JSON through the existing tools (`gen_flasher_tweaks.py`, `registry.py`). The registry's
saved data, key combinations, MIDI numbers and views come from the claims (since 0.4).

### 4.4 Computed conflicts

Two packages conflict when the builder finds any of:

1. the same hook address with different bytes (as `check_overlaps.py` today);
2. the same exclusive point with a different meaning (gesture, descriptor, formatter of the same knob);
3. the same number in the registry (machine id, sound slot, record slot, CC/NRPN);
4. overlapping memory (masks, payload, `memory.ram`);
5. more machines than the layout holds, or a cost sum above the threshold (a warning, not a refusal);
6. the same `order` at the same chain point;
7. a declared conflict with its text.

A package may still list `conflicts_declared` for reasons tools cannot see ("both re-number p-lock rows for
k ≥ 23", notes/56 §3.6): the text is shown on the flasher as the reason, as `clashWhy` does today.

**As built in model27** ([notes/65](https://github.com/durchgemacht/model27/blob/main/notes/65-level-c-claims-check.md),
steps C1 to C4, 2026-10-23). The composer, in Python and in the web flasher alike, computes rules 1 to 4 and 7
against every tweak of its folder; `check_overlaps.py` applies rules 1 to 3 to every pair of tweaks that can be ticked
together. What building them settled:
- a claim on the OS's own slots (k 0–22, record slots 0–23 but 18) is refused, and so is a stock combination taken
  without saying what it replaces (§4.5);
- a formatter belongs to its interface object, whatever the scopes: an object has one formatter pointer, and SAMPLE's
  Cutoff shares object 53 with SNARE's SWEEP;
- two identical gesture claims (the same meaning) are allowed; a slot is a number, so two holders of one slot clash
  even when they mean the same;
- a package meets the builder's shortcut layer only with the claims beyond its shortcuts, which are the layer's
  claims narrowed to its machine;
- a build whose own packages clash is refused; a package's own tweak is not compared with a build that contains it;
- two drawings of one view in scopes that meet are a warning, not a conflict.

SAMPLE alone conflicts with 268 tweaks of model27's folder, SAMPLE with the reference machine with 269. The claims now
compute lfo-plus's clash with the shortcut layer, which was declared by hand (§5.2). Of the folder's 35 137 declared
conflict pairs, the bytes show 35 118, the claims alone 2 (lfo-plus with the track filter and with REF2), both 9;
8 are shown by their text only, and keep it (§9, step 5). Where two cards of the web flasher exclude each other and no
hand-written reason exists, the page says the first kind of clash the claims show.

### 4.5 Claims as model27 reads them

What a mod takes beyond its writes, in one shape, read and checked by model27's `tools/claims.py`:

- **Where they live:** a package's claims in its manifest, a part's in its `part.json`, the tweaks of a folder in one
  declared file (model27: `tweaks/model-cycles_OS1.13/claims/claims.json`), one entry per tweak or family, each with
  the note that shows it and what is still open.
- **Scopes:** `all` (every track and sound), `machine` (a package's own machine), `machine:<NAME>` (a stock machine,
  MACRO, ACID or a package). A package's claims default to its own machine, a tweak's and a part's to every track.
  Two scopes meet when one is `all` or both name the same machine.
- **Numbers and the rest:**
  - `sound_slots` (k 0–32), `record_slots` (0–31 as far as read), `descriptors` (the OS's unused 1–4), as numbers
    or as `{"slots": [first, last], "scope": ...}`;
  - `saved` (`{"where", "bytes"}`), `formatters` (`{"knob", "object", "replaces"}`), `plock_rows` (true when the mod
    re-numbers the p-lock rows of k ≥ 23);
  - `midi` (`{"cc": [...], "nrpn": ["MSB:LSB"]}`), `ram` (`[first, end)`), `views` (`main`, `knob`).
- **A key combination** is written as the table of §5.4 writes it, the held keys then one action joined by `+`:
  `TRACK + TRIG 1-6`, `MACHINES + COLOR/SHAPE/SWEEP/CONTOUR`. `1-6` and `/` expand; the tools compare *atoms*, one
  set of held keys and one action each. The words:
  - the keys by `isKeyHeld`'s code: FUNC 1, TRACK 2, RETRIG 4, MACHINES 5, PUNCH 6, GATE 7, LFO 8, RECORD 9,
    PLAY 10, STOP 11, TRIG 1–16 as 16–31, and PAGE and PATTERN, whose codes are not traced;
  - the knobs: PITCH, DECAY, COLOR, SHAPE, SWEEP, CONTOUR, and DATA for LEVEL/DATA;
  - NOTES (notes played) and DATA-PRESS.
- **A situation** says where it acts: `params` (the parameter pages), `grid`, `keys`, `page:<name>`, `menu:<name>`,
  or `any`. It is the user's situation, not the view whose code reads the keys: views are served from the end of
  their list, so a view higher up takes a combination from every view below it (Control All Config's TRACK + TRIG 1–6
  shadows any reader in `KeyboardView`). Two named situations exclude each other; `any` meets all.
- **The stock combinations,** as far as they are read (model27's `REGISTRY.md` §8):
  - FUNC + knob (fine adjust), TRACK + knob (Control All), LFO + knob (LFO destination), TRIG + knob (p-lock);
  - TRACK + PUNCH/GATE, FUNC + TRACK, FUNC + MACHINES, FUNC + RETRIG, TRACK + RECORD/PLAY/STOP;
  - a press on PUNCH or GATE (it switches them on the selected track; a held RETRIG is not read, so RETRIG + PUNCH
    does the same), FUNC + PUNCH/GATE (it opens a view);
  - RETRIG held + notes (the retrig), a TRIG held + PAGE in the grid (the page turns), LFO on the LFO page (a
    hypothesis).

  A claim that takes one says what it did, in `replaces`.

The track filter's entry in model27's `claims.json`:

```json
{"ids": ["track-filter"], "source": "notes/47",
 "claims": {"sound_slots": [28, 29, 30, 31], "record_slots": [18, 27, 30, 31], "plock_rows": true,
            "descriptors": [1, 2, 3, 4], "midi": {"cc": [74, 71], "nrpn": ["1:20", "1:21"]},
            "gestures": [{"combo": "MACHINES + COLOR/SHAPE/SWEEP/CONTOUR", "situation": "params",
                          "meaning": "Filter Env / Env Decay / Filter Cutoff / Filter Reso"}]}}
```

## 5. The number registry

Numbers that land in users' data cannot be assigned by the builder: a sound stores its machine at `+38`, its
parameters in k slots, its extras in record slots; a project stores settings. A build that assigns them differently
breaks every saved project. The registry lives in the repository as one file, is appended to, never re-ordered, and
the builder refuses a number that is not in it.

**As built in model27:** there is no separate number file.
- Each mod declares the numbers it takes as claims (§4.5).
- The tools refuse two mods that share one in scopes that meet (§4.4).
- `REGISTRY.md` §6 and §9 list who holds which, generated from the claims.
- Machine ids follow option C (§5.1).

### 5.1 Machine ids

**Today:** 0–5 are the stock machines (KICK, SNARE, METAL, PERC, TONE, CHORD). With Model-TG its Sampler is 6 and
the first added machine is 7 (`TG_FIRST`); without, the first added machine is 6. Added machines are numbered in the
build's order: the Syntakt engines, then MACRO, then ACID last (the flasher cards say so). SAMPLE and sdvintage-7th
are 6 in their own builds; sdvintage-exact replaces SNARE at 1. **So a machine's id depends on the combination, and
a project does not move between two builds.** model27's composer numbers machines in the order it is given, SAMPLE
first, so SAMPLE is 6 in every build there; permanent ids for every package wait for the decision below.

**Options:**

- **A. Stable ids with placeholders.** Each machine package gets a permanent id from 6 upward. A build fills the
  table up to the highest chosen id; unused ids get a placeholder entry (a name like `----`, silent `update`/`render`,
  five dummy descriptors). Simple, robust, costs a slot per gap; with 6 added slots in the current layout the id
  space is small unless the layout grows.
- **B. Translation at the record hooks.** The id in the sound stays the build's index, but the save hook writes the
  package's permanent id into a record slot and the load hook maps it back through a table the build carries.
  No wasted slots; one more thing every build must get right, and sounds saved by stock builds carry no mapping.
- **C. Today's rule, documented.** The build report names the index of every machine; projects belong to a build.

**Decided in 0.3: option C**
([model27 notes/62](https://github.com/durchgemacht/model27/blob/main/notes/62-level-b4-stable-machine-ids.md) §9).
- A machine's number is its place in the build.
- A track saved with another build's machines plays whatever machine has its number now. That is a happy accident,
  not a fault.
- A user who wants the same sounds after a new build keeps one project per build
  ([notes/64](https://github.com/durchgemacht/model27/blob/main/notes/64-projects-and-machines.md), written for
  users).

The recommendation of 0.2, option A, is withdrawn.

### 5.2 Sound parameter slots k

The sound holds k 0–32; the stock OS uses 0–22 (notes/47 §4.3). Assigned today (notes/56 §3.6, One-Shot TECHNICAL
§6; model27's `REGISTRY.md` §6 generates this map from the claims):

| k | Holder | Content | Status |
|---|---|---|---|
| 0–22 | OS | stock parameters | fixed |
| 23–27 | Model-TG | its filter (k 24) and more | in use |
| 25–27 | One-Shot | sample key: inode, id high, id low | in use; clashes with Model-TG (declared) |
| 28–31 | track-filter | Cutoff, Reso, Env, Decay, through descriptors 1–4 | in use |
| 28–32 | lfo-plus | LFO 2 | in use; clashes with track-filter (declared) |
| 28 (two bits) | One-Shot loop/reverse | proposed, not built | would clash with track-filter and lfo-plus |
| 28–31 | model27's shortcut layer | the four shortcuts of the track's machine, machine-scoped (REF2: k 28, Octave); they stay when a track changes machine, and a new sound starts at 0 | in use in model27; the clashes with the track filter and with lfo-plus are computed from the claims (lfo-plus packs k 28–32 into the same record slots from inside the stock save and load, at other bytes, which no byte check sees) |
| 32 | model27's shortcut layer (0.2) | the two switches of the track's machine: bit 8 RETRIG + PUNCH, bit 9 RETRIG + GATE, the other bits 0; machine-scoped (REF2: Square, Tone open); they stay when a track changes machine | in use in model27; lfo-plus holds k 32 too and already clashes with the layer |

**No k is free.** New persistent parameters need either an agreement between packages (lfo-plus could keep LFO 2 in
memory when the filter is chosen, notes/56 §3.6) or another home in the sound; the 14 bytes after k 32 (`+86`–`+99`)
have not been read and are **[to verify]** before anyone takes them.

### 5.3 Sound record slots

A record stores the sound's parameters as 16-bit slots at `+28 + 2 · slot`; the OS saves k 0–22 through the table at
`0x4010ee4c` and loads slots 0–23 through `0x4010eea8`, slot 18 going to k 0 and so unused (notes/47 §4.3). Assigned
(model27's `REGISTRY.md` §6 generates this map from the claims):

| Slot | Holder | Content |
|---|---|---|
| 0–23 | OS | stock, except 18 |
| 18 | track-filter | one of its four extra values |
| 24–29 | Model-TG | (notes/47 §11) |
| 24–26 | One-Shot | sample key |
| 27 | One-Shot | filter mark (since 0.5) |
| 27 | track-filter | one of its values; clash with One-Shot hidden behind the `sound.record` conflict today |
| 30–31 | track-filter | Cutoff, Reso (`+88`, `+90`) |
| 30–31 | lfo-plus | LFO 2 (`+88`–`+91`, the format of DavidCaruso's firmware, notes/56) |
| 28–31 | model27's shortcut layer | k 28–31, for every sound (`+84`–`+91`); a record from the official OS reads 0 |
| 18 | model27's shortcut layer (0.2) | k 32, the switches, for every sound (`+64`). The stock save never writes slot 18 (its table `0x4010ee4c` maps k 18–22 to slots 19–23), and the stock load puts it on k 0 after slot 0 (`0x4010eea8`), so the stock OS ends every load with k 0 = 0; the layer's load copies slot 18 into k 32 and clears k 0 again. The official OS loads such a record with the switches on k 0 and k 32 at 0. Clashes with the track filter's slot 18 |

### 5.4 Gestures

As declared in model27's claims (its `REGISTRY.md` §8 generates this table), then the stock combinations the
standard names a point for; the other stock ones are in §4.5:

| Gesture | Holder | Scope |
|---|---|---|
| MACHINES held + COLOR/SHAPE/SWEEP/CONTOUR | track-filter (Filter Env, Env Decay, Cutoff, Reso) | all tracks, parameter pages |
| MACHINES held + DECAY/SWEEP/CONTOUR | Model-TG (Attack, Filter, Resonance) | all tracks, parameter pages; shares SWEEP and CONTOUR with the track filter |
| FUNC + PITCH | MACRO | MACRO tracks: FINE as lo-fi RATE/BITS; replaces FINE, the stock fine adjust of PITCH |
| TRACK + TRIG 1–6 | control-all-config | configuration of Control All; TRACK + TRIG 1 is also taken by Modded-Cycles PR #92 |
| TRACK held + FUNC | latching-mute | latches the mute mode; the stock meaning is not read |
| a TRIG held + PAGE | trig-preview | the grid, sequencer stopped or paused: the step sounds; replaces PAGE turning the page |
| RETRIG held + notes | arp | an arpeggio when Arp is not OFF; replaces the retrig |
| TRIG 1–16 | chord-keys | CHORD tracks, in its Keys mode: the chords of a scale |
| FUNC + a press on LEVEL/DATA | midi-live-both | CONFIG › MIDI › PORTS, on the OUT/THRU line |
| LFO | lfo-plus | on the LFO page: LFO 2, 3 and 4; replaces closing the page (a hypothesis) |
| RETRIG + COLOR/SHAPE/SWEEP/CONTOUR | the `shortcut` point (model27) | the tracks of a machine that declares the shortcut; not with FUNC, TRACK, LFO or a held trig. In the stock OS, RETRIG held changes nothing while a knob turns |
| RETRIG + PUNCH/GATE | the `shortcut` point (model27, since 0.5) | the tracks of a machine that declares the switch; not with TRACK. In the stock OS a held RETRIG is not read on these presses, so the layer takes the press from the stock switch; their LEDs show the switches while RETRIG is held |
| PUNCH/GATE pressed | stock switch (`0x4001ac4a`, parameters 44 and 45) | the selected track, all six with TRACK; with RETRIG held, the `shortcut` point's switches on the tracks of a machine that declares them |
| TRACK + PUNCH/GATE | stock Control All (`0x4001ac4a`) | all tracks; One-Shot's loop/reverse study would have taken it on SAMPLE tracks |
| FUNC + TRACK | stock Track Setup menu | the `menu.track_setup` point |
| FUNC + MACHINES | stock PRESET browser | the `browser.open` point |

### 5.5 MIDI

CC 74 and 71, NRPN 1:20 and 1:21 belong to the track-filter (notes/47). The stock OS's own CC and NRPN map is not
in this document yet **[to collect from the OS or the manual]**; until then a new MIDI claim is checked only against
other packages. model27 checks MIDI claims that way (§4.4).

### 5.6 Tweak order numbers

`order` sorts the tweaks and the chains. Today: 25 MACRO, 26 ACID, 27 One-Shot (in its own repository since 0.7 and
in model27; up to 0.6 it was 26, a clash the checker reported as "two payloads at the same order"), 48 track-filter,
60 lfo-plus.
Proposal: machines 20–39, track-chain stages 40–59, UI and sequencer 60–79, MIDI 80–89, cosmetic 90–99, assigned
with the id.

### 5.7 Masks, caves and payload windows

These are *allocations*, not registry numbers: `registry.py` already derives who holds which sprite mask, which
cave bytes and which payload range, and `REGISTRY.md` is the record. The standard changes nothing here except that
the manifest's `claims.masks` is generated from the same source.

## 6. The cost model

The budget is a block: 32 frames at 48 kHz, 666.7 µs. Modded-Cycles' model (notes/27, used in notes/47 §9) counts
166 667 cycles per block at 250 MHz and 1.54 cycles per instruction. Measured on the device (notes/25 §1):

| Part | Share of a block |
|---|---|
| mixer and effects | about 30 % |
| each stock voice that sounds | about 8 % |
| each Syntakt voice that sounds | about 15 % |
| heavy load of notes/36 (Syntakt engines, Model-TG, governor) | about 44 % |
| six filtered and modulated tracks of the track-filter | under 5 % more |

Above about 85 % average the interface starves; the governor (notes/25, 28, 30, 36) mutes the least audible voice
when the sustained load passes its thresholds. It covers voices only, not chain stages.

**Two methods, both to be named in the manifest:**

- **Emulation:** instructions per block per voice, counted by `tools/emu/mcengine.py` in the real voice loop
  (notes/23 §1, notes/43 §8): first block, then mean and maximum over the next 21 blocks, on a defined pattern. MACRO
  reports 6 398–16 522 mean per model, up to 18 502 maximum; TONE is about 5 300 by the same count (from the ratios
  in notes/43 §8). The track-filter reports 643–812 per filtered track. **The emulator does not time memory:** the
  Syntakt engines counted like stock machines and cost nearly twice as much on the device (notes/23 §4).
- **Device:** the share of a block, read with the `syntakt-meter` diagnostic firmware (peak and average in place of
  the machine names, notes/23 §5). This is the number that counts; the emulation number is the one every author can
  produce.

A package declares `cost.emulation {mean, max, pattern}` and, when it has it, `cost.device {percent_per_voice}` or
`cost.device {percent_per_block}` for a chain stage. The builder sums the worst case of the chosen packages (six
voices of the heaviest machine plus every stage) and warns above a threshold to be chosen (section 10).

## 7. Examples: SAMPLE, MACRO, track-filter, REF2

§7.1 is SAMPLE's manifest as model27's composer reads it (One-Shot 0.8), §7.4 REF2's, the template for a machine that
uses every point SAMPLE uses and a shortcut. §7.2 and §7.3 describe MACRO and the track
filter as in Modded-Cycles 1.56, in the shape of draft 0.1: nobody has packaged them yet, their hooks are listed under
`claims` instead of in `uses` (§4.2), and values marked `derived` are what the tools would compute, shown so that
they can be checked against `25-macro.json` and `48-track-filter.json`.

### 7.1 SAMPLE (Cycles One-Shot 0.8, as built in model27)

model27's [`tools/machines/sampler/manifest.json`](https://github.com/durchgemacht/model27/blob/main/tools/machines/sampler/manifest.json)
at `a279f3f`, unchanged:

```json
{
 "schema": "cycles-mod/0.1",
 "id": "sampler",
 "version": "0.8",
 "name": "SAMPLE machine: one-shot sampler for samples on the +Drive (Cycles One-Shot 0.8)",
 "description": "Plays a sample from the +Drive once per trig: 6-tap polyphase at 96 kHz, the stock decimator, the amp chain set up like TONE, then the Model:Samples' filter. The sample is chosen in the PRESET browser and saved with the sound; the main screen shows its waveform.",
 "license": "MIT",
 "credit": {"kind": "own", "who": "durchgemacht and Claude", "repo": "durchgemacht/cycles-one-shot"},
 "source": "https://github.com/durchgemacht/cycles-one-shot/tree/f28ef2f81dd0a4872705c9fb6021462eb5f0bdbf",
 "device": "Model:Cycles",
 "os": "1.13",
 "order": 27,
 "build": {
  "sources": ["src/smp_core.c", "src/smp_coeffs.c", "src/smp_filter.c", "src/smp_filter_tab.c", "src/smp_filter_cf.S",
              "machine/sampler.c", "machine/pool.c", "machine/view.c", "machine/hooks.S"],
  "include": ["src", "machine"],
  "cflags": ["-std=c99", "-mcpu=54418", "-O2", "-ffreestanding", "-fno-builtin", "-fno-tree-loop-distribute-patterns",
             "-nostdlib", "-fno-pic", "-fno-common", "-ffunction-sections", "-fdata-sections", "-Wall", "-Wextra",
             "-Werror"],
  "defines": {"CUTOFF_OBJECT": "53", "POOL_BASE": "0x43800000u", "POOL_LIMIT": "0x46700000u", "CM_SYMBOLS": "1"},
  "objects": ["sampler.o"],
  "patch_points": [
   {"symbol": "cm_machine_index", "kind": "machine_index"},
   {"symbol": "cm_descriptor_base", "kind": "descriptor_base"}
  ],
  "symbols": {"update": "sampler_update", "render": "sampler_render"},
  "keep": ["smp_hook_open", "smp_hook_save2", "smp_hook_save1", "smp_hook_load", "smp_hook_tick", "smp_main_draw", "smp_knob_draw"]
 },
 "uses": [
  {"point": "machine", "name": "SAMPLE", "image": 4, "decay_default": 127,
   "knobs": [
    {"long": "Start", "short": "STRT", "default": 0, "min": 0, "max": 120},
    {"long": "Length", "short": "LEN", "default": 120, "min": 0, "max": 120},
    {"long": "Cutoff", "short": "FREQ", "default": 64},
    {"long": "Reso", "short": "RESO", "default": 0}]},
  {"point": "browser.open", "hook": {"at": "0x400a6602", "stock": "4eb9400a3052", "op": "jsr", "symbol": "smp_hook_open"},
   "why": "the browser reads the opened file: a sample file on a SAMPLE track names the sample instead"},
  {"point": "sound.record", "hook": {"at": "0x4005afa0", "stock": "4feffff448d71c00", "op": "jmp", "symbol": "smp_hook_save2"},
   "why": "sound to record, version 2: the sample's key into record slots 24..27"},
  {"point": "sound.record", "hook": {"at": "0x4005b054", "stock": "4feffff448d71c00", "op": "jmp", "symbol": "smp_hook_save1"},
   "why": "sound to record, version 1"},
  {"point": "sound.record", "hook": {"at": "0x4005aecc", "stock": "4feffff448d71c00", "op": "jmp", "symbol": "smp_hook_load"},
   "why": "record to sound: the key back"},
  {"point": "screen.tick", "hook": {"at": "0x400075f0", "stock": "4eb940076be4", "op": "jsr", "symbol": "smp_hook_tick"},
   "why": "the screen's redraw, about 30 Hz: loads and frees samples"},
  {"point": "view.main", "vtable": {"at": "0x400ffe30", "stock_ptr": "0x4001b22a", "symbol": "smp_main_draw"},
   "why": "MainScreenView's draw method: the waveform on a SAMPLE track"},
  {"point": "view.knob", "vtable": {"at": "0x4010053c", "stock_ptr": "0x4001e40a", "symbol": "smp_knob_draw"},
   "why": "the knob view's draw method"},
  {"point": "memory.ram", "range": ["0x43800000", "0x46700000"], "why": "the sample pool, 47 MB"}
 ],
 "claims": {
  "sound_slots": [25, 26, 27],
  "record_slots": [24, 25, 26, 27],
  "ram": [["0x43800000", "0x46700000"]],
  "formatters": [{"knob": "Cutoff", "object": 53, "replaces": "0x400456c8", "scope": "all",
                  "why": "an interface object has one formatter, and object 53 is also SNARE's SWEEP: smp_fmt_cutoff falls back to the stock one off SAMPLE's tracks and is put in only while the stock one is there (machine/view.c)"}],
  "gestures": [],
  "views": ["main", "knob"]
 },
 "cost": {
  "emulation": {"status": "not measured with mcengine.py; One-Shot's README states about 800 instructions per track and block for the filter alone"},
  "device": {"status": "not measured"}
 },
 "stock_behaviour": {
  "changes": ["the filter runs inside the machine, before Volume + Dist; the Model:Samples has it after"],
  "fixes": [],
  "unchanged": ["at Cutoff 0 the track is untouched bit for bit", "presets on other machines load as before"]
 },
 "compat": {
  "built_against": ["modded-cycles 6a9742a", "modded-cycles 95b63b7", "model27"],
  "requires": [],
  "conflicts_declared": [{"with": "sample-preview", "why": "plays sample files from the same browser and needs Model-TG"},
                         {"with": "sample-preview-st", "why": "the same, on Model-TG's base"}]
 },
 "tests": {"emulation": {"tool": "digiemu", "date": "2026-10-22", "build": "the composed object registered by the registrar runtime, alone (as the web flasher builds it) and with refmachine"},
           "hardware": {"date": "2026-10-22", "by": "maintainer", "build": "SAMPLE alone as the web flasher builds it (notes/60 §13) and as machine 6 next to refmachine (the pair, notes/60 §12); One-Shot's own build (0.7 = 0.8 without CM_SYMBOLS) on 2026-10-21 by the author"},
           "status": "tested"}
}
```

**What the composer computes from it** (model27, 2026-10-22):

- machine 6, descriptors 76 to 80; code and tables, 30 527 bytes, at `0x43000000`; variables, 73 784 bytes, from
  `0x43036000`; the registrar's parameter block for one machine. 123 writes (116 of the registration, 7 of the
  package's hooks and vtable entries), 36 096 bytes added to the image: model27's `27-sampler.json`;
- 268 conflicts against model27's tweak folder: MACRO, ACID and the Syntakt engines (the same machine tables, other
  bytes), Model-TG and the track filter (the same sound-record hooks), sample preview (declared), the other
  machine-adding tweaks;
- with model27's reference machine next to it as machine 7: 269, the union of the two packages' lists. Composing
  adds no conflict;
- `memory.ram` starts above the build's payload (`0x43000000`–`0x43048038` alone): since One-Shot 0.7 the pool no
  longer clashes with the payload window;
- the claims (2026-10-23): sound slots 25–27 against Model-TG's 23–27, record slot 27 against the track filter, as
  rule 1 (the same sound-record hooks) already found; declared only: sample preview, whose reason is text; a warning:
  level-pan-values draws on the main screen too.

### 7.2 MACRO (Modded-Cycles 1.56)

```json
{
  "schema": "cycles-mod/0.1",
  "id": "macro",
  "version": "1.56",
  "name": "Machine MACRO",
  "description": "An added machine: the 47 models of Braids by Emilie Gillet. SHAPE chooses the model, COLOR and SWEEP set it, CONTOUR opens the timbre with the envelope, FUNC + PITCH gives lo-fi (RATE left, BITS right).",
  "license": "LICENSE-Braids",
  "credit": {"kind": "based", "who": "Émilie Gillet", "repo": "pichenettes/eurorack"},
  "device": "Model:Cycles", "os": "1.13",
  "order": 25,
  "build": {"objects": ["macro.o"], "symbols": {"update": "macro_update", "render": "macro_render"}},
  "uses": [
    {"point": "machine", "image": 4, "decay_default": 50,
     "knobs": [
       {"long": "Timbre",     "short": "TIMB", "default": 64},
       {"long": "Model",      "short": "MODL", "default": 0, "min": 0, "max": 46},
       {"long": "Color",      "short": "COLR", "default": 64},
       {"long": "Env Timbre", "short": "ENVT", "default": 0}]},
    {"point": "gesture", "combo": "FUNC + PITCH", "scope": "MACRO tracks", "meaning": "FINE as lo-fi RATE / BITS, one step per click, OFF at the centre",
     "note": "implemented inside the added-machine detours (notes/55 §8), no extra hook address"}
  ],
  "claims": {
    "hooks": "the added-machine mechanics only: boot hook 0x400004b2, moved tables, bounds, detours (122 writes, the same addresses as ACID)",
    "payload": {"ram_bytes": 324576, "image_bytes": 105152, "window": "alone"},
    "sound_slots": [], "record_slots": [], "masks": []
  },
  "cost": {
    "emulation": {"tool": "mcengine.py", "per_model": "notes/43 §8", "mean_range": [6398, 16522], "max": 18502, "tone_reference": 5300},
    "device": {"status": "13 models at 48 kHz tested on the device 2026-10-11; no percent recorded"}
  },
  "stock_behaviour": {
    "changes": ["FUNC + PITCH on MACRO tracks sets FINE instead of PITCH in semitones"],
    "unchanged": ["other tracks and machines"]
  },
  "compat": {"requires_variant": {"model-tg": "32-macro-tg"}, "conflicts_declared": []},
  "tests": {"emulation": {"tool": "test_macro.py"}, "hardware": {"date": "2026-10-12", "by": "maintainer"}, "status": "tested"}
}
```

Computed conflicts: none with ACID or SAMPLE under the shared registrar; sdvintage-exact (replaces SNARE by other
mechanics) stays declared; the 72-entry list of today shrinks to the registry and the limits.

### 7.3 track-filter (Modded-Cycles 1.56)

```json
{
  "schema": "cycles-mod/0.1",
  "id": "track-filter",
  "version": "1.56",
  "name": "A filter per track: lowpass, OFF, highpass",
  "description": "MACHINES held + SWEEP = Filter Cutoff (L64 ... L1, OFF, H1 ... H63), + CONTOUR = Reso, + COLOR = Env, + SHAPE = Env Decay. A 2-pole trapezoidal SVF after Volume + Dist, before pan, level and sends, the Model:Samples' order. At OFF the track is untouched bit for bit.",
  "license": "MIT",
  "credit": {"kind": "own", "who": "Modded-Cycles"},
  "device": "Model:Cycles", "os": "1.13",
  "order": 48,
  "build": {"objects": ["track_filter.o", "tf_ui.o"], "symbols": {"stage": "tf_post"}},
  "uses": [
    {"point": "track.post_voldist", "symbol": "tf_post", "order": 48},
    {"point": "sound.record", "after_save": "tf_save", "after_load": "tf_load"},
    {"point": "param.tables", "symbol": "tf_boot", "why": "copies its four descriptors into the active parameter table at boot"},
    {"point": "sound.copy", "symbol": "see tf_ui.S", "why": "lets the sound's slot copy keep k 28-31"},
    {"point": "param.descriptor", "descriptors": [1, 2, 3, 4], "sound_slots": [28, 29, 30, 31],
     "names": [["Filter Cutoff", "FREQ"], ["Filter Reso", "RESO"], ["Filter Env", "FENV"], ["Env Decay", "FDEC"]]},
    {"point": "gesture", "combo": "MACHINES held + COLOR/SHAPE/SWEEP/CONTOUR", "scope": "all tracks"},
    {"point": "midi", "cc": [74, 71], "nrpn": ["1:20", "1:21"]},
    {"point": "memory.mask", "masks": "15 masks 35x35, 3001 bytes of code, tables and state"}
  ],
  "claims": {
    "hooks": [
      {"at": "0x4004486c", "stock": "4eb94005a274", "point": "param.tables"},
      {"at": "0x40059852", "stock": "4eb940056f58", "point": "track.post_voldist"},
      {"at": "0x4005aa34", "stock": "7417b480640c", "point": "sound.copy"},
      {"at": "0x4005aecc", "stock": "4feffff448d7", "point": "sound.record"},
      {"at": "0x4005afa0", "stock": "4feffff448d7", "point": "sound.record"},
      {"at": "0x4005b054", "stock": "4feffff448d7", "point": "sound.record"}],
    "payload": {"image_bytes": 0, "note": "no append: code lives in freed masks"},
    "sound_slots": [28, 29, 30, 31],
    "record_slots": [18, 27, 30, 31]
  },
  "cost": {
    "emulation": {"tool": "mcengine.py", "per_filtered_track": [643, 812], "six_tracks_off": 350, "six_tracks_modulated_max": 5220},
    "device": {"percent_per_block": "under 5 % for six filtered, modulated tracks (bound verified on the device)"}
  },
  "stock_behaviour": {
    "changes": ["MACHINES held + knob on every track"],
    "fixes": ["p-locks of parameters k >= 23 were saved onto LFO Speed / Multiplier or lost by the stock OS (notes/47 §4.3)"],
    "unchanged": ["at OFF the track is untouched bit for bit, proven on the real code with the other mods"]
  },
  "compat": {"conflicts_declared": [{"with": "lfo-plus", "why": "both re-number the p-lock rows of k >= 23"}]},
  "tests": {"emulation": {"tool": "test_track_filter.py"}, "hardware": {"date": "2026-10-11", "by": "maintainer"}, "status": "tested"}
}
```

Computed conflicts, from model27's claims (2026-10-23):
- Model-TG: MACHINES held + SWEEP and + CONTOUR with another meaning, and record slot 27. The chain on `sound.record`
  would remove the hook conflict, but the gesture stays exclusive.
- lfo-plus: sound slots 28–31 against 28–32, record slots 30–31 against 30–31, and both re-number the p-lock
  rows.
- One-Shot: record slot 27. The `sound.record` chain replaces today's hook conflict.
- One-Shot's loop/reverse study: the bits of k 28.

### 7.4 REF2 (model27's second reference machine)

REF2 exists to be copied. It is REF's contract (a triangle and noise through the stock chain) plus one entry for
every point a machine may use today:
- a shortcut (RETRIG + COLOR = Octave, −2 to +2);
- two switches (RETRIG + PUNCH = Square, RETRIG + GATE = Tone open), since 0.5;
- its main screen label and its knob view;
- a tick that blinks the label's dot;
- a browser hook that shows the name of a file opened on its track.

It is never offered to users. Its manifest as model27's composer reads it, at
[`tools/machines/refmachine2/manifest.json`](https://github.com/durchgemacht/model27/blob/main/tools/machines/refmachine2/manifest.json)
after the switches (model27 notes/63 §18):

```json
{
 "schema": "cycles-mod/0.1",
 "id": "refmachine2",
 "version": "0.1",
 "name": "REF2 machine: model27's second reference machine, a triangle and noise through the stock chain",
 "description": "The template for a machine that uses everything the standard lets a machine use (notes/63). REF's contract with a triangle in place of REF's parabola (B4a); a shortcut, RETRIG + COLOR = Octave, -2 to +2, saved with the sound by the builder's shortcut layer (B4b); its own main screen label and knob view (B4c), and a tick that blinks the label's dot and a browser hook that shows the name of a file opened on its track (B4d), through the builder's dispatchers when SAMPLE takes the same points; two switches, RETRIG + PUNCH = Square and RETRIG + GATE = Tone open, flipped and saved in k 32 by the shortcut layer, read in the note's sound and shown left of the label (notes/63 §16). Never offered to users.",
 "license": "MIT",
 "credit": {"kind": "own", "who": "durchgemacht and Claude", "repo": "durchgemacht/model27"},
 "device": "Model:Cycles",
 "os": "1.13",
 "order": 29,
 "build": {
  "sources": ["refmachine2.c", "ref2_view.c", "ref2_open.S"],
  "include": ["."],
  "cflags": ["-std=c99", "-mcpu=54418", "-O2", "-ffreestanding", "-fno-builtin", "-fno-tree-loop-distribute-patterns",
             "-nostdlib", "-fno-pic", "-fno-common", "-ffunction-sections", "-fdata-sections", "-Wall", "-Wextra",
             "-Werror"],
  "defines": {},
  "objects": ["refmachine2.o"],
  "patch_points": [
   {"symbol": "ref2_machine_index", "kind": "machine_index"}
  ],
  "symbols": {"update": "ref2_update", "render": "ref2_render"},
  "keep": ["ref2_main_draw", "ref2_knob_draw", "ref2_hook_tick", "ref2_hook_open"]
 },
 "uses": [
  {"point": "machine", "name": "REF2", "image": 4, "decay_default": 64,
   "knobs": [
    {"long": "Tone", "short": "TONE", "default": 127},
    {"long": "Noise", "short": "NOIS", "default": 0},
    {"long": "Detune", "short": "DTUN", "default": 64},
    {"long": "Level", "short": "LEVL", "default": 100}]},
  {"point": "shortcut", "key": "RETRIG", "knob": "COLOR",
   "param": {"long": "Octave", "short": "OCT", "min": -2, "max": 2, "default": 0},
   "why": "the note's octave, -2 to +2: k 28, which the voice reads at p + 0x38; the shortcut layer saves it"},
  {"point": "shortcut", "key": "RETRIG", "press": "PUNCH",
   "why": "Square, a switch: bit 8 of k 32, which the voice reads in the note's sound; the shortcut layer flips and saves it"},
  {"point": "shortcut", "key": "RETRIG", "press": "GATE",
   "why": "Tone open, a switch: bit 9 of k 32, which the voice reads in the note's sound; the shortcut layer flips and saves it"},
  {"point": "view.main", "vtable": {"at": "0x400ffe30", "stock_ptr": "0x4001b22a", "symbol": "ref2_main_draw"},
   "why": "MainScreenView's draw method: the stock screen, then OCT and the Octave in its free band on a REF2 track"},
  {"point": "view.knob", "vtable": {"at": "0x4010053c", "stock_ptr": "0x4001e40a", "symbol": "ref2_knob_draw"},
   "why": "the knob view's draw method: the Octave as five boxes while RETRIG + COLOR turns it on a REF2 track"},
  {"point": "screen.tick", "hook": {"at": "0x400075f0", "stock": "4eb940076be4", "op": "jsr", "symbol": "ref2_hook_tick"},
   "why": "the screen's redraw, about 30 Hz: counts the ticks, and the label's dot blinks with them"},
  {"point": "browser.open", "hook": {"at": "0x400a6602", "stock": "4eb9400a3052", "op": "jsr", "symbol": "ref2_hook_open"},
   "why": "the browser reads the file it opens: on a REF2 track the label shows its name, then the stock open reads it"}
 ],
 "claims": {"sound_slots": [28, 32], "record_slots": [18, 28], "ram": [], "gestures": [{"combo": "RETRIG + COLOR", "situation": "params", "meaning": "Octave"}, {"combo": "RETRIG + PUNCH", "meaning": "Square"}, {"combo": "RETRIG + GATE", "meaning": "Tone open"}], "views": ["main", "knob"]},
 "cost": {"emulation": {"status": "not measured"}, "device": {"status": "not measured"}},
 "stock_behaviour": {"changes": ["on its tracks, RETRIG held + PUNCH or GATE flips its switch instead of PUNCH or GATE, and their LEDs show its switches while RETRIG is held"], "fixes": [], "unchanged": ["other tracks and machines", "TRACK + PUNCH or GATE (Control All), with or without RETRIG"]},
 "compat": {"built_against": ["model27"], "requires": [], "conflicts_declared": []},
 "tests": {"emulation": {"tool": "tools/emu/test_compose.py, tools/emu/test_shortcuts.py and tools/emu/test_views.py (Unicorn, the OS's own code, notes/63 §10, §11.4, §12.4, §13.4, and the switches §18.4)", "date": "2026-10-26", "build": "alone, as machine 6 or 7 next to sampler, and as machine 8 of sampler, refmachine, refmachine2"}, "hardware": {"date": "2026-10-26", "by": "maintainer", "build": "sampler + refmachine2 from tools/compose.py (notes/63 §20.1, .syx 8d75e346, MAIN OS 6ee148f3): \"alles wie erwartet\", the list of notes/63 §20.2 (§20.4); REF2 alone and its browser hook not run"}, "status": "experimental"}
}
```

**How each entry is written in its sources** (`refmachine2.c`, `ref2_view.c`, `ref2_open.S`):
- **The voice** reads the shortcut's value in k 28, `params + 0x38`, and clamps it to the declared range.
- **The switches** are read by the voice every block in the sound its note plays (§2.3): k 32 at
  `0x800017ec[t] + 84`, or at `0x800015ae + 66 · t + 64` without one; bit 8 makes the triangle a square, bit 9
  leaves the lowpass out, so a switch acts at once, even on a note already playing. The main draw shows them as
  `S`, `O` or `SO` left of the label.
- **The machine number** is the patch point `ref2_machine_index`, read as `(int32_t)(long)ref2_machine_index` and
  compared with the sound's byte `+38`.
- **The two draws** take `(view, bitmap)`. On a track of their own machine they draw (the main screen: the stock
  draw first, then the label). Otherwise they call the stock draw.
- **The tick** takes `(view)` and returns `0x40076be4(view)`.
- **The browser hook** is assembly. It keeps a0, a2 and d2–d7, notes the file's name on a REF2 track, and ends in a
  jump to the stock read `0x400a3052`.

**What the composer computes from it** (model27, 2026-10-22; the switches 2026-10-26):
- **Alone**, as machine 6: 126 writes, 3 524 bytes of code and tables, 244 bytes of variables, 10 048 bytes added to
  the image. The shortcut layer goes in with its two switch hooks, the dispatchers do not: REF2's four points are
  written directly.
- **With SAMPLE**, as machine 7: the shortcut layer with SAMPLE's three record wrappers chained behind it, and the
  dispatchers for the four shared points, with both machines' methods in the tables.
- **Proven in emulation** on the OS's own code: the knob in 1 792 to 2 240 cases per build, the value's range, clamp,
  display, save and load, a machine change, the octave, the views, the tick and the browser. The switches (model27
  notes/63 §18): the press against the stock OS in up to 2 400 cases, the LEDs in up to 640, slot 18 and k 32 in the
  record, the four sounds and a switch acting from the block it flips at.
- **On a Model:Cycles** (2026-10-26, model27 notes/63 §20.4): the build SAMPLE + REF2, as expected: the Octave and
  its views, the switches and their LEDs, a switch alone kept by the project, sounds copied, saved and loaded with
  them. Not yet: REF2 alone, and its browser hook.
- **The claims** (2026-10-23, the switches 2026-10-26): through the shortcut layer, k 28–32 and record slots 18 and
  28–31 of every sound. lfo-plus's clash is computed from them, and so are the track filter's and Model-TG's.

## 8. Builder and web flasher

**The target**, as in 0.1: an offline builder in Python reads the manifests of the chosen packages, assigns machine
ids from the registry, lays out payloads, places the objects at their addresses, fills the dispatcher tables,
applies the writes with `build.py`'s functions and writes the tweak JSON, the card and the report; the web flasher
does the same in the browser.

**As built in model27** ([notes/59](https://github.com/durchgemacht/model27/blob/main/notes/59-level-b-builder.md),
[notes/60](https://github.com/durchgemacht/model27/blob/main/notes/60-level-b2-relocator-runtime.md), increments B1
and B2, tested on a Model:Cycles on 2026-10-22):

- **The composer**, `tools/compose.py`, reads the manifests of the chosen packages in the order given, places each
  committed object with `tools/relocate.py` (§4.2), lays code and tables out from `0x43000000` and variables from
  `0x43036000`, registers all machines at once, writes each package's hooks and vtable entries, computes the
  conflicts (§4.4) and emits one tweak per selection, which `tools/build.py` applies like any other. It needs
  `python3` and the user's official file, no compiler and no assembler. Since step B4 it also adds the shortcut layer
  and the dispatchers where a build needs them (§3.3), and since the switches (0.5) the layer's press and LED hooks
  only when a machine declares a switch. Not yet: cards from manifests.
- **The registrar runtime**, `tools/machines/registrar/registrar.S`: the detours of the added machines (knob rows,
  records, the machine of a descriptor, the MACHINES page) as 826 bytes of code assembled once at `0x43033000`,
  reading a 128-byte parameter block at `0x43033780` that the composer fills per build (the number of machines, the
  first index, and for each machine its picture, first descriptor and record). The boot hook's stub is a committed
  template whose three immediates and segment table the composer fills. Until then these detours were generated and
  assembled per build.
- **Proofs:** the relocator against `ld`, byte for byte; the runtime against the per-build detours (the same writes
  except the jump sites); the OS's own code in Unicorn (model27's `tools/emu/test_compose.py`: the registrar's
  mechanics, the runtime against the per-build detours input for input, the voice loop); the digiemu emulator; a
  Model:Cycles, SAMPLE alone and SAMPLE with a second machine.
- **The generators** need the toolchain: they compile and commit the objects, the runtime and the stub template, and
  check them with `--check`.

**The web flasher, as built:** the relocator, in a form that needs no ELF reader in the browser
([model27 notes/61](https://github.com/durchgemacht/model27/blob/main/notes/61-level-b3-web-flasher.md), increment
B3; notes/63, step B4e).
- Each object reaches the page placed once at offset 0, as code plus a list of relocations (position; base: the code,
  the variables or a patch point; addend).
- The builder's two parts reach it the same way, each laid out once with all its roots.
- The registrar's writes reach it as a frame per number of machines, computed by the Python registrar.
- The page (`docs/flasher/composer.js`) fills in what depends on the packages: the registrar's tables, the shortcut
  layer's data block, the dispatchers' tables.
- `webcompose_check.sh` checks every composition it makes against the Python composer's bytes, with SAMPLE, REF and
  REF2 alone and together.
- The claims reach the page too (since 0.4): the packages' and the parts' with them, the offered tweaks' by id. The
  page applies rules 2 and 3 as the Python composer does, and `webcompose_check.sh` compares the conflicts, with
  probes of claims only. Where two cards exclude each other and no hand-written reason exists, the page says the
  first kind of clash the claims show, in English and in French.

Pre-linked variants per slot are not needed.

- **Verification stays as it is:** `check_overlaps.py` byte for byte and now claim for claim, `ref_mainos.py` for
  every card alone and every pair, `registry.py --check`, each package's emulation proof. The computed conflicts
  are an *additional* check in front of them, not a replacement.

## 9. Migration path

1. **Write the contract and the header** (level A) from the existing identical comments and `os.h`; no firmware
   changes. Every machine author can adopt it at once. *Not started.*
2. **Port One-Shot onto `gen_macro.added_machine`**: the first proof that an external machine goes through the
   shared registrar; it also removes SAMPLE's exclusion of MACRO, ACID and the Syntakt engines. Needs One-Shot's
   `SAMPLE_MACHINE` and `DESC_START` to become patch points and the sample pool to start above the alone payload
   window, or the window to move. *Done in model27:* One-Shot 0.7 moved the pool above the window, 0.8 made the two
   values patch points, and model27's composer and registrar runtime took the place of `added_machine` for packages
   (§8). SAMPLE still excludes MACRO, ACID and the Syntakt engines there, because they are not packaged.
3. **Generate manifests for the existing mods** from their generators: the derived part is already computed there;
   the declared part (gestures, scopes, stock-behaviour changes, tests) is copied from the notes and the cards.
   Authors can start by hand with [APPLYING.md](APPLYING.md). *Started:* SAMPLE and model27's reference machines have
   manifests, written by hand. The declared part of model27's other tweaks is one file of claims
   (`claims/claims.json`): 11 entries for 139 tweaks, those whose saved data, key combinations or views a tool can
   check (§4.5).
4. **Teach `registry.py` to read manifests** for §6 (saved data) and the gestures, instead of the notes; add the
   number registry file and its check. *Done in model27* without a number file: `REGISTRY.md` §6 (saved data), §8
   (key combinations), §9 (descriptors, formatters, p-lock rows, MIDI) and §10 (views) come from the claims (§5).
5. **Compute conflicts** and compare with the declared lists; keep only the declared ones that have a reason tools
   cannot see. *Done in model27* for rules 1 to 4 and 7 (§4.4), with a report of how each declared conflict is
   shown. The declared conflicts of the folder's tweaks stay as Modded-Cycles wrote them; model27's own hand entry
   (the shortcut layer's lfo-plus) is gone, computed now.
6. **Dispatcher tables** for `sound.record`, `screen.tick`, `view.*` and `menu.track_setup`, one at a time, each
   proven in emulation like a mod. *Done in model27* for `sound.record`, `view.main`, `view.knob`, `screen.tick` and
   `browser.open` (§3.3), proven in emulation with SAMPLE and REF2, and played on a Model:Cycles with both on
   2026-10-26, REF2's browser hook aside; `menu.track_setup` not started.
7. **The flasher** builds cards from manifests and fills tables, with the relocator of §8. *Done in model27* for the
   tables (`composer.js`); the cards still come from the tweaks (`gen_flasher_tweaks.py`).

## 10. Open decisions

- The cost threshold the builder warns at, and whether a chain stage counts once or per track.
- The two unread places: `track.post_voice`'s contract and the `fx.bus` location; the sound bytes `+86`–`+99`.
- The stock MIDI map for the `midi` registry.
- Who assigns registry numbers: the maintainer on merge, as with Modded-Cycles' "tested" status. In model27 the
  claims stand in for a number file (§5).
- Whether the shared header is a separate repository that One-Shot and the builders depend on, or lives in a
  builder's `tools/machines/` (Modded-Cycles', model27's).

Decided in 0.2: the web flasher uses the relocator, with objects placed once (§8). Decided in 0.3:
- machine ids follow option C (§5.1);
- shortcuts are a fixed list owned by the builder, RETRIG + COLOR/SHAPE/SWEEP/CONTOUR, machine-scoped (§3.1);
- the shared points go by machine, the tick to every machine (§3.3).

Decided in 0.4
([model27 notes/65](https://github.com/durchgemacht/model27/blob/main/notes/65-level-c-claims-check.md) §11):
- a folder's tweaks keep their claims in one declared file, each with its note (§4.5);
- a key combination is written on a fixed vocabulary and compared as atoms, in situations and scopes (§4.5);
- a claim on a stock combination says what it replaces, or it is refused; two drawings of one view are a warning;
- the web flasher says a computed reason where no hand-written one exists (§4.4).

Decided in 0.5
([model27 notes/63](https://github.com/durchgemacht/model27/blob/main/notes/63-level-b4-shortcuts-dispatchers.md)
§16.11):
- the list grows by two switches, RETRIG + PUNCH and RETRIG + GATE, each on or off and saved with the sound, in k 32
  and record slot 18 (§3.1, §5.2, §5.3);
- on the machine's tracks PUNCH's and GATE's LEDs show the switches while RETRIG is held; whether and where the screen
  shows them is the machine's;
- TRACK keeps Control All; a held trig changes nothing;
- the builder's existing shortcut layer takes them (0.2), not a new part.

## 11. Sources

Read for this draft, all local copies:

- Modded-Cycles fork at `95b63b7` (release 1.56): `tools/gen_macro.py` (`MACHINE`, `added_machine`, `other_ids`),
  `tools/gen_acid.py`, `tools/gen_syntakt_engines.py` (`LAYOUT`, `check_room`, `TG_FIRST`, `PAY_ALONE`, `PAY_TG`,
  `END_LIMIT`), `tools/gen_sdvintage_exact.py` (`IMAGE_LEN`, `CAVE`, `HOOK`), `tools/machines/acid/acid.c`,
  `tools/machines/macro/macro.cc`, `tools/build.py`, `tools/check_overlaps.py`, `tools/registry.py`,
  `tools/gen_flasher_tweaks.py`, `tweaks/model-cycles_OS1.13/REGISTRY.md`, `25-macro.json`, `26-acid.json`,
  `48-track-filter.json`; notes 18, 23, 25, 43, 47, 49, 52, 55, 56.
- Cycles One-Shot 0.6 (`cycles-sampler/`): `README.md`, `TECHNICAL.md` §1–§9, §14, `machine/os.h`,
  `machine/sampler.c`, `machine/pool.c`, `machine/pool.h`, `machine/hooks.S`, `machine/build.py`,
  `build/sampler-tweak.json`.
- The Track Setup menu and Control All facts: the loop/reverse study of 2026-10-19 (read on the OS 1.13, not built).
- Verified in this session: One-Shot's build against Modded-Cycles `6a9742a` and `95b63b7` gives the same image
  (SHA-256 `7d9a73bf…`); `tools/build.py -t sampler` with `sampler-tweak.json` as a plain tweak file gives the same
  image; `check_overlaps.py` with the sampler added reports 95 undeclared partners, all machine-adding tweaks newer
  than `6a9742a`, and no other.
- For 0.2, [model27](https://github.com/durchgemacht/model27) at `a279f3f`: `tools/compose.py`, `tools/relocate.py`,
  `tools/gen_registrar.py`, `tools/machines/registrar/registrar.S` and `registrar.json`, `tools/machines/sampler/`
  (manifest, `sampler.o`), `tools/machines/refmachine/`, `tweaks/model-cycles_OS1.13/27-sampler.json`; notes 59, 60
  and 61. Cycles One-Shot 0.7 and 0.8 (`f28ef2f`): the pool's range and the patch points (`pool.h`, `view.c`).
- For 0.3, model27 after step B4e (2026-10-22):
  - `tools/compose.py` (`shortcut_layer`, `dispatch_layer`, `web_part`) and `docs/flasher/composer.js`;
  - `tools/machines/shortcuts/`, `tools/machines/dispatch/` and `tools/machines/refmachine2/`;
  - `tools/emu/test_shortcuts.py` and `test_views.py`;
  - notes 62, 63 and 64.
- For 0.4, model27 after step C4 (2026-10-23):
  - `tools/claims.py`, `tweaks/model-cycles_OS1.13/claims/claims.json` and the claims of the manifests and of the
    shortcut part;
  - `tools/compose.py` and `docs/flasher/composer.js` (the claims' rules), `tools/check_overlaps.py`,
    `tools/registry.py` and `REGISTRY.md` §6 and §8 to §10;
  - `tools/machines/sampler/machine/view.c` (the formatter of object 53);
  - notes 65, 34, 42, 53, 55 and 58.
- For 0.5, model27 after notes/63 §18 (2026-10-26, commits `c9eac8b`, `514b6dc`, `99d80b9`):
  - `tools/machines/shortcuts/` 0.2 (`shortcuts.S`, `part.json`), `tools/compose.py` (`shortcut_presses`,
    `shortcut_layer`) and `docs/flasher/composer.js`, `tools/claims.py`;
  - `tools/machines/refmachine2/` (the switches, the marks);
  - `tools/emu/test_shortcuts.py`, `test_views.py` and `test_refchord.py`, and the maintainer's run of them;
  - the OS 1.13: `0x4001ac4a` and its two callers, MainScreenView's LED method `0x4001cbfe`, the smoother `0x40058474`
    and its caller `0x4005979e`, the save and load tables `0x4010ee4c` and `0x4010eea8`;
  - notes 63 §15 to §18, 66 §2.1 and 47 §4.
