##### This research was done on patch 1.27,

# OM0 in GTA V

Some research on OM0 in GTA V: how it's achieved in game, why OM0 instances "break" and what limitations it has.

## What is OM0 in GTA V

If you are reading this, you are probably familiar with concept of OM0 in 3D-era GTA's and know what ``OnMission`` variable is. So, what's different for GTA V?

First of all, ``OnMission`` is no longer a single variable but 2\*: ``MISSION_TYPE`` and ``MISSION_FLAG``. 

``MISSION_TYPE`` or ``Global_34913`` is an enum (or int value from 0 to 18). Those values are:

```
0 - Some Main Missions (Simeon Missions, Assassinations, Trevor Philips Industries, 
Crystal Maze, Deep Inside, Friedlender etc), House Intro Cutscenes, Main Mission Replays
1 - ??? (Most likely completely unused)
2 - Heist Prep missions
3 - Some Main Missions
4 - Towing, Bail Bond, S&F, Rampages, Hunting
5 - Random Events + Altruist Cult
6 - Friend Activity
7 - Friend Activity
8 - ???    
9 - Taxi, Darts, One Of Carwashes, Races, Shooting Range, Thriathlon, BaseJumping, Yoga, Trafficking, Pilot School, Business Missions, Plane Stunting
10 - Tennis, Golf, Strip Club Private Dance, Booty Call
11 - ??? 
12 - Exile City Denial
13 - Peyote Plants
14 - Director's mode
15 - Free Mode (MISSION_TYPE_OFF_MISSION)
16 - ??? 
17 - Switching characters
18 - Friend knockdown cutscene
```

And ``MISSION_FLAG`` is just a bool value set with native ``void SET_MISSION_FLAG(BOOL toggle);`` and here's some info from [FiveM native reference](https://docs.fivem.net/natives/):

```
If true, the player can't save the game.   
If the parameter is true, sets the mission flag to true, if the parameter is false, the function does nothing at all.  
^ also, if the mission flag is already set, the function does nothing at all  
```

``MISSION_TYPE`` is what we are gonna discuss in this research and ``MISSION_TYPE == 15`` or ``MISSION_TYPE_OFF_MISSION`` is what we call OM0.
What that means is you can't expect the exact same behaviour as in 3D-era GTA's, for example you can't save while OM0 (``MISSION_TYPE_OFF_MISSION``) with ``MISSION_FLAG == true``.

*You could say that there are even more than 2 variables but those 2 are what resemble the old OM variable the most.

## How OM0 is achieved

To get to OM0 state, player has to die or get arrested while starting S&F mission. Let's see why.

First of all, unlike main missions that [start from mission_triggerer scripts](https://github.com/drunderscore/GTA-Research/blob/master/Mission-Triggering-Research-And-Bugs%20.md), S&F missions are triggered in ``launcher_*`` scripts, where ``*`` is the internal name of the specific character's missions.
For this research we'll use our favorite Cletus's mission Target Practice. Internally it's called ``hunting1`` and it is started from ``launcher_hunting``.

Here, we quickly find the part that sets ``MISSION_TYPE`` to ``MISSION_TYPE_OFF_MISSION``.

```
void func_252(var uParam0)//Position - 0xD956
{
	if (*uParam0 == -1)
	{
		return;
	}
	if (!*uParam0 == Global_34875)
	{
		*uParam0 = -1;
		return;
	}
	*uParam0 = -1;
	Global_34874 = 0;
	Global_34876 = 0;
	Global_34913 = 15; //MISSION_TYPE_OFF_MISSION
	Global_54747 = 0;
	Global_54748 = 0;
}
```

If we search for references to this function, we'll only find one and it is right at the beginning of the script:

```
if (PLAYER::HAS_FORCE_CLEANUP_OCCURRED(83))
{
	func_253("Force cleanup [TERMINATING]");
	if (Var0 != -1)
	{
		if (Global_96440[Var0 /*10*/].f_9 != -1)
		{
			func_253("Relinquishing candidate id...");
			func_252(&(Global_96440[Var0 /*10*/].f_9)); //our function
		}
	}
	func_240(&Var0, 1);
}
```

Here we can see that something called ``FORCE_CLEANUP`` is occurred. You could say that ``FORCE_CLEANUP`` is an event sent from other scripts or the executable that signalizes that this script has to clean the things that it created (for example objects, vehicles, screen effects etc) and terminate.
Other scripts call native ``void FORCE_CLEANUP(int cleanupFlags);`` to trigger the cleanup.

As we can see, the script expects the cleanup with ``cleanupFlags == 83``. Cleanup flags are actually a bitfield, that means that we are expecting cleanups ``64``, ``16``, ``2`` and ``1``.
With mods (by calling ``int GET_CAUSE_OF_MOST_RECENT_FORCE_CLEANUP();`` every frame) we can find out that when the player dies or gets arrested ``force_cleanup(1)`` is called (You could also check this in the exe itself probably). 

And that's it, that's how OM0 is achieved.

### Why drowning

As we know, we have to die while starting S&F mission. But you can't just pull out a c4 and blow yourself up. That's because ``launcher_hunting``  sets 2 types of invincibility, total invincibility and specific proofs:

```
entity::set_entity_invincible(player::player_ped_id(), 1);
Sets a ped or an object totally invincible. It doesn't take any kind of damage. Peds will not ragdoll on explosions and the tazer animation won't apply either.

entity::set_entity_proofs(player::player_ped_id(), true, true, true, true, true, false, 0, false);
SET_ENTITY_PROOFS(Entity entity, BOOL bulletProof, BOOL fireProof, BOOL explosionProof, BOOL collisionProof, BOOL meleeProof, BOOL steamProof, BOOL p7, BOOL drownProof);
```

If you've watched GTA V speedruns before runners started to use Director Mode setup, then you know that we used to drown to get OM0. So why that works?

That's because ``hunting1`` script itself will clean the invincibility for us before starting the cutscene. You'll probably expect to find something like ``set_entity_invincible(player::player_ped_id(), 0);`` but in reality there is another way to set\remove the invincibility,
that's ``void SET_PLAYER_CONTROL(Player player, BOOL bHasControl, int flags);`` native. The game is kind enough to make you invincible when it takes away the control from you and makes you vulnerable when it gives control back.

Luckily for us, ``hunting1`` gives us the control back before the cutscene, even though we've never lost it and the native itself makes us vulnerable unconditionally. Now that we are vulnerable, only proofs are left and if we look closely at the ``set_entity_proofs`` call 
we'll see that we are actually not steam or drown proof.

With that knowledge all that's left is to delay the cutscene from playing by interrupting Cletus's dialogue with Trevor's special ability or alt-tapping and drown.

### Why Director Mode

Drowning strat is rather slow because of the slow walk forced by ``hunting1`` and the fact that drowning itself is not fast. What can we do to make the process faster? Well, we still have proofs to deal with. 
If we were to somehow disable both the proofs and the invincibility, we'll be able to blow ourselves up and significantly speed up the process of getting OM0. That's where Director Mode comes in.

``director_mode`` script removes both invincibility and proofs after you close "Director Mode is not available whilst playing a mission." warning. This message occurs when you try to launch DM with ``MISSION_TYPE != MISSION_TYPE_OFF_MISSION`` (+ few other special cases we don't care about here). 

The plan here is simple: start DM just before you trigger S&F mission. Sounds easy but there are 2 problems: 

First problem here is that the game checks ``MISSION_TYPE`` 2 times if you start DM from interactions menu. First, it is checked in ``main`` before launching ``director_mode`` script and the second time during the initialization of ``director_mode`` itself. 
Only ``director_mode`` script removes the proofs, meaning that you have to pass first check before it starts and fail the second one to trigger removal of proofs. This alone requires close to frame perfect inputs from the player.

The second problem is that ``launcher_hunting`` doesn't set proofs on the same frame as it changes ``MISSION_TYPE``. It first sets ``MISSION_TYPE`` and then it waits for ``hunting1`` script to load before setting the proofs. What that means is that we can end up in the situations where we successfully
got the warning screen from ``director_mode`` but still have no proofs set. In this case, ``director_mode`` will remove the proofs only for the ``launcher_hunting`` to set them back in few frames.

These two problems combined make this trick close to frame perfect but with clever setups and references it is still possible to execute it with some consistency.

## Why OM0 "breaks"

In GTA V you can prevent OM0 instance of a mission from displaying mission failed\passed screen and resetting mission checkpoint value, we call this broken OM0. 

The reason for this is this check:

```
if (!Global_96440[iVar0 /*10*/].f_4)
{
    return;
}

Global_96440 is some kind of mission array, probably RC_MISSIONS (RC is Random Character in the scripts or S&F in the game).
iVar0 - current RC mission id.
Not sure what f_4 is but it's set to 1 when mission starts in rc launcher.
```

This check is used every time we need to finish the script (force_cleanup, mission fail or completing the mission).
Normally during OM0 we wouldn't pass this check, the function will continue it's execution and checkpoint will be set to 0/mf will trigger. 
To get the broken state we need to start any script that sets ``MISSION_TYPE`` to anything other than ``15`` or ``17`` and then back to ``15``.
That's because a script called ``randomchar_controller`` terminates when ``MISSION_TYPE != 15 || 17`` and then starts again when opposite condition is met. And when ``randomchar_controller`` starts, it executes this function:

```
void func_130()//Position - 0x9710
{
    int iVar0;
    
    iVar0 = 0;
    while (iVar0 < 63)
    {
        Global_96440[iVar0 /*10*/].f_5 = 0;
        Global_96440[iVar0 /*10*/].f_6 = 0;
        Global_96440[iVar0 /*10*/].f_4 = 0; // f_4 is set to 0
        Global_96440[iVar0 /*10*/].f_7 = 0;
        Global_96440[iVar0 /*10*/].f_8 = -1;
        Global_96440[iVar0 /*10*/].f_9 = -1;
        iVar0++;
    }
}
```
Here it resets all RC missions back to their original states thus breaking OM0.

## Why we can't OM0 everything with OM0 S&F instance

If when we finish the mission normally we end up with ``MISSION_TYPE_OFF_MISSION`` then why can't we start OM0 S&F instane, start main mission and finish S&F instance to get OM0 on main mission?

That's because S&F mission scripts will never set ``MISSION_TYPE`` back to ``15`` after OM0 because there is a check for ``Global_96440[iVar0].f_9`` to not be ``-1``.
``Global_96440[iVar0].f_9`` is the index that shows order in which 'mission' scripts were launched, let's call it ``LAUNCH_MISSION_ID``. For example if we were to start 5 'missions' before starting S&F mission, this variable will be 6. In this context mission is anything that modifies ``MISSION_TYPE``.

Example from ``hunting1``:

```
if (Global_96440[iVar0].f_9 == -1)
{
}
else
{
	func_176(&(Global_96440[iVar0 /*10*/].f_9));
}

void func_176(var uParam0)//Position - 0x1FEB2
{
	if (*uParam0 == -1)
	{
		return;
	}
	if (!*uParam0 == Global_34875)
	{
		*uParam0 = -1;
		return;
	}
	*uParam0 = -1;
	Global_34874 = 0;
	Global_34876 = 0;
	Global_34913 = 15; //MISSION_TYPE_OFF_MISSION
	Global_54747 = 0;
	Global_54748 = 0;
}
```

However, if we'll look back at the ``launcher_hunting`` termination we will see that ``Global_96440[iVar0].f_9`` will be set to ``-1`` on death\arrest.

```
if (PLAYER::HAS_FORCE_CLEANUP_OCCURRED(83))
{
	...
	func_252(&(Global_96440[Var0 /*10*/].f_9)); //here we pass LAUNCH_MISSION_ID
	...
}

void func_252(var uParam0)
{
	if (*uParam0 == -1)
	{
		return;
	}
	if (!*uParam0 == Global_34875)
	{
		*uParam0 = -1;
		return;
	}
	*uParam0 = -1; //LAUNCH_MISSION_ID = -1
	Global_34874 = 0;
	Global_34876 = 0;
	Global_34913 = 15; //MISSION_TYPE_OFF_MISSION
	Global_54747 = 0;
	Global_54748 = 0;
}
```

And this prevents us from setting ``MISSION_TYPE`` to ``MISSION_TYPE_OFF_MISSION``.

## But what if we got OM0 without ``LAUNCH_MISSION_ID = -1``?

Even if we were to somehow bypass this, the game actually has a simple and smart way to check if we can set ``MISSION_TYPE`` to ``MISSION_TYPE_OFF_MISSION``.

Let's once again go back to our favorite ``func_252``:

```
void func_252(var uParam0)
{
	...
	if (!*uParam0 == Global_34875) //if LAUNCH_MISSION_ID != Global_34875
	{
		*uParam0 = -1;
		return;
	}
	Global_34913 = 15; //MISSION_TYPE_OFF_MISSION
	...
}
```

Alright, so we won't get to setting ``MISSION_TYPE`` if ``LAUNCH_MISSION_ID`` is not equal to some ``Global_34875``. Let's find what this global does.

From ``launcher_hunting``:

```
iVar0 = func_141(&(Global_96440[iParam0 /*10*/].f_9), 1, 4, 0, 0); //LAUNCH_MISSION_ID is passed

int func_141(var uParam0, int iParam1, int iParam2, bool bParam3, int iParam4)//Position - 0x94FF
{
	//uParam0 is LAUNCH_MISSION_ID
	...
	if (iParam1 == 0)
	{
		if (func_145(0))
		{
			return 0;
		}
		Global_34877++; //LAST_LAUNCH_ID++
		*uParam0 = Global_34877; //LAUNCH_MISSION_ID = LAST_LAUNCH_ID++
		...
		Global_34913 = iParam2; //MISSION_TYPE to iParam2
		Global_34875 = *uParam0; //Global_34875 = LAUNCH_MISSION_ID
		Global_34876 = iParam4;
		Global_34874 = 0;
		return 1;
	}
	...
}
```

What we see here is the function that changes ``MISSION_TYPE`` to the desired value when we start any 'mission' script. Every time it is called, it increments ``LAST_LAUNCH_ID`` and puts it into ``LAUNCH_MISSION_ID`` for the mission that called it. 
And it also stores ``LAST_LAUNCH_ID`` to ``Global_34875``. I'm not sure why the redundancy but let's assume that ``Global_34875`` is also ``LAST_LAUNCH_ID``.

Now let's combine the information from both ``func_252`` and ``func_141``.

What we get is this: only the last mission that changed ``MISSION_TYPE`` can set it back to ``MISSION_TYPE_OFF_MISSION``. 
That means that if we start main mission while OM0 S&F only main mission will be able to set ``MISSION_TYPE`` back to ``MISSION_TYPE_OFF_MISSION``.

There are some exceptions to this rule however, some scripts set ``MISSION_TYPE_OFF_MISSION`` unconditionally, sometimes explicitly (main for example) and sometimes implicitly by passing ``LAST_LAUNCH_ID`` itself as ``LAUNCH_MISSION_ID``.

Those scripts are:
```
benchmark
candidate_controller
main
mission_repeat_controller
```

It might be possible to exploit this somehow but as of this moment I have no idea how.

## What if we somehow start another mission while ``MISSION_TYPE != MISSION_TYPE_OFF_MISSION``

Will this allow us to OM0 first mission using the second one? No. 

Remember the function that sets ``MISSION_TYPE`` to the desired value? There is a check that will prevent us from doing that in there.

``` 
int func_141(var uParam0, int iParam1, int iParam2, bool bParam3, int iParam4)//Position - 0x94FF
{
	...
	if (iParam1 == 0)
	{
		if (func_145(0)) 
		{
			return 0; // return 0 if can't start MISSION_TYPE 0 from current MISSION_TYPE
		}
		...
		Global_34913 = iParam2; //MISSION_TYPE to iParam2
		return 1;
	}
	...
}

int func_145(int iParam0) //__IS_CURRENTLY_ON_MISSION_TO_TYPE
{
	if (Global_34913 == 15)
	{
		return 0;
	}
	if (func_143(iParam0))
	{
		return 0;
	}
	return 1;
}

bool func_143(int iParam0)//__CAN_MISSION_TYPE_START_AGAINST_CURRENT_TYPE
{
	return func_144(iParam0, Global_34913);//__CAN_MISSION_TYPE_START_AGAINST_TYPE
}

int func_144(int iParam0, int iParam1)//__CAN_MISSION_TYPE_START_AGAINST_TYPE
{
	if (iParam1 == 15)
	{
		return 1; //we can always start from MISSION_TYPE_OFF_MISSION
	}
	if (iParam0 == 15)
	{
		return 0;
	}
	switch (iParam0)
	{
		...
		case 0:
			switch (iParam1)
			{
				case 5:
				case 17:
					return 1; //for MISSION_TYPE 0 we can only start from MISSION_TYPE 5 or 17
					break;
			}
			break;
	}
```

The game checks if we can launch a mission with ``MISSION_TYPE`` ``0`` before setting the ``MISSION_TYPE`` to the value we want and updating ``LAST_LAUNCH_ID``.

As we can see in the code ``MISSION_TYPE`` ``0`` can only start if current ``MISSION_TYPE`` is ``15`` (``MISSION_TYPE_OFF_MISSION``), ``17`` (``MISSION_TYPE_SWITCH``) or ``5`` (``MISSION_TYPE_RANDOM_EVENT``). 

That's it, we can't start other missions while we are not in freemode\switching\have an ongoing random event.

This, of course, also has exceptions... or at least one. ``friendactivity`` unconditionally switches between ``MISSION_TYPE`` ``6`` and ``7`` and scripts that usually set their own ``MISSION_TYPE`` (stripclub, golf, darts) 
skip this step completely by explicitly checking for those ``MISSION_TYPES`` when they start. 

Because of that ``friendactivity`` still has ``LAST_LAUNCH_ID`` and can set ``MISSION_TYPE_OFF_MISSION``. That's why you can [OM0 stripclub](https://youtu.be/SdyrM0q9jfk) script.

There might be other exceptions but so far I haven't found any.

## Why S&F OM0 transfering works?

You can actually 'transfer' OM0 from one S&F mission to another. To do this, you OM0 the first one then trigger the second one, fail the first one and exit the mission. After that the second S&F mission will start OM0-ed.

So, having learned stuff from previous points you might wonder how this is possible. Actually, it's pretty simple, we just skip setting ``MISSION_TYPE`` completely.

Let's see how this works in ``launcher_hunting``:

```
int func_2(int iParam0)//Position - 0x2EA
{
	if (!func_148())
	{
		while (!func_139(*iParam0)) //while (failed to set MISSION_TYPE)
		{
			...
			SYSTEM::WAIT(0);
		}
	}
	...
	//rest of the function, mission script is launched here 
}
````

Alright, what do we see here. ``func_139`` endlessly tries to set ``MISSION_TYPE`` until it succeeds. This means that the only way to skip setting it is failing ``if (!func_148())`` check.

Let's check ``func_148()``:

```
int func_148()//Position - 0x98F3
{
	if (Global_89962 == 10 || Global_89962 == 9)
	{
		return 1;
	}
	return 0;
}
```

What happens here is pretty obvious, ``Global_89962`` should be familiar to us from [mission fail & retry research](https://github.com/drunderscore/GTA-Research/blob/master/Mission-Fail-Retry-Research.md) (actually globals are different there since different game versions were used, I'll probably update old research to use 1.27 sometime in the future). 

``Global_89962`` is ``MISSION_FAILED_STATE`` and we check if we are not in 'after retry' state. Coincidentally, that's exactly the state we have after OM0-ing first S&F mission since we fail the mission to get OM0.

That's all there is to it, setting ``MISSION_TYPE`` is completely skipped, leaving us in ``MISSION_TYPE_OFF_MISSION`` state in the second S&F mission.