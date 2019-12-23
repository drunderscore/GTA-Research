##### This research was done on patch 1.27, ScriptHook gameversion 4.

# Notifications in GTA V

The game keeps track of texts, emails, and phone calls (collectively 'notifications') through a single script, `comms_controller.ysc`.
Most (if not all) notifications that are received during freeroam are managed in this script. The text for carbine rifles, the text from Wade for minisub, the text from Lester telling Michael to invest in stocks, etc.

This write-up is a technical description of how the internals of the notification system work. For more generalized info on notification timings, rules, and specific workings, another write-up will be made. _(unfinished)_

## Timing notifications

The native `GAMEPLAY::GET_GAME_TIMER` is used to schedule notifications. It simply counts in milliseconds since the 'game started' (rough definition, however unimportant as its simply used as a timer).
This **IS** effected by timescale (switching chars, radio station, selecting weapon, etc).

## Next Notification Time (0x8A88)

The global `0x8A88` tracks the next time the script is allowed to send a notification. If one is sent successfully, this value is set to the **current game time**, plus 20000 (20 seconds).
This value is initialized VERY early on in the script, with the exact line below.

`Global_8A88 = GAMEPLAY::GET_GAME_TIMER() + 20000;`

This means that a notification is **impossible** to be sent until 20 seconds after this script starting.

## Global Char Struct Array (0x20DD1)

The global `0x17C49 + 0x744E (0x1F097)` contains an array of some kind of structure. The elements have a size of `29`.

I have not researched this very much, however it is important as seen below when referenced. A general outline of some important values is seen below.

```
Offsets:
0: ped model hash.
1: int, usually whole and resonable. usually equal to offset 2.
2: read above.
12: array, indexed by char index?
3: Character/notif name GXT entry (CELL_*)
7: Character/notif pic GXT entry (CELL_*)
```

All character names and character pictures are relative to each other with their GXT entries. By adding 200 (decimal) to the name, you can get pic name, and vice-versa by subtracting 200.
_Example:_
The string `Michael` is GXT entry `CELL_101`, and his picture `CHAR_MICHAEL` is `CELL_301`.

This is essentially linking ped model hashes to other data. Other important data is probably stored here aswell.

## Important Areas Array (0x8862)

The global `0x8862` is an array of objects that are `5` in size, with a capacity of `76`.
It contains 'important areas' that notifications can index in order to prevent them from being sent when the player is too close to one of these positions. _Example:_ Simion's first phone call to Franklin. Franklin can't be near Simion's place, OR his safehouse, for the call to be received.

The objects are structued like this:

```
Offsets:
0: Vector3, position of the important loation.
1: (y component of vector above)
2: (z component of vector above)
3: float, how far do you have to be from this position.
4: int, index to this array (see below).
```

Offset 4 is usually used recursively to index this array. _Example:_ The first phone call from Simion checks that we aren't near his place (index 74). Index 74, offset 4, has it's value as 0. Because of this, we do the same check as above (recursively), but with index 0 of this array (which is Franklin's safehouse). This is how the game assures you aren't near either of those important areas.

## Text Messages, Emails, and Phone Calls

These are the three types of notifications that the player is eligible to receive. They are all separated into arrays of their own data structures, however each type share the same base class.

The array of text message objects is stored at `0x17C49 + 0x1738 + 0x28B (0x1960C)`. The objects contained are `14` in size.
The count of objects in the array is stored at `0x17C49 + 0x1738 + 0x2FC (0x1967D)`.

The array of email objects is stored at `0x17C49 + 0x1738 + 0x2FD (0x1967E)`. The objects contained are `10` in size.
The count of objects in the array is stored at `0x17C49 + 0x1738 + 0x362 (0x196E3)`.

The array of phone call objects is stored at `0x17C49 + 0x1738 (0x19381)`. The objects contained are `15` in size.
The count of objects in the array is stored at `0x17C49 + 0x1738 + 0x88 (0x19409)`.

The basic structure is seen below. **This is obviously not complete.**

```
Offsets:
0: Probably a hash. This value corresponds to what notification this is (what text to show). is NOT the gxt label hash.
1: bitfield.
2: bitfield. Tells who gets this text. bit 0 for michael, bit 1 for franklin, bit 2 for trevor.
3: int, 5 and greater seems to be an important value.
4: Next notification attempt time.
5: Next notification time delta, or notification time offset. how much to increase offset 4 by. (offset 4) = (GAMEPLAY::GET_GAME_TIMER() + this), used when this notification couldn't send for whatever reason.
6: defines the sender, index into global char struct.
7: index into important areas array. if too close, this will not send. (see above for more details on this array)
9: index of a switch-case (no global array!) of functions to execute when attempting to send. if the function executed doesn't return true, we don't send. (used for extra logic, when?) the execution is done dynamically.
```

## Notification Logic

There are a few steps to receiving a notification. We'll go in the same steps as the game does, in order.

### Global next notification time

This is the next game time that any notification can be received. If the current game time is **greater than or equal to** this value, we are ready to attempt a notification.

This value is only updated **when a notification is successfully sent**.

### How to choose which notification to send?

The type of notification that is attempted to be sent is _"essentially"_ random. Because the game is always looping to attempt to send each type of notification separate from each other, it is all up to randomness in execution time whether we land on a phone call, text message, or email.

Other than that, the specific notification that will be sent is always loaded the same. _(unconfirmed?)_

### Can we send this specific notification?

There area a few general rules when sending notification the game checks for, aswell as a few specific values from the notification data. While not all the rules are understood yet, this will be updated with specifics once we do know.

### The notification couldn't be sent!

When the above rules fail to check out, the game will not send that specific notification. Instead, the notification data will be updated with its own internal _"next notification time"_.

The value will be sent with the current game time, plus a value specific to each notification. (some are short like 10 seconds, some are very long like 2 minutes)

This means that not only does the global next notification time have to be ready, but the current game time has to be **greater than or equal to** this internal next notification time for this notification to be sent.

Because of this, some notification may seem to take longer than they actually should. The reality is, it's because they were unable to be sent the first time.
