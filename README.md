# Cycles Mod Standard

**Draft 0.6, public.** This page says what the idea is, first in plain words, then with a bit more technical
detail. The full text with every address, rule and source is in [STANDARD.md](STANDARD.md). Much of it is built:
[model27](https://github.com/durchgemacht/model27) composes added machines from manifests, tested on a Model:Cycles,
and lets them share the screen, the browser and saved sounds and use a fixed list of shortcuts and switches, proven
in an emulator. Since draft 0.6 it composes effects too: a package that plays in the delay or the reverb slot, chosen
per pattern, tested on a Model:Cycles with the first one, Digital Delay.
Its tools also compute which mods clash over a saved slot, a key combination or a display, from what each mod
declares (see [Status and next steps](#status-and-next-steps)). To port a machine, follow [APPLYING.md](APPLYING.md)
§12; to port an effect, §13.

**Authors:** durchgemacht and Claude. **License:** MIT ([LICENSE](LICENSE)).

Everything here was read in the code of [Modded-Cycles](https://github.com/18nelli18/Modded-Cycles) (release 1.56,
the last one with a mod picker), [Cycles One-Shot](https://github.com/durchgemacht/cycles-one-shot) 0.6 to 0.8 and
model27, and in the notes the projects keep about the Model:Cycles OS 1.13.

## The problem, in one minute

The Model:Cycles mod scene has grown fast. Several people now add machines, effects and features to the OS, often
with AI assistants. The results are good, but four things are done again by every mod:

1. **Everyone re-derives the OS.** The addresses of the decimator, the amp envelope, the VCA and the decay table are
   defined four times in four files. The comment that explains how a machine talks to the OS is the same text in
   two projects.
2. **Every combination is a separate build.** A machine's slot, its parameter numbers and its place in memory are
   fixed when its patch is generated. That is why the tweaks folder holds 283 files, 252 of them combinations of
   Syntakt engines, and why SAMPLE can only ever be machine number 6.
3. **"Works with" is a hand-written list.** One-Shot's list does not know ACID, because ACID came later. A checker
   finds 95 undeclared clashes, all of the same kind. Nobody is to blame; lists go stale.
4. **Numbers in your projects depend on what you ticked.** A saved sound stores its machine as a number. MACRO is
   number 6 in one build and 7 in another. A project does not move between two builds.

## The idea, in one picture

```
 A  the contract          what every machine must do; the OS decides most of it already
 ─────────────────────────────────────────────────────────────────────────────────────
 B  extension points      named places in the OS, each with a rule for sharing it;
                          the builder assigns slots, numbers and memory, not the author
 ─────────────────────────────────────────────────────────────────────────────────────
 C  manifest + registry   one file per mod, from which the patch, the flasher card and
                          the "works with" are generated; permanent numbers for anything
                          that ends up in a saved project
```

None of this is a new loader or a new build system. It is the existing Modded-Cycles tooling with three things
written down and one thing moved: the decision *where a mod goes* moves from the author to the builder.

## What it would mean

**If you make music with the box:** you choose machines and features, up to the limit the OS tables allow. When two
things do not fit, the flasher says *why*: same gesture, same memory, same saved-data slot. The load each choice adds
is shown. Once machine numbers are permanent, a project made with one build loads on the next one.

**If you build a mod, with or without an AI:** you write against one header and one contract instead of hunting
addresses in someone else's notes. You never pick a slot or a memory address. You write one manifest; the patch, the
flasher card and the compatibility come out of it. If you use something beyond the plain machine interface (a key
combination, a screen, saved data) you say so in the manifest, and the tools check it against everyone else.

**If you maintain the OS:** compatibility is computed, not maintained. A registry says who owns which number.
New mods arrive as a package you can check mechanically before you test them on your own device.

## What does not change

- Builds still start from your own official Elektron file. No firmware bytes are shared.
- A mod is still a Modded-Cycles tweak at the end: the same patch format, the same byte checks, the same emulation
  proofs, the same web flasher.
- Every mod is still proven in emulation and tested on a real device before it is called tested.
- MIT, as before.

## "Four knobs" is not a dogma, it is the OS's layout

An added machine gets exactly five parameter slots from the OS: COLOR, SHAPE, SWEEP, CONTOUR and the amp decay.
DECAY, GATE and PUNCH come from the stock amp chain. So "four knobs" is not a rule anyone has to agree on. It is
what the box does. The standard only names what may be added on top, and asks that it be declared:

- a **core machine** uses the machine interface and nothing else: four knobs, pitch, the stock amp chain, silent when
  idle, no OS function touched;
- an **extension** is anything beyond that: MACRO's lo-fi on FUNC + PITCH, ACID using PUNCH as a waveform switch,
  One-Shot's waveform screen and sample browser, the track filter's gesture on MACHINES + knob. All fine. All
  declared, so that two mods do not claim the same thing without anyone noticing.

## The one hard limit: numbers that end up in your projects

Some numbers are not the builder's to choose, because they are already written into thousands of saved sounds:

- **the machine number** at byte 38 of every sound;
- **the ten spare parameter slots** of a sound, k 23 to 32: the OS uses 0 to 22, Model-TG took 23 to 27, One-Shot 25
  to 27, the track filter 28 to 31, LFO-Plus 28 to 32. **There is no free slot left.** Two of those pairs already
  exclude each other, one of them for this reason alone;
- **the spare slots in a saved sound record**, used by Model-TG, One-Shot, the track filter and LFO-Plus, with one
  overlap nobody has declared yet.

For these the standard proposes a **registry**: a small file in the repository where a number is assigned once and
never moved, like MIDI CC numbers. The builder refuses two mods with the same number. For machine numbers there were
three ways to get there (permanent numbers with placeholders, a translation when sounds are saved and loaded, or
"projects belong to a build" as today). Draft 0.3 takes the third. A track remembers a machine's place in the build,
so a project from a build with other machines plays other machines, a happy accident. Keep one project per build to
keep its sounds.

## Where upstream stands

The draft was read against Modded-Cycles 1.56, the last release where you ticked mods in a flasher. Since
2026-10-19 upstream is at **2.0: one OS instead of a mod picker**, with a *Mod Settings* page in the box (for example
2-channel or 6-channel USB audio, switched without reflashing). Its README says this exact firmware has not yet run on
a real Model:Cycles.

What that changes for the standard: levels A and B apply to a single OS just as well, because a single OS is one
build with every package in it. Level C's "tick a card" becomes "merge a package". The Mod Settings page is itself one
of the extension points (a menu that mods append to). And permanent machine numbers matter *more* in a single OS,
because every release must load every earlier project.

---

# A bit more technical

## The machine contract in ten lines

Every machine gives the OS two functions, called once per block of 32 frames at 48 kHz for each track that plays it:

```c
void update(int32_t pmod, char *voice, const char *params);   /* then */
void render(int32_t *out, char *voice);                        /* 32 frames, Q31 */
```

- `pmod` is the trig's note; `params` are the track's parameters as 8.8 words at fixed offsets (PITCH, COLOR, SHAPE,
  SWEEP, CONTOUR, PUNCH, GATE, FINE, DECAY).
- The voice is a fixed block of memory per track. Three flags in it say "a trig in this block", "a trig in the last
  block", "the note ended". The OS clears the voice when a track changes machine; a machine keeps a marker to notice.
- `render` ends with the stock chain: decimator (if the machine runs at 96 kHz), amp envelope, VCA, PUNCH, in that
  order. TONE's setup of the chain is the reference.
- The note a machine plays is PITCH − 64 + the trig's note + FINE, clamped to 0–127. Every stock machine does that.
- A machine does nothing while its voice is idle. Six idle tracks cost nothing.
- All of this is the OS's doing. The standard writes it down once, in a shared header, so that nobody copies it.

## Extension points

A place in the OS the standard owns: one hook, one table of callbacks, one rule for several mods at once. A mod never
detours such a place itself; it registers a callback in its manifest and the builder fills the table.

| Point | What it is | How mods share it | Who uses it today |
|---|---|---|---|
| `machine` | the machine tables | **table**, builder assigns the slot; 6 added machines fit in the current layout (to be raised soon), 14 on the MACHINES page | MACRO, ACID, Syntakt engines, Model-TG's sampler, SAMPLE |
| `track.post_voldist` | the track chain after Volume + Dist, before pan, level and sends | **chain**, in declared order; a register contract to keep | track filter |
| `sound.record` | saving and loading a sound | **chain**: one machine's "around" wrapper behind model27's shortcut layer | Model-TG, track filter, One-Shot (outside model27: three separate hooks, declared as conflicts) |
| `browser.open` | the PRESET browser opens a file | **by machine**: the machine of the track it loads into | One-Shot (sample files on a SAMPLE track), REF2 |
| `screen.tick` | the screen's redraw, about 30 times a second | **append**: every machine | One-Shot (loads samples in the background), REF2 |
| `view.main`, `view.knob` | the main screen and the knob value screen | **by machine**: the selected track's machine draws, or the stock screen | One-Shot, REF2 |
| `shortcut` | RETRIG held + COLOR, SHAPE, SWEEP or CONTOUR on a machine's tracks, and RETRIG held + a press on PUNCH or GATE (since 0.5) | **by machine**, from a fixed list: up to four extra values and two on/off switches per machine, saved with the sound; while RETRIG is held, PUNCH's and GATE's LEDs show the switches | REF2 (Octave; Square, Tone open) |
| `fx.delay`, `fx.reverb` | the mixer's delay and reverb | **table by id**: the stock effect, then the build's effects; the playing pattern chooses one, with TRACK held + Delay Time or Reverb Size; each slot has a line of its own for its effects | Digital Delay; model27's two reference effects |
| `menu.track_setup` | the FUNC + TRACK menu | **append** | nobody yet (read for One-Shot's loop/reverse study) |
| `gesture` | a key combination and its meaning | **exclusive** per combination, situation and scope; computed in model27 | track filter, Model-TG, MACRO, Control All Config, arp, and more (STANDARD.md §5.4); stock |
| `param.sound`, record slots | the spare slots of a sound | **registry**; computed from the claims in model27 | Model-TG, track filter, LFO-Plus, One-Shot, model27's shortcuts |
| `param.descriptor` | the OS's four unused "Error" parameter descriptors | **exclusive** | track filter (makes them real parameters with p-locks, LFO, CC) |
| `memory.*` | freed sprite masks, the payload copied to RAM at boot, unused RAM | **allocation** by the builder | everyone with code |

One place is marked *unread* in the draft: the chain directly after the voice loop. The delay and reverb, unread
until draft 0.5, are the effect slots since 0.6.

## The manifest, trimmed

One JSON file per mod. The core says who the mod is; `build` says how its one compiled object is made and which of
its symbols the builder needs; `uses` lists the extension points, each hook with the original bytes it replaces;
`claims` lists the saved-data slots, key combinations, displays and memory it takes (the shapes are in STANDARD.md
§4.5); `cost`, `stock_behaviour` and `tests` are declared by
the author. Trimmed from SAMPLE's manifest as model27 builds it:

```json
{
  "schema": "cycles-mod/0.1",
  "id": "sampler", "version": "0.8", "license": "MIT", "order": 27,
  "build": {
    "objects": ["sampler.o"],
    "symbols": {"update": "sampler_update", "render": "sampler_render"},
    "keep": ["smp_hook_open", "smp_hook_save2", "smp_hook_save1", "smp_hook_load", "smp_hook_tick", "smp_main_draw", "smp_knob_draw"],
    "patch_points": [{"symbol": "cm_machine_index", "kind": "machine_index"},
                     {"symbol": "cm_descriptor_base", "kind": "descriptor_base"}]
  },
  "uses": [
    {"point": "machine", "name": "SAMPLE", "image": 4, "decay_default": 127,
     "knobs": [{"long": "Start", "short": "STRT", "default": 0, "min": 0, "max": 120},
               {"long": "Length", "short": "LEN", "default": 120, "min": 0, "max": 120},
               {"long": "Cutoff", "short": "FREQ", "default": 64},
               {"long": "Reso", "short": "RESO", "default": 0}]},
    {"point": "browser.open", "hook": {"at": "0x400a6602", "stock": "4eb9400a3052", "op": "jsr", "symbol": "smp_hook_open"}},
    {"point": "view.main", "vtable": {"at": "0x400ffe30", "stock_ptr": "0x4001b22a", "symbol": "smp_main_draw"}},
    {"point": "memory.ram", "range": ["0x43800000", "0x46700000"], "why": "the sample pool, 47 MB"}
  ],
  "claims": {"sound_slots": [25, 26, 27], "record_slots": [24, 25, 26, 27]},
  "stock_behaviour": {"changes": ["the filter runs inside the machine, before Volume + Dist; the Model:Samples has it after"]},
  "tests": {"hardware": {"date": "2026-10-22", "by": "maintainer"}, "status": "tested"}
}
```

From this model27's composer builds the tweak: it places the object, gives SAMPLE its machine number and parameter
slots, registers it, writes its hooks and computes what it cannot be installed with. The flasher card and the
registry entry still come from that tweak. The full manifest is in STANDARD.md, section 7.1; MACRO and the track
filter follow there in the older shape.

## How "works with" gets computed

Two mods clash when any of these is true: the same hook address with different bytes; the same exclusive point with
a different meaning (a gesture, a formatter); the same number in the registry; overlapping memory; more machines than
the tables hold; the same order at the same chain point; or a declared conflict with a written reason, for the cases
tools cannot see. Everything else is compatible until a test proves otherwise.

model27's tools compute all of these but the cost, for machines and tweaks alike:
- each mod declares its claims;
- the composer, in Python and in the browser, adds the clashes to a build's conflicts;
- the overlap check refuses two mods that can be ticked together and clash.

SAMPLE's conflicts come out of its bytes and claims, 268 against model27's tweaks, and putting a second machine next
to it adds none. A clash nobody had seen in the bytes, LFO-Plus against model27's shortcuts over the same saved slots,
is now computed instead of written by hand. Applied to today's mods as a whole: One-Shot's former 77-entry list
shrinks to three registry clashes and one declared reason (the memory overlap is gone since One-Shot 0.7 moved its
pool), ACID is covered, and the track filter's hidden clash with One-Shot on record slot 27 becomes visible.

## The cost model

The budget is one audio block: 32 frames, 667 microseconds, about 166 667 CPU cycles. Measured on the device by
Modded-Cycles: the mixer and effects take about 30 % of a block, each stock voice that sounds about 8 %, each Syntakt
voice about 15 %. Above roughly 85 % the interface starves; a governor mutes the quietest voice before that.

A mod declares its cost twice: instructions per block counted in the emulator (every author can do this), and the
share of a block read on a real device (the number that counts, because the emulator does not see cache effects; the
Syntakt engines counted like stock machines and cost almost twice as much on the box). The builder sums the worst
case of what is chosen and warns above a threshold.

An effect declares its count per audio block, and only one effect plays in a slot at a time: the builder compares the
dearest effect of each slot with the stock one, warns above a tenth of a block and refuses a build above a quarter.
Digital Delay costs half a percent of a block more than the stock delay.

## How a build works

As built in model27 for added machines:

1. The composer reads the manifests of the chosen packages, in order.
2. It places each package's compiled object at its address without a linker: the objects only carry absolute
   addresses to fix up. Values that depend on the build (the machine number, the first parameter slot) are filled in
   the same way.
3. It registers all chosen machines at once through a runtime written once, by filling a small table. No code is
   generated per combination.
4. It writes each package's hooks, after checking the original bytes in your file, and computes what the result
   cannot be installed with.
5. Modded-Cycles' existing build code applies the result to your official file and re-signs it.
6. The existing checks run unchanged: byte overlaps, every card alone and in pairs, the registry, each package's
   emulation proof.

All of this runs with Python alone, and the web flasher does the same in the browser, byte for byte. Each object
comes placed once with its list of fix-ups. The registrar's part comes as a table per number of machines, and the
shortcut layer and the dispatchers as two more objects placed once (model27 notes/61 and 63; no pre-linked variants
per slot). Since draft 0.4 the registry's saved slots, key combinations, MIDI numbers and displays come from the
claims. Still to come: cards generated from manifests.

## Status and next steps

**Built** in [model27](https://github.com/durchgemacht/model27), 2026-10-21 to 26:
- **the `machine` extension point and the manifest**, for added machines. A composer places committed objects and
  registers them through a runtime, with Python alone, and the web flasher does the same in the browser. SAMPLE
  (Cycles One-Shot 0.8) and a small reference machine were the first two packages. This is proven on the OS's own
  code in an emulator and tested on a Model:Cycles; SAMPLE is offered, tested, in model27's web flasher;
- **the machine shortcuts, and the dispatchers** for saving sounds, the main and knob screens, the screen's tick and
  the browser. A second reference machine, REF2, uses all of them next to SAMPLE, and its manifest is the template
  for porting a machine. This is proven in the emulator and, since 2026-10-26, played on a Model:Cycles next to
  SAMPLE;
- **the claims:** what each mod takes beyond its bytes (saved slots, key combinations, descriptors, formatters, MIDI
  numbers, memory, displays), declared in one shape and checked. The tools compute the clashes from them, the
  registry lists them, and the web flasher explains a computed clash in English and French;
- **two switches** (2026-10-26): RETRIG + PUNCH and RETRIG + GATE, each on or off and saved with the sound, on the
  tracks of a machine that wants them; REF2 uses them for Square and Tone open. This is proven in the emulator and
  played on a Model:Cycles (2026-10-26);
- **the effect slots** (2026-10-27): the mixer's delay and reverb take effects from packages, chosen per pattern with
  TRACK held + Delay Time or Reverb Size, each slot with a line of its own and a counted cost. The first effect,
  Digital Delay, is PPPCCC's digital delay ported to a package, its sound theirs bit for bit; it is offered in
  model27's web flasher and played on a Model:Cycles from it, alone and next to SAMPLE.

The technical record is in model27's notes
[59](https://github.com/durchgemacht/model27/blob/main/notes/59-level-b-builder.md),
[60](https://github.com/durchgemacht/model27/blob/main/notes/60-level-b2-relocator-runtime.md),
[61](https://github.com/durchgemacht/model27/blob/main/notes/61-level-b3-web-flasher.md),
[63](https://github.com/durchgemacht/model27/blob/main/notes/63-level-b4-shortcuts-dispatchers.md),
[65](https://github.com/durchgemacht/model27/blob/main/notes/65-level-c-claims-check.md) and
[68](https://github.com/durchgemacht/model27/blob/main/notes/68-fx-slots.md).

**Checked for draft 0.1:**

- One-Shot builds byte for byte the same against Modded-Cycles 1.56 as against the older commit it was written for.
- Its tweak JSON, dropped into the tweaks folder as a plain file, builds the identical image with Modded-Cycles'
  own `build.py`. The integration path exists; it was not documented.
- Modded-Cycles' overlap checker, with One-Shot added, reports 95 undeclared partners, all machine-adding tweaks that
  arrived after One-Shot's reference commit, and nothing else.
- All 71 addresses in the draft were checked against the source files they came from.

The path, in order: write the contract and the shared header; port One-Shot onto a shared multi-machine registrar as
the first proof (done in model27); generate manifests for the existing mods from their generators (three written by
hand so far, and in model27 the other mods' claims in one declared file); let the registry tool read them (done in
model27); compute conflicts and compare with the hand lists (done in model27); add the callback tables one point at a
time, each proven in emulation like a mod (done in model27 for the five points SAMPLE uses); last, the flasher (done
in model27).

Open decisions: the cost threshold for machines; the unread place after the voice loop; the stock MIDI map; who
assigns registry numbers; for effects, more parameters, more slots, a MIDI choice and a guard at run time. Decided
in 0.2: the web flasher uses a relocator. Decided in 0.3: machine numbers stay the build's
("projects belong to a build"); shortcuts are a fixed list; the shared points go by machine. Decided in 0.4: the
tweaks' claims live in one declared file; key combinations are written on a fixed vocabulary and compared in
situations and scopes; taking a stock combination must say what it replaces; two displays on one screen are a
warning; the web flasher explains a computed clash. Decided in 0.5: two switches join the shortcuts, RETRIG + PUNCH
and RETRIG + GATE; their LEDs show them while RETRIG is held, and the rest of how a switch shows is the machine's.
Decided in 0.6: the delay and the reverb are two slots, one effect each at a time; the pattern keeps the choice; each
slot has its own line; effect ids are fixed; an effect's cost is checked per slot.

## Read more

- [STANDARD.md](STANDARD.md): the full draft with every address, rule and source.
- Modded-Cycles' notes 18 (adding a machine), 43 (MACRO), 47 (track filter), 49 and 52 (checking mods against each
  other, the register of free space), 23 and 25 (load measurements and the governor).
- Cycles One-Shot's TECHNICAL.md, sections 3 to 6 (the machine mechanics, the audio side, sounds and records).
- model27's notes 59 to 65: the builder as built, its proofs, the web flasher, the shortcuts and dispatchers, what
  users should know about projects and machines, and the claims; note 68, the effect slots.
- [APPLYING.md](APPLYING.md): inventory a mod (sections 1 to 11), port a machine to model27's builder (section 12),
  port an effect (section 13).
