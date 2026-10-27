# Applying the standard to your mod — a checklist

**Status:** draft 0.6, 2026-10-27, companion to [STANDARD.md](STANDARD.md). Written for the author of an existing
Model:Cycles mod and the AI assistant they work with. Sections 1 to 11 inventory a mod; section 12 ports a machine to
model27's builder, step by step, with REF2 as the template; section 13 ports an effect, with Digital Delay as the
template.
**Authors:** durchgemacht and Claude. **License:** MIT ([LICENSE](LICENSE)).

**What sections 1 to 11 produce:** an *inventory* of what the mod touches, a *manifest*, a list of *collisions* with
other mods and the registry, and a list of *deviations* from the machine contract. **They do not produce a rewrite.**
Much of the target exists:
- [model27](https://github.com/durchgemacht/model27)'s composer reads manifests and registers added machines
  (STANDARD.md §8);
- it adds the machine shortcuts and the dispatchers for the points SAMPLE uses (STANDARD.md §3.3);
- it plays effects from packages in the mixer's delay and reverb slots, chosen per pattern (STANDARD.md §3.5);
- its web flasher does the same in the browser;
- its tools compute the clashes over saved slots, key combinations, descriptors, formatters, MIDI and memory from
  what each mod declares, its claims, and its registry lists them (STANDARD.md §4.4, §4.5).

There is still no shared header, and no number file: in model27 the claims stand in for one (STANDARD.md §5). Do not
let anyone, human or AI, invent those for your mod alone: three mods would get three different inventions, which is
the problem the standard is meant to end. The reference implementation is model27's (STANDARD.md §8, §9). To move a
machine onto it, follow section 12; an effect, section 13.

- [0. What you need](#0-what-you-need)
- [1. Describe the mod in five lines](#1-describe-the-mod-in-five-lines)
- [2. Inventory: what the mod touches](#2-inventory-what-the-mod-touches)
- [3. If it is a machine: the contract check](#3-if-it-is-a-machine-the-contract-check)
- [4. Map every touch to an extension point](#4-map-every-touch-to-an-extension-point)
- [5. Registry check](#5-registry-check)
- [6. Cost](#6-cost)
- [7. Write the manifest](#7-write-the-manifest)
- [8. Compute "works with" by hand](#8-compute-works-with-by-hand)
- [9. What to send back](#9-what-to-send-back)
- [10. A prompt for your AI assistant](#10-a-prompt-for-your-ai-assistant)
- [11. What not to do yet](#11-what-not-to-do-yet)
- [12. Porting a machine to model27's builder](#12-porting-a-machine-to-model27s-builder)
- [13. Porting an effect to model27's builder](#13-porting-an-effect-to-model27s-builder)

## 0. What you need

- the mod's sources and its tweak JSON, or the generator that writes it (`tools/gen_<mod>.py` in Modded-Cycles);
- a checkout of [Modded-Cycles](https://github.com/18nelli18/Modded-Cycles) at a release with the mod picker
  (1.56, commit `95b63b7`, or earlier): its `tools/`, its `notes/` and `tweaks/model-cycles_OS1.13/REGISTRY.md`;
- for a machine, [Cycles One-Shot](https://github.com/durchgemacht/cycles-one-shot)'s `TECHNICAL.md` §5 and §6, which
  describe the voice loop and the sound in the OS's terms;
- for a machine, model27's packages as worked examples, `tools/machines/sampler/`, `tools/machines/refmachine/`
  (the smallest machine) and `tools/machines/refmachine2/` (a shortcut, both views, the tick and the browser; manifest
  in STANDARD.md §7.4), each with its manifest, sources and committed object, and its notes 59, 60 and 63;
- [STANDARD.md](STANDARD.md), sections 2 to 6;
- your own official `model-cycles_OS1.13.syx`, if a step needs a build or an emulation run. Nothing in this
  checklist asks you to share it.

Everything below is to be read from the code and the tweak, not from memory. Where a step says "verify", the
answer is a file and a line, or "not verified".

## 1. Describe the mod in five lines

Write down, in plain words:

1. what it does, as a musician would say it;
2. on which tracks: all of them, or the tracks of one machine, or a kit-level thing;
3. which knobs, buttons or key combinations it takes, and what those did in the stock OS;
4. what it stores: in the sound, the kit, the pattern, the project, nowhere;
5. what it costs, if you have ever measured it; otherwise "unknown".

These five lines become `description`, the `scope` fields and `stock_behaviour.changes` of the manifest.

## 2. Inventory: what the mod touches

Derive it from the tweak JSON and the tools, not from what you remember. Addresses are virtual addresses of the
MAIN OS; the JSON's `off` is the offset from `0x40000400`.

### 2.1 Hooks

List every write whose `new` bytes start with `4eb9` (`jsr`), `4ef9` (`jmp`), `61ff` (`bsr.l`) or `60ff`
(`bra.l`). For each: the address, the stock bytes, and what the stock code did there, read from the OS or from the
mod's own note. A few lines of Python do the listing:

```python
import json
BASE = 0x40000400
t = json.load(open("tweaks/model-cycles_OS1.13/<nn>-<mod>.json"))
for w in t["writes"]:
    if w["new"].startswith(("4eb9", "4ef9", "61ff", "60ff")):
        print(hex(BASE + w["off"]), "stock", w["old"], "->", w["new"])
```

### 2.2 Rewritten pointers

Four-byte writes whose `old` bytes are themselves an address in the OS (`40…`): vtable entries, references to a
table the mod moved. `REGISTRY.md` §5 lists them per mod once `registry.py` has run.

### 2.3 Masks, caves and payload

Run `python3 tools/registry.py` in the Modded-Cycles checkout and take the rows of `REGISTRY.md` §1 to §3 that
name your mod: which sprite masks it writes, which cave bytes, and, if the tweak has an `append`, its `dest`, its
`size` and the bytes it adds to the image.

### 2.4 RAM outside the payload

Search the sources for absolute addresses (`0x4…`, `0x8…`, `0xfc…`). Every one that is not in the mod's own payload
and not in the shared list of STANDARD.md §2 is a claim: the OS's data it reads or writes, or free RAM it takes.
One-Shot's sample pool, `0x43800000`–`0x46700000` since 0.7, is the example of the second kind.

### 2.5 Sound slots and record slots

- A sound's parameter k lives at `+20 + 2·k`; k 23 to 32 are at `+66` to `+84`. Search the sources for those offsets
  and for direct reads of the audio side's copy at `0x800015ae + 66·t`.
- A record's slot s lives at `+28 + 2·s`; slot 24 is `+76`. Search for those offsets and for hooks on the three record
  functions `0x4005afa0`, `0x4005b054`, `0x4005aecc`.
- Write down every k and every slot the mod reads or writes, and whether it keeps them on every sound or only on
  its own machine's.

### 2.6 Descriptors and formatters

Which parameter descriptors the mod adds or re-uses (the OS's 1 to 4, or 76 + 5·n for an added machine), and
whether it replaces a knob's formatter (the `+0x20` of a parameter object, STANDARD.md §2.5).

### 2.7 Gestures, menus and views

- key combinations the mod reads, written as STANDARD.md §4.5 writes them (the held keys, then one action), where
  they act (the parameter pages, the grid, the keyboard, a page, a menu), and the stock meaning they replace or
  extend;
- menus it appends to (the Track Setup caller `0x4001c510`, STANDARD.md §3.1);
- views it draws on: the vtable entries `0x400ffe30` (main screen) and `0x4010053c` (knob view), the redraw tick
  `0x400075f0`, or other draw hooks (level-pan-values uses `0x400081f2` and `0x4001aa4e`).

### 2.8 MIDI

CC and NRPN numbers the mod listens to or sends.

### 2.9 The governor

For a machine: whether `update` and `render` tolerate not being called for a quiet voice (`GOV_SKIP`,
STANDARD.md §2.4). For a chain stage: whether it runs even when the governor mutes voices.

## 3. If it is a machine: the contract check

Go through STANDARD.md §2.6 and answer each line with a file and a line number, or "deviates" with a reason:

| Rule | How to check |
|---|---|
| `update(pmod, voice, params)` then `render(out, voice)`, from the `update`/`render` tables | the generator's `MACHINE` entry and the two symbols |
| the pitch law: PITCH − 64 + note + FINE, clamped 0–127 | compare one note against TONE in the emulator; or read the computation |
| the stock chain in TONE's order, or a documented deviation | read the end of `render` against TONE's setup (`0x400aa7b8`); One-Shot's filter after the chain is the example of a documented deviation |
| nothing while idle | count instructions per block with six quiet voices (`tools/emu/mcengine.py`); the number should be near zero for your machine |
| survives the voice reset (`0x400a7ab8`) and a governor gap of three blocks | a marker at `+0x2c` of the voice, and a timer or block-count test; read it |
| state in its own BSS, code in its own payload, no OS detour | the inventory of section 2: an empty 2.1 apart from the added-machine mechanics, an empty 2.4 |
| other tracks bit for bit as stock | render a pattern with your machine on one track and the others stock, diff the other tracks against a stock render (`tools/emu`, the approach of `ref_mainos.py`) |
| "off" positions bit for bit | the same diff for your own track with every knob at its off or neutral position, if the machine has one |
| cost declared | section 6 |

Everything that is not an answer with a line number is a deviation to list in `stock_behaviour.changes`.

## 4. Map every touch to an extension point

Take the inventory of section 2 and, for each item, find the row of STANDARD.md §3.1 it belongs to:

| Inventory item | Extension point | Composition rule |
|---|---|---|
| the added-machine mechanics | `machine` | table |
| a hook on the Volume + Dist call `0x40059852` | `track.post_voldist` | chain, by `order` |
| hooks on the three record functions | `sound.record` | chain: one wrapper per function, behind model27's shortcut layer |
| a hook on the parameter-table builder `0x4004486c` | `param.tables` | append |
| a hook on the sound's slot copy `0x4005aa34` | `sound.copy` | chain |
| a hook on the browser's file read `0x400a6602` | `browser.open` | by machine (the track it loads into) |
| a hook on the redraw `0x400075f0` | `screen.tick` | append (every machine) |
| a rewritten draw entry `0x400ffe30` or `0x4010053c` | `view.main`, `view.knob` | by machine (the selected track) |
| RETRIG held + COLOR, SHAPE, SWEEP or CONTOUR on your machine's tracks | `shortcut` | by machine, from the builder's list |
| RETRIG held + a press on PUNCH or GATE on your machine's tracks (a switch) | `shortcut` | by machine, from the builder's list |
| a hook on the mixer's call of the delay `0x4005699c` or of the reverb `0x40056a2c` | `fx.delay`, `fx.reverb` | table by id: the effect slots (section 13) |
| a hook on the delay line's write DMA `0x40056a3c`, or the stock delay's line | none: an effect gets the slot's own line (STANDARD.md §3.5) | — |
| a key combination that picks the effect | `gesture`, owned by the effect slots (TRACK held + Delay Time / Reverb Size) | exclusive |
| an entry appended to Track Setup | `menu.track_setup` | append |
| a key combination | `gesture` | exclusive |
| sound slots, record slots | `param.sound` | registry |
| the OS's descriptors 1–4 | `param.descriptor` | exclusive |
| CC, NRPN | `midi` | registry |
| sprite masks, caves | `memory.mask` | allocation |
| `append` in the tweak | `memory.payload` | allocation |
| free RAM | `memory.ram` | allocation |

An item that fits no row is a **plain hook claim**: keep it in `claims.hooks` with its stock bytes and its reason,
and add one line on what the extension point *would* be. That line is how the table of §3.1 grows.

Outside model27 no point is owned by a builder, so your mod still places its own detours there; the mapping is a
declaration, not a code change. In model27 the points of STANDARD.md §3.3 are dispatched when two machines take them;
section 12 says what your methods must do for that.

## 5. Registry check

Compare the inventory with STANDARD.md §5 and write every collision down:

- **Machine id** (§5.1): the id is the position in the build (option C, no permanent ids). Record what your build
  assigns and in which combinations it changes.
- **Sound slots k** (§5.2): the table says who holds k 23 to 32; model27's `REGISTRY.md` §6 keeps the current map.
  **No k is free.** A collision with your slots is a
  conflict to declare; a need for a new persistent parameter is a question for the registry, not a slot to take.
- **Record slots** (§5.3): same check; slot 18 is unused by the OS and taken by the track-filter.
- **Gestures** (§5.4): the same combination with another meaning, in situations that meet, on the same tracks is
  an exclusive conflict; the same combination on different machines' tracks is not. model27's `REGISTRY.md` §8
  lists what is taken, and the stock combinations a mod may take only by saying what it replaces.
- **MIDI** (§5.5): collisions with other mods; the stock map is not collected yet.
- **Order** (§5.6): the same `order` as another package at the same chain point is a conflict; note today's number.

## 6. Cost

Measure, and write the method next to the number (STANDARD.md §6):

- **Emulation:** instructions per block with `tools/emu/mcengine.py`'s counter, in the real voice loop: the first
  block, then mean and maximum over the next 21 blocks. For a machine, per voice, with one track trigged and the
  other five on KICK at rest (the pattern of notes/23 §1 and notes/43 §8); for a chain stage, per block, with six
  tracks in the states that matter (the track-filter's table in notes/47 §9 is the model: all off, one on, six on,
  six modulated).
- **Device:** the share of a block from the `syntakt-meter` diagnostic firmware, if you can run it. The emulator does
  not time memory: the Syntakt engines counted like stock machines and cost nearly twice as much on the device.
  A device number beats an emulation number; an emulation number beats none.

Put the reference next to it: TONE is about 5 300 instructions per block per voice by the same count.

## 7. Write the manifest

Fill the skeleton below; STANDARD.md §4.2 explains each field, §7.1 is a complete manifest as model27's composer
reads it, §7.2 and §7.3 are two more in the older shape. Each hook goes into `uses`, with the original bytes it
replaces. What tools will derive later (rewritten pointers, masks, payload sizes) goes into `_derived_by_hand`,
filled by hand from sections 2 to 6, so that nobody mistakes it for tool output.

```json
{
  "schema": "cycles-mod/0.1",
  "id": "<tweak id, the package's folder name>",
  "version": "<release or git describe>",
  "name": "<as on the flasher card>",
  "description": "<the five lines of section 1, condensed>",
  "license": "<MIT | file name>",
  "credit": {"kind": "<own | based>", "who": "<name or handle>", "repo": "<owner/repo>"},
  "source": "<URL of the sources at the commit you read>",
  "device": "Model:Cycles", "os": "1.13",
  "order": 0,
  "build": {
    "sources": ["<file.c>", "<file.S>"],
    "include": [],
    "cflags": ["<your build's flags; a package adds -fno-pic -fno-common -ffunction-sections -fdata-sections>"],
    "defines": {},
    "objects": ["<id>.o"],
    "symbols": {"update": "<for a machine>", "render": "<for a machine>"},
    "keep": ["<every other symbol a hook or vtable entry points at>"],
    "patch_points": [{"symbol": "<undefined symbol>", "kind": "<machine_index | descriptor_base>"}]
  },
  "uses": [
    {"point": "<id from STANDARD.md §3.1>", "scope": "<all tracks | tracks of <machine>>",
     "hook": {"at": "0x........", "stock": "<old bytes>", "op": "<jsr | jmp>", "symbol": "<your function>"},
     "why": "<one line>"},
    {"point": "<id>", "vtable": {"at": "0x........", "stock_ptr": "0x........", "symbol": "<your method>"},
     "why": "<one line>"}
  ],
  "claims": {
    "ram": [],
    "sound_slots": [], "record_slots": [], "saved": [], "descriptors": [], "formatters": [], "plock_rows": false,
    "gestures": [{"combo": "<held keys> + <action>", "situation": "<any | params | grid | keys | page:<name> | menu:<name>>",
                  "meaning": "<what it does>", "replaces": "<only when it takes a stock combination: what it did>"}],
    "views": [], "midi": {"cc": [], "nrpn": []}
  },
  "_derived_by_hand": {
    "pointers": [],
    "masks": [],
    "payload": {"code_and_tables": 0, "bss": 0, "image_bytes": 0, "window": "<alone | with-tg | none>"}
  },
  "cost": {
    "emulation": {"tool": "mcengine.py", "pattern": "<named>", "mean": 0, "max": 0, "first_block": 0},
    "device": {"status": "<not measured | percent and how>"}
  },
  "stock_behaviour": {"changes": [], "fixes": [], "unchanged": []},
  "compat": {"built_against": ["modded-cycles <commit>"], "requires": [], "conflicts_declared": [{"with": "<id>", "why": "<text>"}]},
  "tests": {"emulation": {"tool": "<script>", "date": "<YYYY-MM-DD>"}, "hardware": {"date": "<YYYY-MM-DD>", "by": "<who>"}, "status": "<experimental | tested>"}
}
```

## 8. Compute "works with" by hand

Until a tool does it, apply the seven rules of STANDARD.md §4.4 against each other mod in the builds you care
about, starting with those in your own build:

1. same hook address, different bytes;
2. same exclusive point, different meaning;
3. same registry number;
4. overlapping masks, payload or RAM;
5. more machines than the layout holds (6 added for now, to be raised soon), or a cost sum that looks too high;
6. same `order` at the same chain point;
7. a reason only a human knows.

Compare the result with the tweak's declared `conflicts`. Keep every declared conflict that has a reason of kind 7
and write that reason into `conflicts_declared`; the others become computed once the tools exist. Do not remove a
declared conflict from the tweak JSON because of this exercise.

## 9. What to send back

- `manifest.json` (section 7);
- `inventory.md`: sections 1, 2, 4, 5 and 6 as written, with file and line references;
- the deviations of section 3, if the mod is a machine;
- open questions: a slot you need, a point that fits no row, a gesture that clashes.

Put them next to the mod (`tools/machines/<mod>/` in a Modded-Cycles checkout) or send them to the standard's
repository as an example. Do not change the mod's code for this.

## 10. A prompt for your AI assistant

Paste this, with the paths filled in:

```text
Read STANDARD.md and APPLYING.md from <path to the standard>. The mod to inventory is <path to sources> with its
tweak <path to JSON> in a Modded-Cycles checkout at <path>, release 1.56.

Produce, as files, nothing else:
1. inventory.md following APPLYING.md sections 1, 2, 4, 5 and 6, every statement with a file and line reference
   or the words "not verified";
2. manifest.json following APPLYING.md section 7, its claims in the shapes of STANDARD.md §4.5, the derived parts
   under "_derived_by_hand";
3. if the mod is a machine, deviations.md following section 3;
4. questions.md with anything that fits no extension point, needs a registry number, or could not be measured.

Do not change the mod's code. Do not invent a header, a registrar, a registry file or dispatcher tables (the
registrar and the dispatchers that exist are model27's). Do not assign a machine id or a sound slot. Where STANDARD.md
and the code disagree, say so and quote both.
```

## 11. What not to do yet

- do not rewrite the mod against a header or a registry that do not exist yet; packaging a machine for model27's
  composer is a separate step, with its own proofs (section 12, STANDARD.md §8);
- do not assign yourself a machine id, a k slot, a record slot or an order number;
- do not drop declared conflicts because the rules would compute them;
- do not report an emulator instruction count as a device percentage;
- do not ship a manifest as "derived" when it was filled by hand.

## 12. Porting a machine to model27's builder

This section is for a machine that already plays on the Model:Cycles (its own tweak, or a generator such as
`gen_macro.py`) and should become a *package*: an object model27's composer places and registers with other machines,
in Python and in the web flasher. REF2 (`tools/machines/refmachine2/`, STANDARD.md §7.4) does everything below; copy
its files, then replace its sound. Work in a model27 checkout; every step has a check, and a step is done when its
check passes.

### 12.1 The object

1. **Sources** in `tools/machines/<id>/`, C99 and assembly, MIT or a license that allows it (STANDARD.md §4.2
   `license`). No libc: no `memcpy`, no initialised local arrays (GCC may turn them into a `memcpy` call), no float
   if you can avoid it.
2. **Flags:** `-std=c99 -mcpu=54418 -O2 -ffreestanding -fno-builtin -fno-tree-loop-distribute-patterns -nostdlib
   -fno-pic -fno-common -ffunction-sections -fdata-sections -Wall -Wextra -Werror`, as REF2's manifest has them.
   They give an object with only `R_68K_32` relocations, which `tools/relocate.py` can place.
3. **No build-time constants for the build's numbers.** The machine's number and its first descriptor are patch
   points: an undefined symbol whose address is the value.

   ```c
   extern char ref2_machine_index[];                   /* kind machine_index */
   #define REF2_MACHINE ((int32_t)(long)ref2_machine_index)
   ```
4. **Variables** in your own BSS; `.data` and `.rodata` go with the code. A section aligned to more than 4 is
   refused.
5. **The object** is the `ld -r` of the sources, `tools/machines/<id>/<id>.o`, written and checked by a generator
   (`tools/gen_refmachine.py` is the model: `compose.build_object`, `compose.emit_bytes`, `--check`).

*Check:* `python3 tools/relocate.py tools/machines/<id>/<id>.o` lists your sections, `R_68K_32` only, and your patch
points as the only undefined symbols.

### 12.2 The voice

Follow STANDARD.md §2.6:
- `update(pmod, voice, params)` and `render(out, voice)`;
- the pitch law of §2.1;
- the stock chain at the end of `render`;
- nothing while idle;
- a marker at `voice + 0x2c` for the voice reset.

REF2's `refmachine2.c` is all of it in 150 lines.

### 12.3 The manifest

Start from REF2's (STANDARD.md §7.4). Then:
- `id` is your folder's name;
- `order` comes from the maintainer;
- `claims` say what the machine keeps and answers beyond its writes, in the shapes of STANDARD.md §4.5, on its own
  machine's tracks by default: a shortcut's k and record slot and its RETRIG combination in `params` (as REF2's),
  a switch's k 32, record slot 18 and its RETRIG combination in any situation (as REF2's), the views it draws on,
  and its RAM, the same ranges as its `memory.ram` entries;
- `build` names your sources, your object, your patch points, `update`/`render`, and in `keep` every function a hook
  or vtable entry points at;
- the `machine` entry gives four knobs (long name, short name of 4 characters at most, default, and `min`/`max` if not
  0–127), the picture of a stock machine (`image`, 0–5) and DECAY's default.

*Check:* `python3 tools/claims.py --check` finds the claims well formed and in line with the uses.
`python3 tools/compose.py --cycles <official file> --machines <id> --compare-ld` composes your machine alone,
identical to `ld`. Then compose it with `sampler` before and after it.

### 12.4 A shortcut or a switch (optional)

Your machine may give its tracks up to four extra values, one per knob, on **RETRIG held + COLOR, SHAPE, SWEEP or
CONTOUR**, and up to two switches, on **RETRIG held + a press on PUNCH or GATE** (below). No other combination is
accepted.

```json
{"point": "shortcut", "key": "RETRIG", "knob": "COLOR",
 "param": {"long": "Octave", "short": "OCT", "min": -2, "max": 2, "default": 0}}
```

- **0 is the neutral setting**, and `default` must be 0: a new sound, a sound saved before your machine had the
  shortcut, and a sound from the official OS all read 0. Choose the range around it.
- **Where to read it:** COLOR's value is in k 28, SHAPE's in k 29, SWEEP's in k 30 and CONTOUR's in k 31, at
  `params + 0x38 + 2 · i` in `update`, an 8.8 signed word. Clamp it to your range.
- **What you do not write:** the builder's shortcut layer shows the value, saves it with the sound and loads it back.
  Do not hook the record functions for it.
- **What it does not do:** the value is not p-locked and is not an LFO destination. Holding FUNC, TRACK, LFO or a trig
  turns the knob back to its own parameter.
- **It stays when a track changes machine**, and the next machine reads it its own way.
- **Optional `format`:** a kept function `int32_t f(void *object, int32_t value, char *buffer)` that writes at most
  three characters. Without it the value shows as an integer.

**A switch** (since draft 0.5) is on or off, saved with the sound:

```json
{"point": "shortcut", "key": "RETRIG", "press": "PUNCH"}
```

- **Off is 0**, the setting of a new sound, of a sound saved before, and of a sound from the official OS.
- **On your machine's tracks** a press on PUNCH (or GATE) with RETRIG held flips your switch, and PUNCH (or GATE)
  stays as it was. While RETRIG is held, PUNCH's and GATE's LEDs show your switches. Everywhere else, and with TRACK
  held (Control All), the press does what it does in the stock OS. A held trig changes nothing.
- **Where to read it:** bit 8 (PUNCH) or bit 9 (GATE) of k 32 in the sound the note plays, not in `params`, which the
  OS smooths (STANDARD.md §2.3). As REF2's `refmachine2.c` reads it:

  ```c
  #define CUR_SND(t)  (((const char *volatile *)0x800017ec)[t])
  #define TARGETS(t)  ((const char *)0x800015ae + 66 * (t))
  const char *snd = CUR_SND(t);
  uint16_t sw = snd ? *(const volatile int16_t *)(snd + 84) : *(const int16_t *)(TARGETS(t) + 64);
  ```

  Read in `update`, every block, it acts at once; read only in the block a note starts (`v + 0x38` set), it acts
  from the next note.
- **What you do not write:** the shortcut layer flips the switch, lights the LEDs and saves it. Showing it on the
  screen, if you want to, is yours: REF2 draws `S` and `O` left of its label.
- **It stays when a track changes machine**, like a value.

### 12.5 The points SAMPLE uses

Each one is a `uses` entry with the hook or vtable entry exactly as in REF2's manifest. The builder writes it
directly when your machine is the only one there, and goes through its dispatcher when another machine of the build
takes it too. **So write each method as if it were alone:** it must check that the track is yours and do the stock
work otherwise.

| Point | Your method | It must |
|---|---|---|
| `view.main` | `void main_draw(void *view, void *bitmap)` at `0x400ffe30` | draw only on the selected track of your machine and without a value window open (`0x4006b704(0x400cf9a8())` is 0); otherwise call the stock `0x4001b22a(view, bitmap)` |
| `view.knob` | `void knob_draw(void *view, void *bitmap)` at `0x4010053c` | draw only for your machine's parameter (`*(int32_t *)(view + 0x230)`, its descriptor), a value window open and no trig held (`0x4006b736(0x400cf9a8())` is 0); otherwise call the stock `0x4001e40a(view, bitmap)` |
| `screen.tick` | `uint32_t tick(void *view)`, `jsr` at `0x400075f0` | do its work quickly for every track, then return `0x40076be4(view)`; it may run from two tasks, so lock what you share; to redraw the main screen, call `0x40076082(main_view)` with the view your draw last got |
| `browser.open` | an assembly hook, `jsr` at `0x400a6602` | keep a0 (&result), a2 (the browser's pointer) and d2–d7; take the file only on a track of your machine (the browser object's `+664`); otherwise end with `jmp 0x400a3052` |
| `sound.record` | an "around" wrapper over the first 8 bytes of `0x4005afa0`, `0x4005b054`, `0x4005aecc` (One-Shot's `hooks.S`) | replay the 8 bytes, run the body, call your `after` function, keep `d0`, act only on sounds of your machine and only when the function returned 1. Only one machine may have one; use a shortcut where you can |

How to find the selected track's sound and machine is in REF2's `ref2_view.c`: `ref2_sound_of`, `selected_track`. Use
the same: the dispatchers find the machine the same way.

### 12.6 Proofs

Copy the tests and point them at your machine:
- `tools/emu/test_compose.py`, the registration and the voice loop;
- `tools/emu/test_shortcuts.py`, if you declared a shortcut or a switch;
- `tools/emu/test_views.py`, if you take a view, the tick or the browser.

They run the OS's own code in Unicorn. Then:
- `tools/webcompose_check.sh <official file>`: the web flasher composes your machine as the Python composer does;
- `claims.py --check`, `check_overlaps.py` (your claims against every mod's) and `registry.py --check`;
- the maintainer's device.

### 12.7 Not allowed

- a shortcut outside the list, two shortcuts on one knob, or a default other than 0;
- a switch outside the list, the same press twice, or a switch read at `params + 0x40` (it glides there);
- a sound slot or record slot of your own (k 23–32 and the record slots are taken: STANDARD.md §5.2, §5.3);
- a hook on the knob lookup `0x401005a8`, on `0x4001ac4a` or `0x4001cc86` (the shortcut layer's), or on the record
  functions next to another machine's wrapper;
- a method that draws, ticks or opens without checking the track: next to another machine it would run for that
  machine's tracks too;
- RAM outside your BSS other than a `memory.ram` claim that stays clear of the payload window.

### 12.8 A prompt for your AI assistant

```text
Read STANDARD.md (sections 2, 3, 4 and 7.4) and APPLYING.md section 12 from <path to the standard>, then model27's
tools/machines/refmachine2/ (manifest.json, refmachine2.c, ref2_view.c, ref2_open.S) in <path to model27>.

Port the machine in <path to its sources> to a package in model27's tools/machines/<id>/, following APPLYING.md
section 12 step by step: the object (12.1), the voice (12.2), the manifest (12.3), a shortcut or a switch only if the
machine needs an extra value or an on/off setting (12.4), and only the points of 12.5 the machine already uses, each
method checking its own
track; declare its claims as STANDARD.md §4.5 shapes them. Write the generator and the proofs by copying model27's
for REF2. Run every check named in section 12 and
report each result; where one fails, stop and say why.

Do not invent shortcuts, sound or record slots, points or dispatchers. Do not change model27's composer, its parts or
SAMPLE. Do not mark anything tested: the maintainer tests on the device.
```

## 13. Porting an effect to model27's builder

An effect replaces the mixer's delay or reverb while the playing pattern chooses it (STANDARD.md §3.5). It is a
package like a machine, smaller: one function per audio block, an optional reset, two parameters, an id. model27's
Digital Delay (`tools/machines/digidelay/`, STANDARD.md §7.5) is the template: a port of PPPCCC's digital delay whose
sound stays theirs bit for bit.

### 13.1 The object

Write the effect in C (or assembly) in `tools/machines/<id>/`, compile it with REF2's flags (`-mcpu=54418 -O2
-ffreestanding -fno-pic -fno-common -ffunction-sections -fdata-sections`), and commit the `ld -r` of the objects as
`<id>.o`. List the entry points in `build.roots`, the process first, then the reset. A generator in the shape of
`tools/gen_digidelay.py` writes the object, the tweak of the effect alone (the web flasher's card needs it) and the
license text, and checks them with `--check`.

### 13.2 The call

```c
void my_process(int32_t *out, const int32_t *in, const int16_t *params);
void my_reset(void);
```

- `in`, `out`: 32 stereo frames of signed 32 bits, L then R; write all 64 words of `out` at every call.
- `params`: the slot's two parameters, × 256: Time and Fdbk for the delay, Size (its byte) and Tone for the reverb.
  Their names on the screen are yours (`params` in the manifest), their knobs and p-locks stay the OS's.
- It runs in the audio interrupt: keep `d2`–`d7`/`a2`–`a6` (the C ABI does), use the EMAC freely, call nothing of the
  interface, never allocate, wait or lock.
- `my_reset` runs just before the first block after the effect is chosen, in that block: keep it short.
- A delay time that follows the tempo: the stock formula reads Time and the tempo at `0x40149310` (STANDARD.md
  §3.5); write it in your own code.

### 13.3 The line

If the effect needs a long buffer, do not take the stock delay's line: ask for the slot's.

```c
extern int32_t fx_line[];                   /* the slot's line: its address is your patch point of kind fx_line */
```

- `"line": <bytes>` in the slot entry, a multiple of 16 up to 4 MiB, and `{"symbol": "fx_line", "kind": "fx_line"}`
  in `build.patch_points`;
- the slot's other effects use the same RAM when they play: after your reset, read only what you wrote since (count
  the frames written, as Digital Delay's `n` does), or clear it in the reset if that fits in a block (32 KB costs a
  quarter of one);
- the CPU only, through the cache; zero at start-up.

### 13.4 The manifest

Copy Digital Delay's (STANDARD.md §7.5) and change:
- `id`, `version`, `name`, `description`, `license`, `credit`, `source`, `order`;
- in `uses`, the slot entry: `point`, `id` (ask the maintainer for one, STANDARD.md §5.8: 1–239 for effects users
  get), `process`, `reset`, `line`, `params` (two, in the slot's order, the long name of 15 characters at most and the
  short one of 5, `format` none or `"int"`), `name` (at most 15 characters);
- `claims`: usually none; `memory.ram` only for RAM beyond your variables and the line;
- `cost`: section 13.5;
- `tests`: what you ran, and `status` `experimental` until the maintainer's device.

### 13.5 Cost

Count your process with the proof's cost section (`test_fxslot.py`, section 6): instructions per block, the first
block, then the mean and maximum over 21 blocks after 32 more, the reset apart, and declare them in
`cost.emulation`. The builder compares the slot's dearest effect with the stock one: past a tenth of a block it warns,
past a quarter it refuses the build.

### 13.6 Proofs

- Add your effect to `tools/emu/test_fxslot.py`: a model of its arithmetic in Python, bit for bit (as `Digital`), and
  its selections (alone, with the other slot's effect, with SAMPLE); the proof runs the slot's sites, the audio
  through the playing pattern, the gesture, the pattern header and the cost on the OS's own code.
- When the effect comes from another firmware mod, compare it with that mod's own code too, as `--pppccc` does for
  Digital Delay: the original applied to the official OS, both run on the same blocks.
- `tools/webcompose_check.sh <official file>` with a selection of yours; `claims.py --check`, `check_overlaps.py`,
  `registry.py --check`; then the maintainer's device.

### 13.7 Not allowed

- a hook of your own on the mixer, the encoder handler or the pattern header: the effect slots take them;
- the stock effects' lines and DMA (`0x42340000`–`0x42800000`, `0x400573fa`, `0x40057942`);
- an id that is not yours, or one of 240–255 in a card;
- a reset that clears more than a block can afford, or reading the line before you wrote it;
- more than two parameters for now (reserved, STANDARD.md §3.5).

### 13.8 A prompt for your AI assistant

```text
Read STANDARD.md (sections 3.5, 5.8, 6 and 7.5) and APPLYING.md section 13 from <path to the standard>, then
model27's tools/machines/digidelay/ (manifest.json, digidelay.c), tools/gen_digidelay.py and tools/emu/test_fxslot.py
in <path to model27>.

Port the effect in <path to its sources> to a package in model27's tools/machines/<id>/, following APPLYING.md
section 13 step by step: the object (13.1), the call (13.2), the slot's line only if it needs a long buffer (13.3), the
manifest (13.4), its cost (13.5). Keep the effect's arithmetic as it is; where the original reads or writes the OS
outside the call's arguments, say so and stop. Write the generator and the proofs by copying model27's for Digital
Delay, with a Python model of the arithmetic, and run every check named in 13.6; report each result.

Do not invent ids, slots, parameters or hooks. Do not change model27's composer or its parts. Do not mark anything
tested: the maintainer tests on the device.
```
