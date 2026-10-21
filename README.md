# Cycles Mod Standard

**A draft, not published.** This page says what the idea is, first in plain words, then with a bit more technical
detail. The full text with every address, rule and source is in [STANDARD.md](STANDARD.md).

**Authors:** durchgemacht and Claude.

Everything here was read in the code of [Modded-Cycles](https://github.com/18nelli18/Modded-Cycles) (release 1.56,
the last one with a mod picker) and [Cycles One-Shot](https://github.com/durchgemacht/cycles-one-shot) 0.6, and in
the notes both projects keep about the Model:Cycles OS 1.13.

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
never moved, like MIDI CC numbers. The builder refuses two mods with the same number. For machine numbers there are
three ways to get there (permanent numbers with placeholders, a translation when sounds are saved and loaded, or
"projects belong to a build" as today); the draft recommends permanent numbers and leaves the decision open.

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
| `machine` | the machine tables | **table**, builder assigns the slot; 6 added machines fit in the current layout, 14 on the MACHINES page | MACRO, ACID, Syntakt engines, Model-TG's sampler, SAMPLE |
| `track.post_voldist` | the track chain after Volume + Dist, before pan, level and sends | **chain**, in declared order; a register contract to keep | track filter |
| `sound.record` | saving and loading a sound | **chain**: every mod's "after save" and "after load" run | Model-TG, track filter, One-Shot (today: three separate hooks, declared as conflicts) |
| `browser.open` | the PRESET browser opens a file | **chain**, first taker wins | One-Shot (sample files on a SAMPLE track) |
| `screen.tick` | the screen's redraw, about 30 times a second | **append** | One-Shot (loads samples in the background) |
| `view.main`, `view.knob` | the main screen and the knob value screen | **chain**, each draws then calls the next | One-Shot |
| `menu.track_setup` | the FUNC + TRACK menu | **append** | nobody yet (read for One-Shot's loop/reverse study) |
| `gesture` | a key combination and its meaning | **exclusive** per combination and scope | track filter, MACRO, Control All Config, stock |
| `param.sound`, record slots | the spare slots of a sound | **registry** | Model-TG, track filter, LFO-Plus, One-Shot |
| `param.descriptor` | the OS's four unused "Error" parameter descriptors | **exclusive** | track filter (makes them real parameters with p-locks, LFO, CC) |
| `memory.*` | freed sprite masks, the payload copied to RAM at boot, unused RAM | **allocation** by the builder | everyone with code |

Two places are marked *unread* in the draft: the chain directly after the voice loop, and the delay/reverb bus. A
third effect would need the second one read first.

## The manifest, trimmed

One JSON file per mod. The core says who the mod is; `uses` lists the extension points; `claims` are mostly derived
by tools from the compiled object; `cost`, `stock_behaviour` and `tests` are declared by the author.

```json
{
  "schema": "cycles-mod/0.1",
  "id": "sampler", "version": "0.6", "license": "MIT",
  "uses": [
    {"point": "machine", "image": 4, "decay_default": 127,
     "knobs": [{"long": "Start", "short": "STRT", "default": 0, "min": 0, "max": 120},
               {"long": "Length", "short": "LEN", "default": 120, "min": 0, "max": 120},
               {"long": "Cutoff", "short": "FREQ", "default": 64, "format": "own"},
               {"long": "Reso", "short": "RESO", "default": 0}]},
    {"point": "browser.open", "takes": "sample files on a SAMPLE track"},
    {"point": "sound.record", "after_save": "smp_after_save", "after_load": "smp_after_load"},
    {"point": "view.main", "scope": "SAMPLE tracks", "calls_next": true},
    {"point": "memory.ram", "range": ["0x42800000", "0x46700000"], "why": "sample pool, 63 MB"}
  ],
  "claims": {"sound_slots": [25, 26, 27], "record_slots": [24, 25, 26, 27]},
  "cost": {"device": {"status": "not measured"}},
  "stock_behaviour": {"changes": ["the filter runs before Volume + Dist, the Model:Samples has it after"]},
  "tests": {"hardware": {"date": "2026-10-20"}, "status": "experimental"}
}
```

From this the tools generate the tweak JSON, the flasher card, the registry entry and the computed compatibility.
The full examples for SAMPLE, MACRO and the track filter are in STANDARD.md, section 7.

## How "works with" gets computed

Two mods clash when any of these is true: the same hook address with different bytes; the same exclusive point with
a different meaning (a gesture, a formatter); the same number in the registry; overlapping memory; more machines than
the tables hold; the same order at the same chain point; or a declared conflict with a written reason, for the cases
tools cannot see. Everything else is compatible until a test proves otherwise.

Applied to today's mods: One-Shot's 77-entry conflict list shrinks to three registry clashes, one memory overlap
and one declared reason, and ACID, which the list does not know, is covered. The track filter's hidden clash with One-Shot on record slot 27
becomes visible.

## The cost model

The budget is one audio block: 32 frames, 667 microseconds, about 166 667 CPU cycles. Measured on the device by
Modded-Cycles: the mixer and effects take about 30 % of a block, each stock voice that sounds about 8 %, each Syntakt
voice about 15 %. Above roughly 85 % the interface starves; a governor mutes the quietest voice before that.

A mod declares its cost twice: instructions per block counted in the emulator (every author can do this), and the
share of a block read on a real device (the number that counts, because the emulator does not see cache effects; the
Syntakt engines counted like stock machines and cost almost twice as much on the box). The builder sums the worst
case of what is chosen and warns above a threshold.

## How a build would work

1. The builder reads the manifests of the chosen packages.
2. It assigns machine numbers from the registry, lays out the payloads, links the compiled objects at their addresses.
3. It fills the callback tables of the extension points. No code is generated per combination: tables are numbers.
4. It applies the writes with Modded-Cycles' existing build code and re-signs the file.
5. It writes the tweak JSON, the flasher card and a report: which machine got which number, where things sit, the
   cost sum.
6. The existing checks run unchanged: byte overlaps, every card alone and in pairs, the registry, each package's
   emulation proof.

The web flasher cannot run a linker. Either a small relocator in JavaScript for absolute 32-bit references, or each
machine shipped pre-linked once per possible slot (N machines × 8 slots instead of 2^N combinations). Open.

## Status and next steps

Nothing of this is implemented. What has been checked:

- One-Shot builds byte for byte the same against Modded-Cycles 1.56 as against the older commit it was written for.
- Its tweak JSON, dropped into the tweaks folder as a plain file, builds the identical image with Modded-Cycles'
  own `build.py`. The integration path exists; it was not documented.
- Modded-Cycles' overlap checker, with One-Shot added, reports 95 undeclared partners, all machine-adding tweaks that
  arrived after One-Shot's reference commit, and nothing else.
- All 71 addresses in the draft were checked against the source files they came from.

The path, in order: write the contract and the shared header; port One-Shot onto Modded-Cycles' existing
multi-machine registrar as the first proof; generate manifests for the existing mods from their generators; let the
registry tool read manifests; compute conflicts and compare with the hand lists; add the callback tables one point at
a time, each proven in emulation like a mod; last, the flasher.

Open decisions: how machine numbers become permanent; the cost threshold; the two unread places in the OS; the stock
MIDI map; relocator or pre-linked slots; who assigns registry numbers.

## Read more

- [STANDARD.md](STANDARD.md): the full draft with every address, rule and source.
- Modded-Cycles' notes 18 (adding a machine), 43 (MACRO), 47 (track filter), 49 and 52 (checking mods against each
  other, the register of free space), 23 and 25 (load measurements and the governor).
- Cycles One-Shot's TECHNICAL.md, sections 3 to 6 (the machine mechanics, the audio side, sounds and records).
