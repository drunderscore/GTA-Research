##### This research was done on patch 1.48, ScriptHook gameversion 50.

# General knowledge of YSC scripts

## Globals

Scripts have global memory to use and pass variables through, statically. These offsets are the same between scripts of the same version, but **_will change with each patch_**.

Decompiled globals are all in hexadecimal, and look like this: `Global_26BECF`, or `Global_40001.f_B44`.

## YSC Arrays

Arrays in memory are length-prefixed. This means the address to an array is **not it's first element**, but it's size.

## Types

Structs of variable size can be passed around and returned by functions.

## Hash

This game uses `Jenkins one-at-a-time` hashing.

More times than not, JOAAT hashes aren't hashed during runtime, and are already calculated, baked into the script. We actually **know** some of these, however, the decompiler used did not at the time. You will see these often as seemingly, large, even negative numbers.
