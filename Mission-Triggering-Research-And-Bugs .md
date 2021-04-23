##### This research was done on build 944. Not exactly sure which 1.xx version it is.

# Mission Triggering in GTA V

Some reasearch on mission triggering + explanation for 2 bugs related to it.

## Scripts Responsible for Mission Triggering

```
standard_global_init: Initializes `MISSIONS_ARRAY`
flow_controller: Adds missions to `CURRENTLY_AVAILABLE_MISSIONS_ARRAY` + starts mission scripts
mission_triggerer_a-d: Performs the necessary checks for missions to actually start
```

## standard_global_init

`standard_global_init` initializes `Global_82607` (`MISSIONS_ARRAY`) inside `func_63()`. It looks like this:

```
void func_63()//Position - 0x2C04
{
	func_66();
	func_65(66, "agency_heist1", "AH1", 230, 1, 1, -1, -1, 8192);
	func_65(67, "agency_heist2", "AH2", 230, 2, 2, -1, -1, 8);
	func_65(68, "agency_prep1", "AHP1", 231, 3, 7, -1, -1, 108608);
	func_65(69, "agency_heist3A", "AH3a", 232, 1, 1, 20, 21, 134242560);
	func_65(70, "agency_heist3B", "AH3b", 233, 3, 1, 0, 3, 134242304);
	....
}
```

Let's look at `func_65()`:

```
void func_65(int iParam0, char* sParam1, char* sParam2, int iParam3, int iParam4, int iParam5, int iParam6, int iParam7, int iParam8)//Position - 0x34D7
{
	StringCopy(&(Global_82607[iParam0 /*34*/]), sParam1, 24); //script name
	Global_82607[iParam0 /*34*/].f_6 = gameplay::get_hash_key(sParam1); //hash_key of script name
	StringCopy(&(Global_82607[iParam0 /*34*/].f_8), sParam2, 8); //mission name in global.gxt
	Global_82607[iParam0 /*34*/].f_11 = iParam4; //characters that can start this mission
	Global_82607[iParam0 /*34*/].f_12 = iParam5; //unknown
	Global_82607[iParam0 /*34*/].f_10 = iParam3; //unknown
	Global_82607[iParam0 /*34*/].f_13 = iParam6; //mission timeframe start (-1 if none)
	Global_82607[iParam0 /*34*/].f_14 = iParam7; //mission timeframe end (-1 if none)
	Global_82607[iParam0 /*34*/].f_15 = iParam8; //some kind of bitfield (for example contains field for `MISSION_TYPE` which dictates what is allowed during the mission (using interior garages, etc))
}
```

`MISSIONS_ARRAY` is used by various scripts to perform mission related operations.

## flow_controller

`flow_controller` adds new missions to `Global_88752` (`CURRENTLY_AVAILABLE_MISSIONS_ARRAY`) inside `func_619()`:

```
void func_619(var uParam0, int iParam1, var uParam2, var uParam3, var uParam4)//Position - 0x756B2
{
	//this function sets MISSION_CAN_BE_STARTED (Global_88752[iVar0]) flag for mission_triggerers
	
	//iParam1 - mission id
	//uParam4 - characters that can start that mission
	int iVar0;
	int iVar1;
	
	iVar0 = -1; //last mission with MISSION_CAN_BE_STARTED == 0
	iVar1 = 0;
	iVar1 = 0;

	//Global_88751 - Global_88752 size/length
	while (iVar1 < Global_88751) //looking for entry with MISSION_CAN_BE_STARTED == 0
	{
		if (Global_88752[iVar1 /*17*/] == 0)
		{
			iVar0 = iVar1; //entry found
		}
		iVar1++;
	}

	if (iVar0 == -1) //no missions with MISSION_CAN_BE_STARTED == 0
	{
		iVar0 = Global_88751;
		Global_88751++; //size++, will create new item in CURRENTLY_AVAILABLE_MISSIONS_ARRAY
		if (Global_88751 > 8)
		{
		}
	}
	Global_88752[iVar0 /*17*/] = 1; //MISSION_CAN_BE_STARTED = 1
	Global_88752[iVar0 /*17*/].f_1 = 0;
	Global_88752[iVar0 /*17*/].f_6 = uParam2;
	Global_88752[iVar0 /*17*/].f_4 = uParam3;
	Global_88752[iVar0 /*17*/].f_7 = -1;
	Global_88752[iVar0 /*17*/].f_5 = iParam1; //mission id
	Global_88752[iVar0 /*17*/].f_8 = uParam4; //characters that can start that mission
	if (!gameplay::is_bit_set(Global_82607[iParam1 /*34*/].f_15, 27))
	{
		func_285(iParam1, 1);
	}
	*uParam0 = iVar0;
}
```

## mission_triggerer_a-d

Now that we have `CURRENTLY_AVAILABLE_MISSIONS_ARRAY` with some missions that have `MISSION_CAN_BE_STARTED` set to 1, we need to actually check for specific conditions that will trigger mission execution.
Thats where mission_triggerers come to play.

Inside any mission_triggerer's main loop (here `mission_triggerer_b` is used) you'll find `func_1()` which will look something like this:

```
if (Global_101652.f_8028) //CAN_START_MISSIONS (false for directors mode and benchmark)
	{
		if (!Global_88749) //if(!SOME_MISSION_PASSED_THE_CHECKS)
		{
			if (!entity::is_entity_dead(player::player_ped_id(), 0))
			{
				iVar0 = 0;
				while (iVar0 < Global_88751)
				{
					Global_88740 = Global_88752[iVar0 /*17*/].f_5;
					if (Global_88752[iVar0 /*17*/] == 1 && !Global_88749) //if(MISSION_CAN_BE_STARTED && !SOME_MISSION_PASSED_THE_CHECKS)
					{
						...
						else
						{
							switch (Global_88752[iVar0 /*17*/].f_3)
							{
								case 0:
									func_26(iVar0, uParam0); //checks for starting missions are here
									break;
								
								case 1:
									func_19(iVar0, uParam0); //probably missions that start from character switches
									break;
								
								case 2:
									break;
								}
							}
					}
					iVar0++;
				}
				if (Global_88743 != -1)
				{
					//some specific missions (mrsphilips2, martin1, family6, finale_endgame)
					...
				}
			}
		}
		else if (Global_88749 == 1) // if(SOME_MISSION_PASSED_THE_CHECKS)
		{
			//sets some variables + switches Global_88749 back to 0
			...
		}
		...
	}
```

What happens here is: for every mission in `CURRENTLY_AVAILABLE_MISSIONS_ARRAY` with `MISSION_CAN_BE_STARTED == 1` mission_triggerer launches `func_26()` which looks like this:

```
void func_26(int iParam0, var uParam1)//Position - 0x12D8
{
	...
	iVar0 = Global_88752[iParam0 /*17*/].f_5; //mission id
	...
	if (func_156(iVar0)) //some missions exclusion check
	{
		if (!gameplay::is_bit_set(Global_88752[iParam0 /*17*/].f_10.f_1, 0))
		{
			Stack.Push(iVar0); //mission id
			Stack.Push(&(Global_88752[iParam0 /*17*/].f_10)); //some bitfields, dunno
			Stack.Push(&(uParam1->f_1[iParam0 /*13*/])); //callbacks stack
			Call_Loc(*uParam1); //func_165 call from stack
			func_155(&(Global_88752[iParam0 /*17*/].f_10));
		}
		func_150(iVar0, vVar2, &(uParam1->f_113));
		if (!func_27(iParam0, uParam1)) //main check that decides if mission should start
		{
			return; //if conditions to start doesn't met just return
		}
	}
	Global_88750 = iParam0; //array index for mission to start
	Global_88749 = 1; //SOME_MISSION_PASSED_THE_CHECKS = 1
}
```

`func_27()` is really big and probably deserves it's own write up. All we need to know for now is that it executes the necessary 
pre-mission events (spawning cars, peds, starting pre-mission dialogs, etc...) and returns true if all conditions for mission starting were met.
Those pre-mission events are set inside `func_165()`. It's called right before `func_27()` and looks like this (this one is for "stingers" mission):

```
void func_165(int iParam0, var uParam1, var uParam2)//Position - 0x7D60
{
	switch (iParam0) //switch (mission id)
	{
		case 78: //stingers
			func_984(uParam1, iParam0, 140f, 150f, 50f, 7, 500f, 0); 
			uParam2->f_1 = 447675/*func_726*/; //empty
			uParam2->f_2 = 447640/*func_725*/; //request models
			uParam2->f_3 = 447605/*func_724*/; //set models as no longer needed
			uParam2->f_4 = 447538/*func_723*/; //checks if models are loaded
			uParam2->f_5 = 446434/*func_720*/; //creates everything for mission scene in freemode
			uParam2->f_6 = 446353/*func_719*/; //vehicles and peds configuration
			uParam2->f_7 = 446272/*func_718*/; //deleting vehicles and peds
			uParam2->f_10 = 445987/*func_715*/; //checks if player's close enough + creates a blip
			uParam2->f_11 = 445944/*func_714*/; //checks if policet is alive
			uParam2->f_12 = 445935/*func_713*/; //always false
			uParam2->f_8 = 445927/*func_712*/; //empty
			uParam2->f_9 = 445919/*func_711*/; //empty
			break;
			
		...	
	}
}
```

Those callbacks are different for every mission. Mission ids come from `standard_global_init`, so you can check those there to determine which mission is which.

## Starting mission scripts

Finally, after all the checks are done, the game needs to actually start the mission script. Here's what happens for missions that start with cutscenes:

For those missions, callback `uParam2->f_5` from `func_165()` will call a function which looks like this:

```
void func_206(int iParam0, char* sParam1, int iParam2, int iParam3, int iParam4)//Position - 0x929D
{
	//sets variables for mission script to start (flow_controller func_957())

	// int iParam0 - mission id
	// char* sParam1 - cutscene name
	// int iParam2, int iParam3, int iParam4 - cutscene scenes depending on character (-1 if player can't start this mission)

	if (Global_101652.f_8028)
	{
		if (!(Global_69700 && Global_69702 == iParam0))
		{
			Global_69702 = iParam0; //mission id
			StringCopy(&Global_69703, sParam1, 24);
			Global_69709[0] = iParam2; //player0 start
			Global_69709[1] = iParam3; //player1 start
			Global_69709[2] = iParam4; //player2 start
			Global_69700 = 1; //start mission script = 1
			Global_69701 = 0; //not really sure
			func_207();
		}
	}
}
```

`Global_69702` - mission id for which the script should be started
`Global_69700` - SHOULD_START_MISSION_SCRIPT

All that's left is to request and start the script, `flow_controller` takes care of requesting scripts with `func_957()`:

```
	if (Global_69700)
	{
		if (!gameplay::are_strings_equal(&Global_69703, "NONE"))
		{
			if (Global_69713 == -1)
			{
				func_962(&Global_69713, 2);
			}
			else
			{
				switch (func_961(Global_69713))
				{
					case 1:
						if (!cutscene::is_cutscene_active())
						{
							iVar0 = func_189(); //gets current character
							if (func_22(iVar0)) 
							{
								if (Global_69709[iVar0] != -1) //if current character can start that mission
								{
									script::request_script_with_name_hash(Global_82607[Global_69702 /*34*/].f_6); //requesting mission script
									...
								}
							}
						}
						...
					}
				}
			}
		}
	}
```

Now we need to start the script and... as of now I've yet to find where mission scripts are actually started after being requested.

## Bug 1 "Invincibility + No Wanted level"

[Video](https://youtu.be/BrZOtY-xAI0)

The game doesn't want you to die during mission triggerring, for this reason some of the callbacks from `func_165()` will call a function that will make you invulnerable.
Usually this function is called through intermediate function that also sets invulnerabilities to mission vehicles\peds\etc and sets `Global_88739` to 1.

Let' look at the code:

```
void func_210()//Position - 0xBD9C
{
	int iVar0;
	int iVar1;
	
	if (!Global_88739)
	{
		... //sets invulnerabilities for mission vehicles, peds, etc 
		func_211(1); //sets player's invulnerabilities
		player::force_cleanup(8);
		Global_88739 = 1;
	}
}

void func_211(int iParam0)//Position - 0xBF9A
{
	//set player proofs
	if (!entity::is_entity_dead(player::player_ped_id(), 0))
	{
		entity::set_entity_proofs(player::player_ped_id(), true, true, true, true, true, false, 0, false);
		ped::set_ped_config_flag(player::player_ped_id(), 157, false);
		entity::set_entity_invincible(player::player_ped_id(), true);
		ped::set_ped_stealth_movement(player::player_ped_id(), 0, 0);
		if (iParam0)
		{
			weapon::set_current_ped_weapon(player::player_ped_id(), joaat("weapon_unarmed"), false);
		}
		ped::remove_ped_helmet(player::player_ped_id(), 0);
	}
	if (player::is_player_playing(player::player_id()))
	{
		player::clear_player_wanted_level(player::player_id());
	}
	player::set_max_wanted_level(0);
	player::set_wanted_level_multiplier(0f);
}
```

Now let's look how the game strips those invulnerabilities:

```
	if (Global_88739)
	{
		func_53(); // reverses func_210() and func_211()
	}
```


As you can see, `Global_88739` is crucial here because it decides if we should revert invulnerabilities that we've set before. I guess you could call this variable `IS_CLEANUP_NEEDED`.

Finally, let's see how "The Third Way" sets invulnerabilities:

```
if (func_743(&Local_1252) && interior::is_valid_interior(interior::get_interior_from_entity(player::player_ped_id())))
{
	func_211(0); //sets player's proofs directly
	func_732(75, 1);
	object::_set_door_ajar_angle(-565026078, 0f, 1, 0);
	...
}
```

Here's our answer, for some reason 3-rd way calls `func_211()` directly, skipping `func_210` and as a result doesn't set `IS_CLEANUP_NEEDED` to 1 preventing the script from properly removing player's proofs.
In fact, that's the only mission that does this, curious.

## Bug 2 "Wrong Mission Start\Fail"

[Video](https://youtu.be/fCHfJNBCwQE)

As we can see in the video, after starting The Merryweather Heist in an unusual way, mission failed screen for Towtruck appears for some reason.

A quick search for mission fail reason string shows that fail was triggered from `fbi4_prep2` script which is the script for the towtruck mission. 
This means that the script was indeed started and that checks inside `func_27()` from `mission_triggerer_b` were satisfied, strange.

Common sense tells us that there are usually only two ways to start that mission: either find a random towtruck in traffic or simply drive to mission location on the map.
Let's check mission id for towtruck mission in `standard_global_init`, that would be 34.  Now let's head back to callbacks from `func_165()` and check those for this mission. 

Here, we'll find callback f_10 which looks like this:

```
int func_851()//Position - 0x7E4AA
{
	vector3 vVar0;
	float fVar3;
	
	vVar0 = { func_148(191, 0) };
	if (!ped::is_ped_injured(player::player_ped_id()))
	{
		fVar3 = system::vdist2(entity::get_entity_coords(player::player_ped_id(), 1), vVar0);
		if (fVar3 < (75f * 75f)) \\are we close enough to mission?
		{
			func_854();
			iLocal_1114 = 1;
			if (!ui::does_blip_exist(iLocal_1113))
			{
				iLocal_1113 = func_852(Global_88316[0], 0, 0);
			}
			return true;
		}
		else if (ped::is_ped_in_any_vehicle(player::player_ped_id(), 0)) 
		{
			func_854();
			iLocal_1114 = 1;
			return vehicle::is_vehicle_model(ped::get_vehicle_ped_is_in(player::player_ped_id(), 0), joaat("towtruck")); \\are we driving a towtruck?
		}
	}
	return false;
}
```

Exactly what we were hoping for. But there are indeed only 2 conditions to start that mission and in the video we clearly never satisfied them.

Let's look how this callback is called:

```
bool func_125(var uParam0, var uParam1)//Position - 0x6441
{
	int iVar0;
	int iVar1;
	int iVar2;
	
	iVar0 = Global_88752[uParam0 /*17*/].f_5; //mission id
	if (gameplay::is_bit_set(Global_88752[uParam0 /*17*/].f_8, func_20())) //checks if current character can start this mission
	{
		...
		if (!gameplay::is_bit_set(Global_82607[iVar0 /*34*/].f_15, 22))
		{
			if (Global_88739) //IS_CLEANUP_NEEDED
			{
				if (uParam1->f_111 == -1) //those 2 checks are the reason why you have to do it 2 times
				{
					uParam1->f_111 = gameplay::get_game_timer();
				}
				else if ((gameplay::get_game_timer() - uParam1->f_111) > 12000) //if 12 seconds have passed
				{
					iVar2 = 1;
				}
			}
		}
		if (cam::is_screen_faded_in())
		{
			Call_Loc(uParam1->f_1[uParam0 /*13*/].f_10); //call to callback for start condition check
			if (StackVal || iVar2) //if callback returned true or 12 seconds have passed
			{
				if (func_127(iVar1)) //requirements to start like not getting arrested etc
				{
					if (gameplay::is_bit_set(Global_88752[uParam0 /*17*/].f_10.f_1, 4) || Global_88741 == iVar0)
					{
						...
						return true;
					}
				}
			}
		}
	}
	return false;
}
```

And `func_125` in turn is called from `func_27()` like this:

```
	if (func_125(uParam0, uParam1))
	{
		return true;
	}
```

You'll be surprised but once again the culprit behind this bug is `Global_88739` (`IS_CLEANUP_NEEDED`). As you can see from the code above, there is actually a third way to start the mission, a very weird one.
The first time you'll try to start The Merryweather Heist the way shown in the video, `IS_CLEANUP_NEEDED` will be set to 1 thus saving current game time to `uParam1->f_111` since it is not initialized yet (== -1). 
The second time you'll start the mission, `IS_CLEANUP_NEEDED` will be set to 1 once again while `uParam1->f_111` will already have the game time we saved last time, so all that's left is to check if 12 seconds have passed.