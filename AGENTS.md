# AGENTS.md

Personal fork (`danielrfh`) of `KinesisCorporation/Adv360-Pro-ZMK`: a ZMK user config for the Kinesis Advantage 360 Pro split keyboard. There is no app code and there are no tests or linters. The only way to verify a change is a successful firmware build.

## Where things live
- `config/adv360.keymap` is the only keymap. `adv360_left.keymap` and `adv360_right.keymap` just `#include` it.
- Macros go in `config/macros.dtsi`, which is included inside the keymap's `behaviors {}` node. `config/boards/arm/adv360/macros.dtsi` is a stale copy that nothing includes.
- There are no `config/*.conf` files. Kconfig lives in `config/boards/arm/adv360/adv360_{left,right}_defconfig`:
  - Left is the split central, so host-facing options (`CONFIG_BT_BAS`, `CONFIG_ZMK_HID_KEYBOARD_EXTENDED_REPORT`) go in the left file.
  - LED, backlight and indicator-color options apply per half, so set them in both files.
- The build uses neither `config/keymap.json` (old Kinesis GUI, stale) nor `config/info.json` (layout data for the web keymap editor). Don't copy keymap edits into them.
- ZMK itself is not in this repo. `config/west.yml` pins `ReFil/zmk` @ `adv360-z3.5-2`, the Kinesis fork on Zephyr 3.5. zmk.dev documents newer ZMK, and the README says its RGB underglow, backlight and power-management pages don't apply here. Before using a behavior or keycode, check it exists in the fork (`app/dts/behaviors/`, `app/include/dt-bindings/zmk/`). A snapshot is under "Behaviors in the pinned fork" below.
- The board definition uses Zephyr's legacy `boards/arm/` format. Don't migrate it to HWMv2 / `board.yml`.

## Keymap gotchas
- Every layer needs exactly 76 bindings in key-position order. See `assets/key-positions.md`; thumb keys sit mid-row in rows 3–5 of each block. A wrong count can still build, but it shifts keys.
- Layers are referenced by index: `&tog 1`, `&mo 2`, `&mo 3` and `&tog 4` in the keymap, plus `&to 0` and `&to 4` in the `Mac_BT` / `Win_BT` macros in `config/macros.dtsi`. The index also sets the indicator LED color (see the README table). Inserting or reordering layers means updating all of those references.
- `Win` (index 4) is a mostly-`&trans` overlay on `Mac` (index 0), toggled with `&tog 4`. Put Windows-only differences there.
- Mod+1 / Mod+2 run `&Mac_BT` / `&Win_BT`: they select BT profile 1 / 2, then run `&to 0` (Base/Mac) / `&to 4` (Win), which also turns off Keypad and a still-held `&mo 3`. The active profile is saved to flash about 30 s after a switch (`CONFIG_ZMK_SETTINGS_SAVE_DEBOUNCE`) and survives power-off, but layer state doesn't. There's no deep sleep (`CONFIG_ZMK_SLEEP` is off), so only a power cycle, reset or flash clears layers. After power-on on profile 2, press Mod+2 again; selecting the active profile is a no-op, so it doesn't disconnect.
- `&macro_ver` (on the mod layer) is only defined in the generated `config/version.dtsi`. The committed file is empty on purpose and there is no fallback.
- Most keymap commits come from `keymap-editor[bot]` (Nick Coutsos's web editor). It pushes straight to `origin/V3.0` and re-aligns the binding columns itself. Run `git pull` before editing, and don't hand-realign layers.
- On Studio-enabled ("Clique") firmware, once the keymap has been edited in ZMK Studio or Clique, later `.keymap` changes are ignored until "Restore Stock Settings". Suspect this when a flashed change doesn't show up.

## Behaviors in the pinned fork
Snapshot of `ReFil/zmk` `adv360-z3.5-2` @ `d93499b` (2026-08-27), from the fork's source and a local build's `.config`. The branch moves, so re-check anything not listed here.
- Default instances: `&kp`, `&mt`, `&lt`, `&mo`, `&tog`, `&to`, `&sl`, `&sk`, `&kt`, `&trans`, `&none`, `&caps_word`, `&key_repeat`, `&gresc`, `&bt`, `&out`, `&sys_reset`, `&bootloader`, `&ext_power`, `&bl`, `&rgb_ug`, `&stp` (Kinesis battery indicator, `STP_BAT`), `&studio_unlock`, and the mouse behaviors `&mkp` / `&mmv` / `&msc`.
- `&soft_off` has a devicetree node but no driver. It needs `CONFIG_ZMK_PM_SOFT_OFF=y`, which isn't set.
- Compatibles for custom behaviors: `zmk,behavior-hold-tap` (used by `mtb`), `zmk,behavior-tap-dance`, `zmk,behavior-mod-morph`, `zmk,behavior-sticky-key`, `zmk,behavior-toggle-layer`, and `zmk,behavior-macro` (plus `-one-param` / `-two-param`). Combos and conditional layers are compiled in.
- `&tog` supports `toggle-mode = "on" | "off" | "flip"` (default `flip`). For an on-only or off-only toggle, define your own `zmk,behavior-toggle-layer` instance with that property.
- `&to N` turns off every layer except the default (0), including `&mo` layers that are still held, then turns on N.
- Macros can chain any behavior, including `&bt` and layer changes. Each binding is tapped by default (30 ms tap, 15 ms wait); `&macro_press`, `&macro_release`, `&macro_pause_for_release`, `&macro_tap_time`, `&macro_wait_time` and `&macro_param_*` change that.
- `&bt BT_SEL n` does nothing if profile `n` is already active.
- Keycode headers: the keymap includes `bt.h`, `keys.h`, `rgb.h`, `backlight.h` and `stp.h`. Mouse values (`MB1`–`MB5`, `MOVE_*`, `SCRL_*`) are already in scope, as `Double_Click` shows. The `&out` values (`OUT_USB`, `OUT_BLE`, `OUT_TOG`) need `#include <dt-bindings/zmk/outputs.h>`, which the keymap doesn't have yet.
- The Studio build (left half) uses the `studio-rpc-usb-uart` snippet. It sets `ZMK_BEHAVIORS_KEEP_ALL`, so every default behavior is compiled in and can be assigned from Studio/Clique. Non-Studio builds (the right half and the Legacy CI job) drop default behaviors that nothing references.
- Studio/Clique show a behavior's `display-name` (falling back to its node name), so give new behaviors and macros one. Saved bindings refer to a CRC16 of the behavior's device name (its node name, or its `label` if set), so renaming a node orphans saved bindings that use it. Since `d93499b`, orphaned bindings act as `&trans`.
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
  - Picking up new commits on the same `revision` needs `make clean_image`.
  - `make clean_firmware` and `make clean` delete every `.uf2` in `firmware/`, including older builds.
- Don't run `west` on the host in the repo root. `west update` clones `zephyr/`, `zmk/`, `modules/` and more there, and none of it is gitignored. `west build` also fails unless `bin/get_version*.sh` ran first.
- The build commands exist in three places; keep them in sync:
  - `bin/build.sh` for local builds (Studio on the left half only, via `-S studio-rpc-usb-uart -DCONFIG_ZMK_STUDIO=y`)
  - the Legacy job in `.github/workflows/build.yml` (no Studio)
  - the Clique job in the same workflow (Studio on the left half)

## CI & git
- CI runs on every push and PR (about 3 min) and uploads `firmware-clique` and `firmware-no-clique` artifacts. Status without auth: `https://api.github.com/repos/danielrfh/Adv360-Pro-ZMK/actions/runs?per_page=5`.
- `V3.0` is the default and only branch. Upstream (`KinesisCorporation/Adv360-Pro-ZMK` @ `V3.0`) is merged in with GitHub's "Sync fork" button; there is no local `upstream` remote.
- `CHANGELOG.md` entries and `.github/pull_request_template.md` are for PRs to upstream Kinesis. Personal keymap commits don't add changelog entries.
