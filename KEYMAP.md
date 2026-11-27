# Keymap

What this fork's keymap does. One layout serves a Mac on Bluetooth profile 1 and a Windows PC on profile 2. For building, flashing and the full layer-colour table, see the [upstream README](README.md). The bindings are in [config/adv360.keymap](config/adv360.keymap). Numbers in brackets are key positions from [assets/key-positions.md](assets/key-positions.md).

## Mac and Windows

| Keys | Bluetooth profile | Layer |
| --- | --- | --- |
| Mod+1 | 1 | Base (Mac) |
| Mod+2 | 2 | Win |
| Mod+3, Mod+4, Mod+5 | 3, 4, 5 | unchanged |

- Mod+1 and Mod+2 also turn off Keypad and Mods off.
- The key right of T (20) turns Win on or off without changing the Bluetooth profile, for example over USB.
- Layers aren't saved, so the keyboard always starts on Base. The profile is saved about 30 s after a switch, so if the keyboard starts on profile 2, press Mod+2 again. Selecting the profile that's already active doesn't disconnect it.
- Win sits on top of Base and only replaces the keys that differ on Windows. On the PC, kanata provides the home-row mods, Shift selection, Vim layer and Hyper/Meh holds, so Win has a plain key in each of those places.

## Thumb keys

| Key | Base (Mac) | Win |
| --- | --- | --- |
| Left thumb, upper pair (35, 36) | Cmd, Opt | Ctrl, Alt |
| Right thumb, upper pair (37, 38) | Opt, Cmd | Alt, Ctrl |
| Home (52), PgUp (53) | tap: key, hold: Ctrl | tap: key, hold: Win |
| Backspace (65), Space (70) | tap: key, hold: [Vim layer](#vim-layer-mac) | key |
| Del (66), Enter (69) | tap: key, hold: Hyper (Ctrl+Opt+Cmd+Shift) | key |
| End (67), PgDn (68) | tap: key, hold: Meh (Ctrl+Opt+Shift) | key |

On Base:

- Backspace, Space, Del, End, PgDn and Enter start their hold only when you press another key while holding them. Held on their own, they type their key once when you let go.
- To repeat Backspace, Del, End, PgDn or Enter, tap the key, then press and hold it straight away.
- While you type they stay plain keys: if you pressed another key less than 150 ms earlier, they type their key.
- Hyper and Meh only reach the computer once you press another key, so they don't work with the mouse.

## Home-row mods (Mac)

| Key | A | S | D | F | J | K | L | ; |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Hold | Ctrl | Opt | Cmd | Shift | Shift | Cmd | Opt | Ctrl |

- A held key acts as a modifier only with a key on the other hand or a thumb key. For Cmd+C, hold K and press C. For the left-hand mods, = and the ← → keys also count as the other hand.
- While you type they stay letters: if you pressed another key less than 150 ms earlier (250 ms for A, S, L and ;), the key types its letter.
- Ctrl and Opt (A, S, L, ;) take 500 ms to count as held on their own; Cmd and Shift take 200 ms. If you hold A, S, L or ; and let go without pressing another key, it types its letter.
- Ctrl and Opt from A, S, L and ; only reach the computer once you press another key, so they don't work with the mouse. For Ctrl+click or Opt+drag, hold Home or PgUp (Ctrl) or an Opt thumb key. Cmd and Shift from D, F, J and K do work with the mouse once held for 200 ms.

## Shift selection (Mac)

Both Shift keys are still Shift when held. Tap one several times to select text towards its side:

| Taps | Left Shift | Right Shift |
| --- | --- | --- |
| 2 | previous word | next word |
| 3 | to start of line | to end of line |
| 4 | to start of document | to end of document |

A single tap does nothing. Tap quickly: the taps need to be about 200 ms apart or less.

## Vim layer (Mac)

Hold Backspace or Space.

| Key | Does | Key | Does |
| --- | --- | --- | --- |
| H J K L | ← ↓ ↑ → | W, B | next, previous word |
| 0, 4 | start, end of line | S, F | Opt, Shift: hold with H J K L to move by word or select |
| Y, P | copy, paste | D | cut |
| U, R | undo, redo | X | delete forward |
| / | find | C | Caps Word |
| Tab | Cmd+Tab | Backspace (while holding Space) | delete previous word |
| Esc | Esc | Enter | Enter |
| Q, held | quit app (Cmd+Q) | M, held | [Mods off](#mods-off) on or off |

## Mods off

On the Vim layer, hold M for about a third of a second to turn the home-row mods into plain letters. Do it again, or press Mod+1, to turn them back on. Everything else keeps working.

## Keypad and Fn

- **Keypad:** the top-row key right of 5 (6) turns it on or off. The right half becomes a number pad: 7 8 9 on U I O, 4 5 6 on J K L, 1 2 3 on M , . and 0 on Space, with . on [, Num Lock, =, / and * on 7 8 9 0, and −, + and Enter on P ; /. The rest of the keyboard has plain keys with no holds, except the upper thumb keys, which keep their Base or Win modifiers.
- **Fn:** hold either outer bottom corner key (60, 75). The top row from = to - gives F1 to F12.

## Mod layer

Hold Mod, the top-row key left of 6 (7).

| Key | Does |
| --- | --- |
| 1 to 5 | Bluetooth profile, see [Mac and Windows](#mac-and-windows) |
| = | send keystrokes over USB or Bluetooth, see below |
| Right thumb, upper left (37), held for 1 s | clear the active profile's pairing |
| Key right of T (20), key left of Y (21) | bootloader for the left, right half |
| Esc | unlock ZMK Studio and Clique |
| V | type the firmware version |
| Key left of H (39), held | show the battery level, see [LEDs](#leds) |
| Enter | backlight on or off |
| ↑, ↓ | backlight brighter, dimmer |
| Space | status LEDs on or off |

While the left half is plugged into a computer and a Bluetooth profile is connected too, keystrokes go over USB. Mod+= switches them to Bluetooth, and pressing it again switches back. The keyboard remembers the choice. When only USB or only Bluetooth is connected, that one is used.

## LEDs

Each half has a layer LED that shows the highest layer that's on:

| # | Layer | Left, right |
| ---: | --- | :---: |
| 0 | Base | <img valign='middle' src='assets/swatches/000000.svg'/> <img valign='middle' src='assets/swatches/000000.svg'/> |
| 1 | Win | <img valign='middle' src='assets/swatches/FFFFFF.svg'/> <img valign='middle' src='assets/swatches/FFFFFF.svg'/> |
| 2 | Mods off | <img valign='middle' src='assets/swatches/0000FF.svg'/> <img valign='middle' src='assets/swatches/0000FF.svg'/> |
| 3 | Keypad | <img valign='middle' src='assets/swatches/00FF00.svg'/> <img valign='middle' src='assets/swatches/00FF00.svg'/> |
| 4 | Fn | <img valign='middle' src='assets/swatches/FF0000.svg'/> <img valign='middle' src='assets/swatches/FF0000.svg'/> |
| 5 | Mod | <img valign='middle' src='assets/swatches/FF00FF.svg'/> <img valign='middle' src='assets/swatches/FF00FF.svg'/> |
| 6 | Vim | <img valign='middle' src='assets/swatches/00FFFF.svg'/> <img valign='middle' src='assets/swatches/00FFFF.svg'/> |

- Keypad, Fn and Mod sit above Win and Mods off, so they show the same colour on the Mac and on Windows.
- The left half also has a Caps Lock LED and a Bluetooth LED: profile 1 white, 2 blue, 3 red, 4 green, 5 off. It blinks fast while the profile isn't paired, and slowly while it's paired but not connected.
- The right half also has Num Lock and Scroll Lock LEDs. All its LEDs blink red while it isn't connected to the left half.
- While you hold Mod and key 39, each half shows its own battery level: green at 80 % or more, yellow at 50 % or more, orange at 20 % or more, red below that.

## Clique and ZMK Studio

Edits saved from Clique or ZMK Studio are stored on the keyboard and applied over the flashed keymap at every start, so flashing doesn't remove them. If a flashed change doesn't show up, that key was probably edited in Clique. Each edit belongs to a layer number, so reordering the layers in the keymap moves saved edits onto other layers.

"Restore Stock Settings" (unlock first with Mod+Esc) deletes those edits, along with saved layer names and order. It can't be undone, so copy any edit worth keeping into `config/adv360.keymap`, flash, and then restore. Bluetooth pairings and lighting aren't affected.

## Changing the keymap

- Keymap changes only need the left half: run `make left` and flash the left half.
- Put Windows-only differences on the Win layer. Keep Win and Mods off below Keypad, Fn and Mod in the layer order, so those layers work the same on both computers; see [AGENTS.md](AGENTS.md).
- Update this file when a change alters how a key or layer behaves. `README.md` stays as upstream's, so syncing the fork never conflicts on it.
