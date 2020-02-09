##### This research was done on patch 1.48, ScriptHook gameversion 50.

# Research of selector.c for GTA V

## Scaleform Download / Function Definitions

Scaleform is used for many complex UI elements, including the selector UI (known as `player_switch` in scaleform). You can find a link to ALL scaleforms [here](https://mega.nz/#!RcFwkCgA!SSi4wOWGtfPZd4-5yaytKCwFCcfytQnOAieOKFQLPb8).

Here are a few important definitions from these scripts.

```
function SET_SWITCH_SLOT(index, stateEnum, charEnum, selected, pedheadshot_txt_string)

stateEnum:
["", "AVAILABLE", "UNAVAILABLE", "NOTMET"]
```

## Switch Data Global (0x4378)

This is the global object for most of the switching data. Most of the data is offset from this object.

## Has Scaleform Loaded (0x4378)

This is set to 1 the instant you press alt, when the scaleform is loaded and ready to be used. This will stay 1 until the scaleform is marked as no longer needed.

## Currently Selected Swap Player (0x43BD)

The global `0x4378 + 0x45 (0x43BD)` contains the currently selected swap player, as by the selector UI. If the UI isn't open, this will be -1.

```
Values:
-1: not open
0: franklin
1: trevor
2: mp char
3: michael
```

## Has Changed Selection (0x4386)

The global `0x4378 + 0xE (0x4386)` is set to 1 if any change to the selected swap target is made. Done _after_ changing the selected swap player, see above.

## Has Met / Is Available Arrays _(unconfirmed)_

The global `0x4378 + 0x13 (0x438B)` is an array indexed by a character index. It is used to define if we have met this player yet.

The global `0x4378 + 0x1D (0x4395)` is an array indexed by a character index. It is used to define if this player is available. _Example:_ when Michael gets abducted by the Chinese.

## Is Swapping Disabled (0x437D)

The global `0x4378 + 0x5 (0x437D)` can be used to disable character swapping. When holding alt, the UI will no longer show, and the timescale will no longer slow. Character swap buttons will also stop working.
