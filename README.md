# Colo-XD-PBR-symbol-maps

Community symbol maps for Pokémon Colosseum, Pokémon XD: Gale of Darkness and
Pokémon Battle Revolution, in Dolphin's `.map` format. Drop one in Dolphin's
`Maps/` folder, named after the game ID, and its debugger will name functions
instead of showing bare addresses.

## Files

| File | Game | Symbols | Code symbols | Named | Data sections |
|---|---|---:|---:|---:|---|
| `GC6E01.map` | Colosseum (US) | 9,503 | 8,748 | 909 (10%) | `.rodata`, `.data` |
| `GC6P01.map` | Colosseum (PAL) | 8,820 | 8,820 | 872 (10%) | — |
| `GC6J01.map` | Colosseum (JP) | 8,798 | 8,798 | 870 (10%) | — |
| `GXXE01.map` | XD (US) | 14,562 | 13,337 | 12,781 (96%) | `.rodata`, `.data` |
| `GXXP01.map` | XD (PAL) | 14,275 | 13,364 | 12,726 (95%) | `.rodata`, `.data` |
| `GXXJ01.map` | XD (JP) | 14,697 | 13,275 | 12,986 (98%) | `.rodata`, `.data` |
| `RPBE01.map` | Battle Revolution (US) | 14,035 | 13,675 | 3,011 (22%) | all seven |
| `RPBP01.map` | Battle Revolution (PAL) | 14,312 | 13,906 | 3,219 (23%) | all seven |
| `RPBJ01.map` | Battle Revolution (JP) | 13,714 | 13,384 | 2,675 (20%) | all seven |

An unnamed symbol is written `zz_XXXXXXXX_`, carrying its own address. It is
still worth having: the boundary tells you where one function ends and the next
begins. A `zz_` name with words in it — `zz_script_function_95_warp_to_room`,
`zz_Script_interpreter_0x25_callstd` — is a hand annotation worked out by
reading the disassembly rather than taken from any linker map, and the prefix
is what marks it as such. There are 148 in the Colosseum US map, 133 in
Battle Revolution PAL, and a handful elsewhere.

Other files:

- **`NXXJ01.map`** — the development map of the Japanese XD demo (disc ID
  `NXXJ01`). The only genuine CodeWarrior linker output for either GameCube
  game, complete with object files and section layout, and the reference every
  other map here is checked against. In the verbose linker format, not
  Dolphin's.
- **`address based/`** — the raw Dolphin function-boundary analysis each map
  started from, before any name was attached. Kept as the input it is; every
  boundary in it is present in the matching map above.
- **`* official only.map`** — the main map filtered to the names `NXXJ01.map`
  carries, for when you want only what came out of a real linker and none of
  the matching or hand annotation. Derived, so regenerate it rather than edit
  it.
- **`* annotated.map`** — older working files from earlier passes, kept for
  history. **Do not use them.** They are stale (the XD one is missing 9,568 of
  the addresses the main map has) and they carry symbols from unrelated
  GameCube games — Colosseum's offers `Bowser::DrawShadow` and
  `cPlayer::PreUpdate`. Everything in them worth keeping is in the main maps:
  the names a reference build confirms, and the hand annotations, including
  the whole Colosseum script-interpreter opcode table.

## Where the names come from

- **XD**, all three regions: matched against `NXXJ01.map` above. Same codebase,
  so this works well — 95-98% of functions are named.
- **Colosseum**: a different program that shares Genius Sonority's engine and
  Nintendo's SDK with XD, so only the shared code can be named from the demo.
  Hence 9-10%.
- **Battle Revolution**: from [bgsamm/pbr-dtk](https://github.com/bgsamm/pbr-dtk)
  (CC0), a matching decompilation of the PAL disc — `config/RPBP01/symbols.txt`,
  21,628 symbols of which 3,506 carry a real name. The US and Japanese maps are
  matched from the PAL one.

## How a name is checked

A name is only kept where the code agrees with the build the name comes from.
Instruction streams are compared with register fields and immediates masked
out, so a recompile of unchanged source scores near 1.0 and an unrelated
function scores about 0.1. What decides a placement is the gap to the
runner-up rather than the absolute score, which is what stops a name landing
on one of the many near-identical accessor stubs.

Calibrated by porting one region's names onto a build whose own map already
carries them: Colosseum PAL → JP placed 513 names and all 513 landed on the
address the Japanese map already gave them, JP → PAL 516 of 516, and XD
US → PAL 6,126 of 6,147 — the 21 exceptions being a run of runtime symbols the
PAL map itself had shifted by one slot.

For a function too short to compare, position decides instead: the name is
kept only if it sits between two confirmed names in the same order as the
reference build's link.

## What was removed, and why

Earlier revisions of these maps carried `.bss`, `.sdata`, `.sbss`, `.sdata2`
and `.sbss2` sections — about 6,000 symbols per map. They were not of the
builds they claimed. The same list, with identical names, identical relative
offsets and identical sizes, appeared in the Colosseum US map, all three XD
maps and both Battle Revolution maps; and `.bss`, `.sbss` and `.sbss2` were
each rebased onto the same address, so they overlaid one another.

Two independent checks condemned them. Only 1-14% of the `.bss` symbol
addresses were ever built by an instruction, where the demo's own map scores
88%. And `.sdata`'s bytes disagree with the demo at every candidate base, so
no rebase rescues it — the list is the demo's layout, not any retail build's.

They are gone rather than corrected. The correct *locations* are easy — the
last two data sections of a DOL are the small-data pair the EABI addresses
through `r13` and `r2`, and the gaps after them inside the bss range are the
two small-bss sections — but the contents would have to be matched afresh, and
a wrong name is worse than no name.

`.rodata` and `.data` stayed: comparing only the words that are not pointers,
84-95% of their names agree with the demo, and the individual entries that
disagreed were dropped.

Battle Revolution keeps all seven data sections because they come from the
decompilation, not from this matching.

## How these maps are checked

The checks above are run by three scripts kept with the author's GoD-Tool
working copy rather than published here:

- `extract_dols.py` — pulls `main.dol` out of each GameCube disc, so every
  address claim is checked against a real section table rather than assumed.
  Battle Revolution's are Wii discs, so its three executables are supplied
  from a folder.
- `audit_symbol_maps.py` — reports anything that cannot be true: an address in
  no section of that executable, a symbol filed under the wrong section, a size
  running past the end of its section, two symbols at one address, an auto-label
  whose name disagrees with its own address, a function start that the Dolphin
  analysis in `address based/` found and the map dropped, a name on code that
  does not match the same name in the reference build, and a `.bss` section
  whose addresses the code never builds.
- `fix_symbol_maps.py` — applies the repairs, and is re-runnable.
- `make_official_only.py` — derives the `* official only.map` variants from
  the main maps.

All nine maps and both `official only` variants currently pass the audit with
nothing reported.
