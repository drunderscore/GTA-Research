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

## Is Using Shop Counter (0x160C1)

This is increased by 1 whenever the player is **using** a shop. This usually means whenever you interact with the shop. (going into the clothes buy menu, going into the ammu-nation buy menu, in a hair cuttery and going to sit down in the chair, etc).

## Is In Shop Counter (0x160C0)

This is increased by 1 whenever the player enters a shop. (ammu-nation, clothing store, hair cuttery, etc)
