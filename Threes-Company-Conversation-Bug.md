##### This research was done on patch 1.27, ScriptHook gameversion 4.

# Three's Company Conversation Bug

## The bug in question

https://www.twitch.tv/videos/521090595?t=2h11m47s (orignial VOD, DarkViperAU, thanks to Reloe)

Here are clips from the VOD above:

https://clips.twitch.tv/PeppyBillowingSrirachaRitzMitz (Driving to the mission, first recognition)

https://clips.twitch.tv/HyperResilientStingrayDancingBanana (IAA building cutscene conversation lost, allowing Michael to instantly enter)

##### There exist more instances, this is not an isolated case. I couldn't find other VODs or make the clips, however.

# Technical Details

The script for this mission is called `fbi2`. Is also seems that the script `player_scene_m_fbi2` is used for the scene where Trevor is harrasing the FBI agent. Seems it spawns the helicopter/franklin's bike/FBI vehicle itself?

## Cutscenes

Cutscene `fbi_2_mcs_1` is the one that plays the overlapping/early conversation audio.

## Animations Used

In dictionary `missfbi2@leadinoutfbi_2_mcs_1`:

```
_leadin_action_trevor: For Trevor, when harrasing the FBI agent.
```

## Dave Ped (0x15E94)

Dave's ped handle seems to be shared using the shared ped global, described in `Mission-Research.md`.

## Shared Globals (0x15D93)

This global (and it's members) are specifically used in this script, and in `player_scene_m_fbi2`, to have shared data between the two. This includes vehicles, peds, blips, and states. It seems like most of these are initialized by `player_scene_m_fbi2`, and some by `fbi2`. Below is a list of each global and what it should contain.

```
0x15D93: Blip.
0x15D93 + 0x1: Int. Have some blips been created yet?
0x15D93 + 0x2: Vehicle (hash 'frogger')
0x15D93 + 0x3: Vehicle.
0x15D93 + 0x4: Vehicle (hash 'utillitruck3')
0x15D93 + 0x5: Vehicle (hash 'utillitruck2')
0x15D93 + 0x6: Vehicle (hash 'fbi2')
0x15D93 + 0x9: Ped (hash 'cs_fbisuit_01'). FBI agent Trevor harasses?
0x15D93 + 0xA: Int. Have the shared entities been created yet?
```
