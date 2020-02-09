##### This research was done on patch 1.27, ScriptHook gameversion 4.

# Missions in GTA V

**Note:** Missions in GTA V are controlled by many different scripts working together. This allows for seemless starting, ending, skipping, exiting, and failing of missions in the game. The process the game goes through to do this is not simple.

Scripts to look in are mission scripts, `flow_controller`, `mission_repeat_controller`, `replay_controller`, launcher scripts, and many many more.

---

## Mission State Global (0x15F6A)

This global is used to store the state of the current mission. Used very often.

```
0: set on mission fail
5: used when drawing mission fail scaleform for skip
1: set on mission fail, once fail screen as fully faded in
13: normal operation/default?
```

## Misison Fail String (0x10B7D)

This global is updated with the GXT entry of why the mission failed, shown on the fail screen.

## Unknown Mission Global (0x15F8E)

This global is set to `0` upon mission fail.

Seems to be an internal value used for `flow_controller` and `mission_repeat_controller`.

## Shared Ped Global (0x15E94)

This seems to be used to share a ped handle between a mission script and timetable script?

## Mission Failed Scaleform (0x15F7A)

Contains the handle of the mission failed scaleform shown on mission fail.(`MP_BIG_MESSAGE_FREEMODE`)

## Mission Failed Scaleform Flags (0x15F7E)

Contains flags for the mission fail and instructional button scaleforms.

```
Bit 7: Has big message scaleform been initialized yet. (has SHOW_SHARD_CENTERED_MP_MESSAGE_LARGE been called)
Bit 16: Has timescale been slowed. (timescale will be set to either .20f or .08f)
Bit 20: Has TRANSITION_UP been called.
Bit 22: Has big message scaleform message been set.
```

## Mission Failed Instructional Buttons (0x15F7B)

Contains the handle of the instructional buttons scaleform for mission fail. (`INSTRUCTIONAL_BUTTONS`)

## Mission Failed Sound ID (0x15F7C)

## Was Skipped (0x14A40)

This global is set to 1 if the mission/secion was skipped.
