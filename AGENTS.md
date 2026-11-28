# AGENTS.md

Personal fork (`danielrfh`) of `KinesisCorporation/Adv360-Pro-ZMK`: a ZMK user config for the Kinesis Advantage 360 Pro split keyboard. There is no app code and there are no tests or linters. The only way to verify a change is a successful firmware build.

## Where things live
- `config/adv360.keymap` is the only keymap. `adv360_left.keymap` and `adv360_right.keymap` just `#include` it.
- Macros go in `config/macros.dtsi`, which is included inside the keymap's `behaviors {}` node. Behaviors that take parameters, such as the hold-taps and the one-param macro `once`, are defined in that node in the keymap itself, where keymap-editor can read their `#binding-cells` without following the `#include`. `config/boards/arm/adv360/macros.dtsi` is a stale copy that nothing includes.
- There are no `config/*.conf` files. Kconfig lives in `config/boards/arm/adv360/adv360_{left,right}_defconfig`:
  - Left is the split central, so host-facing options (`CONFIG_BT_BAS`, `CONFIG_ZMK_HID_KEYBOARD_EXTENDED_REPORT`) go in the left file.
  - LED, backlight and indicator-color options apply per half, so set them in both files.
- The build uses neither `config/keymap.json` (old Kinesis GUI, stale) nor `config/info.json` (layout data for the web keymap editor). Don't copy keymap edits into them.
- ZMK itself is not in this repo. `config/west.yml` pins `ReFil/zmk` @ `adv360-z3.5-2`, the Kinesis fork on Zephyr 3.5. zmk.dev documents newer ZMK, and the README says its RGB underglow, backlight and power-management pages don't apply here. Before using a behavior or keycode, check it exists in the fork (`app/dts/behaviors/`, `app/include/dt-bindings/zmk/`). A snapshot is under "Behaviors in the pinned fork" below.
- The board definition uses Zephyr's legacy `boards/arm/` format. Don't migrate it to HWMv2 / `board.yml`.
- `KEYMAP.md` is the guide to the layout for people: layers, thumb keys, special keys and LED colours. Update it when a change alters how a key or layer behaves. `README.md` is upstream's; leave it unchanged so "Sync fork" never conflicts on it.

## Keymap gotchas
- Every layer needs exactly 76 bindings in key-position order. See `assets/key-positions.md`; thumb keys sit mid-row in rows 3–5 of each block. A wrong count can still build, but it shifts keys.
- Layers are referenced by index: `&tog 1` (Win), `&mods_toggle 2 0` (ModsOff), `&tog 3` (keypad), `&mo 4` (fn), `&mo 5` (mod), and `&vim_bspc 6 …` / `&vim_space 6 …` (Vim) in the keymap, `if-layers = <1 4>` and `then-layer = <7>` (FnWin) in its `conditional_layers` node, plus `&to 0` and `&to 1` in the `Mac_BT` / `Win_BT` macros in `config/macros.dtsi`. The highest active layer's index also sets the indicator LED color (see the README table and `KEYMAP.md`). Inserting or reordering layers means updating all of those references.
- `Win` (index 1) is a mostly-`&trans` overlay on `Mac` (index 0), toggled with `&tog 1`. Put Windows-only differences there.
- Mac-only features live on `Mac`: home-row mods at 29–32 and 41–44, Shift-select taps at 46 and 59, hold ⌫/␣ (65, 70) for `Vim` (index 6), where holding m toggles `ModsOff` (index 2), and the Hyper/Meh holds on Del, End, PgDn and Enter (66–69). kanata does these on Windows, so `Win` has a plain key at each of those positions.
- `Win` (1) and `ModsOff` (2) sit below `keypad` (3), `fn` (4) and `mod` (5), so those layers override the plain keys of `Win` and `ModsOff` and act the same as on Mac. Keep that order: a layer that must work the same on both computers goes above `ModsOff`. `Vim` (6) stays above them. It can only be turned on from `Mac` or `ModsOff`, because `Win`, keypad and mod don't have the Vim holds at 65 and 70, and `Vim` has `&none` on the keypad and mod keys. `FnWin` (7) is a conditional layer, on only while `Win` and `fn` are both on, so it is never on together with `Vim`.
- `fn` (4) has the system keys of kanata's system layer (hold ˋ on the PC) as Mac shortcuts, and `FnWin` (7) puts the Windows versions over them. A `fn` key with a Mac-only shortcut needs a Windows shortcut or `&none` at the same position on `FnWin`; the F-keys and media keys work on both computers and stay `&trans` there. Send a new system key with `&once <shortcut>`, so it fires once however long the key is held, or `&hold_once <shortcut> 0` if it needs a 350 ms hold. Only volume uses `&kp`, so it repeats.
- On the PC, kanata sees each `FnWin` shortcut before Windows does. Check a new one in the kanata config (`danielrfh/kanata`, checked out at `~/.config/kanata`) with `node sim.mjs`, pressing the modifiers and the key at the same moment, and add an `Adv360:` case to its `tests/cases.json`.
- Devicetree node names must differ in more than case. Zephyr lowercases them in generated names, so a hold-tap `win_sleep` next to the macro `Win_Sleep` fails to link with `multiple definition of __device_dts_ord_…`.
- Mod+1 / Mod+2 run `&Mac_BT` / `&Win_BT`: they select BT profile 1 / 2, then run `&to 0` (Base/Mac) / `&to 1` (Win), which also turns off Keypad, `ModsOff` and a still-held `&mo 5`. The active profile is saved to flash about 30 s after a switch (`CONFIG_ZMK_SETTINGS_SAVE_DEBOUNCE`) and survives power-off, but layer state doesn't. There's no deep sleep (`CONFIG_ZMK_SLEEP` is off), so only a power cycle, reset or flash clears layers. After power-on on profile 2, press Mod+2 again; selecting the active profile is a no-op, so it doesn't disconnect.
- `&macro_ver` (on the mod layer) is only defined in the generated `config/version.dtsi`. The committed file is empty on purpose and there is no fallback.
- Most keymap commits come from `keymap-editor[bot]` (Nick Coutsos's web editor). It pushes straight to `origin/V3.0` and re-aligns the binding columns itself. Run `git pull` before editing, and don't hand-realign layers. It doesn't support preprocessor macros, so don't add `#define`s to the keymap: write key positions and layer indices as plain numbers.
- On Studio-enabled ("Clique") firmware, edits saved from ZMK Studio or Clique live in the keyboard's settings storage. Flashing doesn't erase it, and the edits are re-applied over the flashed keymap at every boot. Only the keys edited there (plus layer names and layer order) keep their Clique version; other keys follow the new `.keymap`. Saved edits are keyed by layer index and key position, so inserting or reordering layers in the `.keymap` moves them onto other layers. Suspect this when a flashed change doesn't show up.
- "Restore Stock Settings" in ZMK Studio or Clique (unlock first with Mod+Esc, `&studio_unlock`) deletes saved bindings that differ from the running firmware's keymap, plus saved layer names and order, and applies the stock keymap immediately. It doesn't touch BT pairings, the active profile or lighting; `settings-reset.uf2` is the full wipe that also unpairs everything. Flash first, then restore. It drops every Clique edit and can't be undone, so move edits worth keeping into `config/adv360.keymap` first.

## Behaviors in the pinned fork
Snapshot of `ReFil/zmk` `adv360-z3.5-2` @ `d93499b` (2026-08-27), from the fork's source and a local build's `.config`. The branch moves, so re-check anything not listed here.
- Default instances: `&kp`, `&mt`, `&lt`, `&mo`, `&tog`, `&to`, `&sl`, `&sk`, `&kt`, `&trans`, `&none`, `&caps_word`, `&key_repeat`, `&gresc`, `&bt`, `&out`, `&sys_reset`, `&bootloader`, `&ext_power`, `&bl`, `&rgb_ug`, `&stp` (Kinesis battery indicator, `STP_BAT`), `&studio_unlock`, and the mouse behaviors `&mkp` / `&mmv` / `&msc`.
- `&soft_off` has a devicetree node but no driver. It needs `CONFIG_ZMK_PM_SOFT_OFF=y`, which isn't set.
- Compatibles for custom behaviors: `zmk,behavior-hold-tap` (used by `mtb` and the Mac-only features), `zmk,behavior-tap-dance` (`sel_l_taps` / `sel_r_taps`), `zmk,behavior-mod-morph` (`power_key`), `zmk,behavior-sticky-key`, `zmk,behavior-toggle-layer`, and `zmk,behavior-macro` (plus `-one-param`, used by `once`, and `-two-param`). Combos and conditional layers are compiled in.
- Conditional layers: entries that share a `then-layer` are OR-ed, so one layer can have several conditions.
- Hold-tap: `require-prior-idle-ms` counts from the last non-modifier keycode sent; layer keys and modifiers don't count. With `retro-tap`, a timeout doesn't press the hold binding; it is pressed when another key goes down, and letting go first sends the tap instead. A hold-tap invokes its bindings with its own press time and decides a tap only on release, so a tap-dance used as its tap binding times its window from that press.
- `&tog` supports `toggle-mode = "on" | "off" | "flip"` (default `flip`). For an on-only or off-only toggle, define your own `zmk,behavior-toggle-layer` instance with that property.
- `&to N` turns off every layer except the default (0), including `&mo` layers that are still held, then turns on N.
- Macros can chain any behavior, including `&bt` and layer changes. Each binding is tapped by default (30 ms tap, 15 ms wait); `&macro_press`, `&macro_release`, `&macro_pause_for_release`, `&macro_tap_time`, `&macro_wait_time` and `&macro_param_*` change that.
- `&bt BT_SEL n` does nothing if profile `n` is already active.
- Keycode headers: the keymap includes `bt.h`, `keys.h`, `outputs.h` (`OUT_USB`, `OUT_BLE`, `OUT_TOG` for `&out`), `rgb.h`, `backlight.h` and `stp.h`. Mouse values (`MB1`–`MB5`, `MOVE_*`, `SCRL_*`) are already in scope, as `Double_Click` shows.
- HID: only the keyboard and consumer usage pages are sent, so Generic Desktop keycodes such as `SYS_SLEEP` and `SYS_PWR` do nothing. `CONFIG_ZMK_HID_CONSUMER_REPORT_USAGES_BASIC=y` limits consumer keys to usages up to 0xFF, so `GLOBE`, `C_AL_*` and `C_AC_*` do nothing either. Implicit modifiers work on consumer keys, as `LA(C_VOL_UP)` on `fn` shows. A keyboard shortcut's modifiers and key go down in one report and come up in one report.
- Output: while USB (to a computer) and the active BT profile are both connected, keystrokes go to the preferred transport, which is USB until `&out` (Mod+=) changes it. The choice is saved to flash.
- The Studio build (left half) uses the `studio-rpc-usb-uart` snippet. It sets `ZMK_BEHAVIORS_KEEP_ALL`, so every default behavior is compiled in and can be assigned from Studio/Clique. Non-Studio builds (the right half and the Legacy CI job) drop default behaviors that nothing references.
- Studio/Clique show a behavior's `display-name` (falling back to its node name), so give new behaviors and macros one. Saved bindings refer to a CRC16 of the behavior's device name (its node name, or its `label` if set), so renaming a node orphans saved bindings that use it. Those keys then fall through to the layer below, like `&trans`.
- For C code (custom behaviors, event listeners), the fork's `app/CMakeLists.txt` accepts `-DZMK_EXTRA_MODULES`, but `bin/build.sh`, CI and the Makefile mounts don't provide one. To make a layer follow the BT profile automatically, listen for `zmk_ble_active_profile_changed`. It fires on a profile switch, when the active profile's host connects or disconnects, and when a profile is paired or cleared.

## Build
- `make left` builds only the left (central) half. That is enough to check and flash keymap or macro changes.
- `make` builds both halves. Both need flashing after changes to the right-half Kconfig, the board files, or `west.yml`.
- Output goes to `firmware/<UTC timestamp>-<HEAD short sha>-{left,right}-clique.uf2`. It is built from the working tree, so uncommitted edits are included.
- Run `make` from the repo root. The Makefile mounts `$(PWD)/config` and `$(PWD)/firmware`, so `make -C` breaks.
- The build needs Podman or Docker; Podman is used when both are installed (on macOS, run `podman machine start` first).
- The container runs with `-it`. If Docker reports "the input device is not a TTY", run `script -q /dev/null make left`.
- The first run builds the image (it clones Zephyr), so it is slow.
- `make` writes `config/version.dtsi` with `bin/get_version_local.sh`, then restores it with `git checkout`. A failed build skips the restore: run `git checkout config/version.dtsi`. Never commit a filled-in version of this file.
- ZMK and Zephyr are copied into the `zmk` image when it is built:
  - Editing `config/west.yml` triggers a rebuild automatically.
  - Picking up new commits on the same `revision` needs `make clean_image`, which also removes the base `zmk-build-arm:stable` image so it gets re-pulled. CI fetches both fresh on every run, so a stale local image builds against different code than CI. Compare `podman run --rm zmk git -C zmk log -1 --oneline` with the head of `ReFil/zmk` `adv360-z3.5-2`.
  - `make clean_firmware` and `make clean` delete every `.uf2` in `firmware/`, including older builds.
- Don't run `west` on the host in the repo root. `west update` clones `zephyr/`, `zmk/`, `modules/` and more there, and none of it is gitignored. `west build` also fails unless `bin/get_version*.sh` ran first.
- The build commands exist in three places; keep them in sync:
  - `bin/build.sh` for local builds (Studio on the left half only, via `-S studio-rpc-usb-uart -DCONFIG_ZMK_STUDIO=y`)
  - the Legacy job in `.github/workflows/build.yml` (no Studio)
  - the Clique job in the same workflow (Studio on the left half)

## CI & git
- CI runs on every push and PR (about 3 min) and uploads `firmware-clique` and `firmware-no-clique` artifacts. Status without auth: `https://api.github.com/repos/danielrfh/Adv360-Pro-ZMK/actions/runs?per_page=5`.
- `V3.0` is the default and only branch. Upstream (`KinesisCorporation/Adv360-Pro-ZMK` @ `V3.0`) is merged in with GitHub's "Sync fork" button; there is no local `upstream` remote.
- Put each change in its own commit, for example keymap and docs apart, so any one of them can be reverted on its own. Push to `origin/V3.0` right after committing. If the push is rejected because `keymap-editor[bot]` pushed in the meantime, run `git pull --rebase` (this clone has `pull.rebase=false`, so a plain `git pull` would merge) and push again.
- `CHANGELOG.md` entries and `.github/pull_request_template.md` are for PRs to upstream Kinesis. Personal keymap commits don't add changelog entries.
