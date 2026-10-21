# Applying the standard to your mod — a checklist

**Status:** draft 0.1, 2026-10-21, companion to [STANDARD.md](STANDARD.md). Written for the author of an existing
Model:Cycles mod and the AI assistant they work with.
**Authors:** durchgemacht and Claude.

**What this produces:** an *inventory* of what the mod touches, a *manifest*, a list of *collisions* with other
mods and the registry, and a list of *deviations* from the machine contract. **It does not produce a rewrite.** The
standard describes a target that does not exist yet: there is no shared header, no registrar that reads manifests,
no registry file, no dispatcher tables. Do not let anyone, human or AI, invent those for your mod alone; three
mods would get three different inventions, which is the problem the standard is meant to end. The reference
implementation comes from the standard's own repository first (STANDARD.md §9).

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

## 0. What you need

- the mod's sources and its tweak JSON, or the generator that writes it (`tools/gen_<mod>.py` in Modded-Cycles);
- a checkout of [Modded-Cycles](https://github.com/18nelli18/Modded-Cycles) at a release with the mod picker
  (1.56, commit `95b63b7`, or earlier): its `tools/`, its `notes/` and `tweaks/model-cycles_OS1.13/REGISTRY.md`;
- for a machine, [Cycles One-Shot](https://github.com/durchgemacht/cycles-one-shot)'s `TECHNICAL.md` §5 and §6, which
  describe the voice loop and the sound in the OS's terms;
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
One-Shot's sample pool, `0x42800000`–`0x46700000`, is the example of the second kind.

### 2.5 Sound slots and record slots

- A sound's parameter k lives at `+20 + 2·k`; k 23 to 32 are at `+66` to `+84`. Search the sources for those offsets
  and for direct reads of the audio side's copy at `0x800015ae + 66·t`.
- A record's slot s lives at `+28 + 2·s`; slot 24 is `+76`. Search for those offsets and for hooks on the three record
  functions `0x4005afa0`, `0x4005b054`, `0x4005aecc`.
- Write down every k and every slot the mod reads or writes.

### 2.6 Descriptors and formatters

Which parameter descriptors the mod adds or re-uses (the OS's 1 to 4, or 76 + 5·n for an added machine), and
whether it replaces a knob's formatter (the `+0x20` of a parameter object, STANDARD.md §2.5).

### 2.7 Gestures, menus and views

- key combinations the mod reads, and the stock meaning they replace or extend;
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
| hooks on the three record functions | `sound.record` | chain |
| a hook on the parameter-table builder `0x4004486c` | `param.tables` | append |
| a hook on the sound's slot copy `0x4005aa34` | `sound.copy` | chain |
| a hook on the browser's file read `0x400a6602` | `browser.open` | chain, first taker |
| a hook on the redraw `0x400075f0` | `screen.tick` | append |
| a rewritten draw entry `0x400ffe30` or `0x4010053c` | `view.main`, `view.knob` | chain |
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

Today no point is owned by the standard, so your mod still places its own detours; the mapping is a declaration,
not a code change.

## 5. Registry check

Compare the inventory with STANDARD.md §5 and write every collision down:

- **Machine id** (§5.1): today the id is the position in the build. Record what your build assigns and in which
  combinations it changes. Do not pick a permanent id yourself; the registry assigns it on merge.
- **Sound slots k** (§5.2): the table says who holds k 23 to 32. **No k is free.** A collision with your slots is a
  conflict to declare; a need for a new persistent parameter is a question for the registry, not a slot to take.
- **Record slots** (§5.3): same check; slot 18 is unused by the OS and taken by the track-filter.
- **Gestures** (§5.4): the same combination with another meaning on the same tracks is an exclusive conflict; the
  same combination on different machines' tracks is not.
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

Fill the skeleton below; STANDARD.md §4.2 explains each field, §7 has three complete examples. Fields marked
*derived* are what tools will compute later; fill them by hand for now from sections 2 to 6, and mark them
`"_derived_by_hand": true` so that nobody mistakes them for tool output.

```json
{
  "schema": "cycles-mod/0.1",
  "id": "<tweak id>",
  "version": "<release or git describe>",
  "name": "<as on the flasher card>",
  "description": "<the five lines of section 1, condensed>",
  "license": "<MIT | file name>",
  "credit": {"kind": "<own | based>", "who": "<name or handle>", "repo": "<owner/repo>"},
  "device": "Model:Cycles", "os": "1.13",
  "order": 0,
  "build": {"objects": ["<file.o>"], "symbols": {}, "patch_points": []},
  "uses": [
    {"point": "<id from STANDARD.md §3.1>", "scope": "<all tracks | tracks of <machine>>", "...": "the point's own fields"}
  ],
  "claims": {
    "_derived_by_hand": true,
    "hooks": [{"at": "0x........", "stock": "<old bytes>", "point": "<point id or plain>", "why": "<one line>"}],
    "pointers": [],
    "masks": [],
    "payload": {"code_and_tables": 0, "bss": 0, "image_bytes": 0, "window": "<alone | with-tg | none>"},
    "ram": [],
    "sound_slots": [], "record_slots": [], "descriptors": [], "formatters": [],
    "gestures": [], "views": [], "menus": [], "midi": {"cc": [], "nrpn": []}
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
5. more machines than the layout holds (6 added), or a cost sum that looks too high;
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
2. manifest.json following APPLYING.md section 7, with "_derived_by_hand": true;
3. if the mod is a machine, deviations.md following section 3;
4. questions.md with anything that fits no extension point, needs a registry number, or could not be measured.

Do not change the mod's code. Do not invent a header, a registrar, a registry file or dispatcher tables. Do not
assign a machine id or a sound slot. Where STANDARD.md and the code disagree, say so and quote both.
```

## 11. What not to do yet

- do not rewrite the mod against a header or a registrar that does not exist;
- do not assign yourself a machine id, a k slot, a record slot or an order number;
- do not drop declared conflicts because the rules would compute them;
- do not report an emulator instruction count as a device percentage;
- do not ship a manifest as "derived" when it was filled by hand.
