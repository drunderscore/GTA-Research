##### This research was done on patch 1.48, ScriptHook gameversion 50.

# Conversations in GTA V

## Phone Numbers Array Global (0x75)

This global contains element of size 10. The objects structure is seen below:

```
Offset 0: String, usually "NO_ANSMSG"
Offset 4: String, GXT entry for a phone number.
```

## Last Called Number (0x6CA)

This contains an index into the phone numbers array for the number which was last called.

This will update once the call is placed, and not change until another call by the player.

## Last Successful Called Number (0x6CA)

This contains an index into the phone numbers array for the number which was last called.

This will only update if the call was successful (aka, if they would pick up).

## Phone Status (0x977)

The global `0x977` is a bitfield of what is happening with the players phone. Below is a rudimentary outline of what some of the bits I have researched are related to.
This is a 32-bit integer.

```
bit 6: set when in sleep mode
bit 25: set when in sleep mode
bit 13: in cutscene?
bit 14: phone is out
bit 17: phone has certain apps open (contacts, photos)
bit 18: photo is ready?
bit 21: just took picture
bit 23: set for one frame when opening any app
bit 27: on call (outgoing)?
bit 30: set after i fail a mission?
```

## Conversation Queue Global (0x3DFA)

_This name comes from debug strings found within the scripts_

This can take on multiple values to tell what is happening in the current conversation.

```
0: No conversation queued.
1: play line? queued? repeats the last.
2: cancel current line?
3: Waiting (seen when calling during ringtone)
4: Ready/talking (when near Simeon's during Repo or talking on phone)
```

## Unknown Conversation Global (0x3DFC)

```
0: no conversation?
7: When near Simeon's during Repo. (once the cutscene is entered, it goes back to 0). set to 7 at seemingly random points during convo
```

## Last Successful Called Number Name (0x3E40)

Contains the last successfully called number name. Probably has to do with the conversation part of the call. Maybe audio?

## Current Phone Call Data (0x3BD8)

Contains info about the call ongoing or that just happened last. See below.

## Phone Call Conversation Structure

Size unknown.

```
Offset 0: int, usually 16.
Offset 2: String. first member of call? (FRANKLIN, or TaxiDispatch)
Offset 12: String. second member of call? sometimes FRANKLIN
Offset 13: string
Offset 22: string
```

## Pause Next Conversation Line (0x41ED)

While this is one, the next line in a conversation will not play until this is zero.
