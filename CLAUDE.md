## Reply format

- Lead with the deliverable or the answer. Nothing before it.
- One question per reply, at most. Say what you recommend.
- Flags only if they block the work or change a decision. Max 3, one line each. Everything else goes into the files, not the reply.
- Report file edits in one line ("Updated X and Y"), never as a list.
- Step-by-step work (a group, a page, a sheet): show only that step. No look-ahead and no recap of earlier steps.
- Folder and page names go in code blocks, never in markdown lists (numbered names render as a. b. c.).
- If more detail might help, offer it in one line instead of including it.
## Project

- Scylla mk2 split keyboard, wireless, nice!nano v2 on both halves. Fork of the Tupinikeebs keyboard builder; `origin` is the user's fork (`KoliaK/scylla_zmk`). It uses the Lily58 shield, so the keymap file is `config/lily58.keymap`, not `scylla.keymap`.
- ZMK is pinned to v0.3 (`config/west.yml`, `.github/workflows/build.yml`, `build.yaml`). Do not bump it without changing all three.
- The user's design notes are in `reference_files/` (untracked, theirs): `1-base_keymap.md`, `2-coding_layer.md`, photos. The keymap implements them; the layer diagrams live in comments inside `config/lily58.keymap`.
- Key position indexes (Lily58 order): thumbs left `52` Alt, `53` Space, `42` SYM, `50` Tab, `51` F2; right `55` Del, `54` Bspc, `43` GUI, `56` F6/Snip, `57` NAV. The rest follow the 12-per-row order; the full map is at the top of the keymap section.
- Layers: Base, SYM (hold `42`), NAV (hold `57`), SYSTEM (SYM + NAV held together: Bluetooth, USB/BLE toggle, Studio unlock).
- Firmware is built by GitHub Actions on every push. Download the `firmware` artifact from the run, then flash `scylla_left` and `scylla_right` (double-tap the reset button, copy the `.uf2` to the drive that appears). Flash `settings_reset` on both halves first only if pairing breaks.
- Check after editing the keymap: every layer must list exactly 58 bindings.
