# Scylla ZMK

Instructions for agents working in this repo. The reply-format rules come first;
the rest is project knowledge from setting the keyboard up.

## Reply format

- Lead with the deliverable or the answer. Nothing before it.
- One question per reply, at most. Say what you recommend.
- Flags only if they block the work or change a decision. Max 3, one line each.
  Everything else goes into the files, not the reply.
- Report file edits in one line ("Updated X and Y"), never as a list.
- Step-by-step work (a group, a page, a sheet): show only that step. No
  look-ahead and no recap of earlier steps.
- Folder and page names go in code blocks, never in markdown lists (numbered
  names render as a. b. c.).
- If more detail might help, offer it in one line instead of including it.

## Project

- Scylla mk2: a wireless split keyboard with a nice!nano v2 in each half. No
  OLED, encoder or trackpad is fitted.
- Fork of the Tupinikeebs keyboard builder. `origin` is the user's fork,
  `KoliaK/scylla_zmk`. No upstream remote is set up; never push upstream.
- It uses the Lily58 shield, so the keymap is `config/lily58.keymap` (not
  `scylla.keymap`) and the build targets are `lily58_left` and `lily58_right`.
- ZMK is pinned to v0.3 in `config/west.yml`, `.github/workflows/build.yml` and
  `build.yaml`. Moving it means changing all three, plus the board name
  (`nice_nano_v2` on v0.3).
- The left half is the central: it talks to the PC over USB and Bluetooth. The
  right half is the peripheral and only talks to the left half.
  `config/lily58_left.conf` sets the Bluetooth name `Scylla`;
  `config/lily58.conf` sets deep sleep after 15 minutes and Bluetooth tuning.
- The physical key positions exist twice and must stay in sync:
  `config/info.json` (Keymap Editor) and the `zmk,physical-layout` block in
  `config/lily58.keymap` (ZMK Studio).
- The README is upstream's, in Portuguese. Leave it alone unless asked.
- The user's design notes are in `reference_files/`. That folder,
  `firmware_bkp/` and `screeshots_for_debugging/` (sic) are the user's and
  untracked: never commit or delete them.

## Keymap

- Layers: Base, SYM (hold position `42`), STENO (toggled by a combo) and NAV
  (hold position `55`). There is no system layer and no gaming layer, by choice.
  STENO sits below NAV on purpose, so a held NAV still shows its arrows.
- Thumb positions, left: `50` Alt, `53` Space, `42` SYM, `52` Tab, `51` F6/Snip.
  Right: `57` Del, `54` Bspc, `43` GUI, `56` F2, `55` NAV. Everything else
  follows the Lily58 order, 12 keys per row for the first three rows. The full
  map is in a comment above the `keymap` node.
- SYM follows `reference_files/2-coding_layer.md`. Cells it leaves empty are
  `&trans`, so they type the base key.
- NAV has IJKL arrows, Home and End on U and O, PgUp and PgDn on P and `;`, and
  F1-F12 on the number row (F11 on `-`, F12 on `'`). Esc stays Esc.
- F6/Snip is `&mt LS(LG(S)) F6`: tap for F6, hold for Win+Shift+S (Snipping
  Tool).
- Bluetooth clear is a combo: `~`, `\|` and F2 pressed together (positions `12`,
  `49`, `56`). It clears only the current profile's pairing with the PC. It does
  not touch USB or the link between the halves.
- STENO is for Plover in keyboard mode: Esc and `-` pressed together (positions
  `0` and `11`) toggle it, from any layer. Each key sends the QWERTY key
  Plover's default map expects, so Plover needs no key setup. Q-row is the
  number bar, A-row and Z-row are the steno rows, thumbs are the vowels
  (A `53`, O `42`, E `43`, U `54`), `-D` and `-Z` are on Enter and `\|`. Alt,
  Tab and NAV pass through. It needs `CONFIG_ZMK_HID_REPORT_TYPE_NKRO=y` in
  `config/lily58.conf` (may not work in some BIOS/UEFI). Plover has its own
  on/off switch and must be on exactly while the layer is on; there is no key
  for that, so it is switched by hand (tray icon, or the `{PLOVER:TOGGLE}`
  stroke).
- Every layer must list exactly 58 bindings. After editing, run the check below;
  each combo's single binding shows up as a `1` and is expected.

```python
import re
s = open("config/lily58.keymap", encoding="utf-8").read()
s = re.sub(r"/\*.*?\*/", "", s, flags=re.S)
s = re.sub(r"//[^\n]*", "", s)
for b in re.findall(r"(?<![-\w])bindings\s*=\s*<(.*?)>;", s, flags=re.S):
    print(len(re.findall(r"&\w+", b)))
```

## Build and flash

- GitHub Actions builds on every push and is the only compile check: there is no
  local ZMK toolchain and no `gh` CLI. A docs-only push also rebuilds, with the
  same firmware.
- Poll the build with the public API (the run appears after a minute or so; wait
  for `completed` and check `conclusion`), then give the user the `html_url` of
  the run.

```bash
api=https://api.github.com/repos/KoliaK/scylla_zmk/actions/runs
curl -s "$api?head_sha=$(git rev-parse HEAD)&per_page=1"
```

The user downloads the `firmware` artifact from the bottom of the run page
(logged in to GitHub; artifacts are hidden otherwise), unzips it, and flashes:

1. Plug in only one half over USB, and flash one half at a time.
2. Double-tap its reset button (inner side), about half a second apart. Too fast
   does not register and can take several tries. The `NICENANO` drive appears.
3. Drag the `.uf2` onto the drive: `scylla_left.uf2` on the left half,
   `scylla_right.uf2` on the right. The drive vanishes by itself; a Windows copy
   error is normal. Do not press reset again afterwards.
4. Test by typing. A single reset press only restarts the half.

## Pairing and resets

- Reset buttons never clear pairing. The Bluetooth-clear combo clears the PC
  pairing only.
- If the user deletes the keyboard from Windows' Bluetooth list, the keyboard
  still remembers the old pairing and will not show up as new until the combo is
  pressed. A board with no saved pairing advertises openly.
- `settings_reset.uf2` must go on both halves (each stores its side of the
  link), then `scylla_left.uf2` and `scylla_right.uf2` again, then pair with
  Windows again.

## Decisions already made

- NAV is held, not toggled: the user kept forgetting to switch the toggle off.
  If holding on the right thumb feels awkward, the fallback is hold to use,
  quick tap to lock.
- The SYSTEM layer (Bluetooth profiles, USB/BLE toggle, Studio unlock) was
  removed: holding SYM and NAV together while editing brackets could reach it by
  accident. If it is ever needed again, put it behind a combo, not a layer
  chord. It is in git at `217b1fe`.
- Commit straight to `master`: it is a solo fork and git history makes reverts
  easy. Still wait for the user to ask before committing or pushing.
- Accents come from the Windows "United States-International" layout (dead keys
  on `'`, `"`, `` ` ``, `~` and `^`). PowerToys Quick Accent was tried and
  rejected. Symbols are only right on a US layout, not ABNT2; Alt+Shift switches
  layouts.
- ZMK Studio is still built into the left half, but its unlock key is gone, so
  it stays locked. Layouts saved from Studio override the `.keymap` until
  "Restore Stock Settings" is run in Studio.

## Gotchas

- Git warns that LF will be replaced by CRLF on Windows. It is harmless.
- A key that types nothing is usually hardware, not the keymap: the `\|` key was
  a bad switch. Reseat it, then swap it with a working neighbour to tell a bad
  switch from a bad socket.
- The `,` key died after a switch swap, and the keymap was fine (`&kp COMMA`,
  position 46). A hot-swap socket's left leaf had a cracked solder joint; the
  fix was reflowing it. Single-part meter checks all passed because the probe
  tip pressed the crack shut, so do not trust a lone `000` on a joint. To find
  a dead key without guessing:
  1. Hold SYM and press it. If the SYM symbol does not type either, the
     matrix never sees the key, so it is hardware.
  2. With the keyboard on, touch the two socket leaves together with tweezers.
     If the key types, only the switch-to-socket contact is bad.
  3. Board off, meter in diode mode, switch removed, touching lightly: red on
     the key's column pin, black on each socket leaf and on both diode legs.
     Compare with a working key in the same column. A good key reads `000` on
     the column-side leaf and `OL` on the other, and about `0.5` through the
     diode (stripe end).
  Nice!nano pin labels: rows 0-4 are `024` `100` `011` `104` `106`; right
  columns 6-11 are `009` `010` `111` `113` `115` `002`; left columns 0-5 are
  `002` `115` `113` `111` `010` `009`.
- The user is new to git, forks and ZMK. Explain basics, go one step at a time,
  and say what each command does.
