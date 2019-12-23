##### This research was done on patch 1.27, ScriptHook gameversion 4.

# General knowledge of YSC scripts

## Globals

Scripts have global memory to use and pass variables through, statically. These offsets are the same between scripts of the same version, but **_will change with each patch_**.

Decompiled globals are all in hexadecimal, and look like this: `Global_26BECF`, or `Global_40001.f_B44`.

## YSC Arrays

Arrays in memory are length-prefixed. This means the address to an array is **not it's first element**, but it's size.

## Types

Structs of variable size can be passed around and returned by functions.

## Hash

This game uses `Jenkins one-at-a-time` hashing, used on many things within the game.

Model names, X360 natives (32-bit version of GTA), GXT entries, etc. Zorg's decompiler should know some of these and make a `joaat` function with the original string. However, sometimes it doesn't. Try to add a comment anywhere there is a hash with a missing translation.

---

## Misison Fail String (0x10B7D)

This global is updated with the GXT entry of why the mission failed, shown on the fail screen.

## Unknown Mission Global (0x15F8E)

This global is set to `0` upon mission fail.

Seems to be an internal value used for `flow_controller` and `mission_repeat_controller`
