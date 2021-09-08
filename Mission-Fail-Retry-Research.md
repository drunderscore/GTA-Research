##### This research was done on build 944. Not exactly sure which 1.xx version it is.

# Mission Failing and Retrying in GTA V

Some research on mission failing (MF from now on), retrying and checkpoints (CP) to understand general workflow of the game.

## Scripts Responsible for MF

```
main: Launches replay_controller.
replay_controller: Displays MF screen, changes the MF state depending on player's input, restarts S&F missions.
flow_controller: Restarts, saves checkpoint on MF and triggers replay_controller for main missions. 
Both S&F and main mission scripts: Change checkpoint, change mission state depending on the checkpoint on restart.
S&F scripts: Save current script name for restarting, save checkpoint on MF, triggers replay_controller.
```

## replay_controller launch

Logic here is pretty straightforward, main.c runs a loop that checks if it should launch other scripts depending on some conditions.

```
void func_1555(var uParam0)//Position - 0xA6660
{
	if (uParam0->f_6)
	{
		switch (*uParam0)
		{
			case 0:
				func_1619(); //mission triggerers
				func_1618(); //player_controller
				func_1617(); //pi_menu interactions menu
				func_1616(); //country_race_controller (stock cars dlc)
				func_1614(); //friends_controller
				func_1613(); //ambientblimp
				break;
			
			case 1:
				func_1612(); //autosave_controller
				func_1610(uParam0); //wardrobe_sp
				func_1609(); //gauntlets
				func_1608(); //launch mission replay
				func_1607(); //animal_controller
				break;
			
			case 2:
				func_1606(); //flow_help
				func_1605(); //comms_controller
				func_1604(); //mission failed screen (replay_controller), launches when game state changes to some mission state
				func_1603(); //launching save_anywhere
				func_1602(); //bootycallhandler
				break;
			
			case 3:
				func_1600(); //ambient_ufos
				func_1599(); //randomchar_controller
				func_1598(); //controller_races
				func_1597(); //ambient_solomon
				func_1594(); //photographywildlife
				func_1593(); //photographymonkey
				func_1592(); //cablecar
				func_1575(); //launchers for darts, bj etc
				break;
		}
		*uParam0++;
		if (*uParam0 > 3)
		{
			*uParam0 = 0;
		}
		func_1574(); //code_controller
		func_1572(); //respawn_controller death/arrest
		func_1556(uParam0); //launching directors mode
	}
	else if (!ped::is_ped_injured(player::player_ped_id()))
	{
		uParam0->f_6 = 1;
	}
} 
```

One of those scripts is `replay_controller`. `func_1604` is responsible for checking if `replay_controller` should be started and starting it if necessary conditions are met. Let's look how it does that:

```
void func_1604()//Position - 0xA87F9
{
	if (((((func_26(0) || func_26(3)) || func_26(2)) || func_26(4)) || func_26(9)) || func_26(10))
	{
		if (script::_get_number_of_instances_of_streamed_script(joaat("replay_controller")) == 0)
		{
			func_1558(joaat("replay_controller"), 1424);
		}
	}
}
``` 

Let's check `func_26()`:

```
int func_26(int iParam0)//Position - 0xB7C
{
	return Global_35777 == iParam0;
}
```

As I said, everything is pretty straightforward, we check if `Global_35777` or `GAME_STATE` is one of the mission states that require MF screen + check if `replay_controller` is not running already. If those conditions are satisfied, we launch `replay_controller`.

## Checkpoints 

Now that `replay_controller` is running, we need to fail the mission to trigger it. But before we look at how it triggers, let's first look at how cps work since we would want to save current CP when we MF.

Once again, nothing extraordinary, we just look for native `playstats_mission_checkpoint` in our mission script. Most likely we'll find this piece of code (`rural_bank_setup` or Paleto Score Setup will be used here):

```
void func_736(int iParam0, char* sParam1, int iParam2, int iParam3, int iParam4, int iParam5)//Position - 0x7E55D
{
	iParam0 - new CP,
	sParam1 - CP description,
	iParam2 - final checkpoint,
	iParam3 - ignore if new CP < current CP,
	iParam4 - character, 0 - current character, other values - specific character,
	iParam5 - smth regarding current vehicle, usually 1
	
	int iVar0;
	int iVar1;
	int iVar2;
	char[] cVar3[8];
	int iVar5;
	var uVar6;
	int iVar10;
	
	...
	iVar0 = 0;	//should change current CP
	if (iParam3 == 1) //if (ignore if new CP < current CP)
	{
		if (iParam0 != Global_91523)
		{
			iVar0 = 1;
		}
	}
	else if (iParam0 > Global_91523) // if(new CP > cur CP)
	{
		iVar0 = 1;
	}
	if (iVar0 == 1) //if (should change current CP)
	{
		func_51(1);
		...
		iVar1 = func_768(script::get_this_script_name(), 1);
		if (iVar1 != -1 && iVar1 != 94) //if (main mission && not mission 94 (mission 94 doesn't exist, useless check))
		{
			Global_101652.f_8028.f_330[iVar1 /*6*/].f_1 = 0;
			iVar2 = func_766(iVar1);
			cVar3 = { Global_82607[iVar1 /*34*/].f_8 };
			if (iVar1 == 90) //jewelry_heist
			{
				switch (Global_101652.f_8028.f_99.f_205[7])
				{
					case 1:
						StringConCat(&cVar3, "A", 8);
						break;
					
					case 2:
						StringConCat(&cVar3, "B", 8);
						break;
					}
			}
			stats::playstats_mission_checkpoint(&cVar3, iVar2, Global_91523, iParam0); //checkpoint
		}
		else
		{
			iVar5 = func_761(script::get_this_script_name(), 1);
			if (iVar5 != -1) //S&F
			{
				Global_101652.f_17517[iVar5 /*6*/].f_4 = 0;
				MemCopy(&uVar6, {func_760(iVar5)}, 4);
				stats::playstats_mission_checkpoint(&uVar6, 0, Global_91523, iParam0);  //checkpoint
			}
			else
			{
				iVar10 = func_759(&(Global_91486.f_3)); //BailBond 1-4
				if (iVar10 > -1)
				{
					Global_101652.f_23929.f_4[iVar10] = 0;
				}
			}
		}
		Global_85997 = iParam2; //saving isFinalCheckpoint
		Global_91523 = iParam0; //saving current CP
		func_737(iParam0, sParam1, iParam4, iParam5);
		...
	}
	...
}
```

Usually this function would be called like this:

```
func_736(1, "stage_drive_to_bank", 0, 0, 0, 1);
```

Now we know that `Global_91523` is `CURRENT_CHECKPOINT` and `Global_85997` is `IS_FINAL_CHECKPOINT` (the latter one isn't really interesting and is used to determine whether the game should display "Skip Mission" or "Skip Section" on MF screens).

With that part out of the way, let's look how MF screen is triggered.

## Mission Failing

First, let's look at `replay_controller` main loop, relevant part looks like this:

```
switch (Global_91486)
{
	case 12:
		Global_91486 = 13;
		break;
	
	case 0:
		func_734();
		break;
	
	case 1:
		func_733();
		break;
	
	case 2:
		func_732();
		break;

	...//other cases
	}
}
```

`replay_controller` checks `Global_91486` or `MISSION_FAILED_STATE` to determine what it should do.

We can quickly check that after failing the mission this Global is equal to 0.

### Script Workflows
So, let's check what sets `MISSION_FAILED_STATE` to 0. Here main missions and S&F go separate ways. Logic is mostly the same but for S&F missions MF is triggered from the script itself while for main missions it's triggered from `flow_controller`.

For main missions algorithm's like this:

```
1) Mission Fail
2) Change some state varialbles
3) Cleanup and Terminate the script
4) flow_controller restarts the script
```

And for S&F missions algorithm's like this:

```
1) Mission Fail
2) Change some state varialbles
3) Wait for MISSION_FAILED_STATE to become 7 or 8
4) Cleanup and Terminate the script
5) replay_controller restarts the script
```

Other than different work flows both save the same information.

Let's look at `flow_controller`. The logic here happens in one of the functions that were discusses in mission triggering research: `Flow_Do_Mission_Now`, `Flow_Do_Mission_At_Blip` or `Flow_Do_Mission_At_Switch`.

At the end of each of those functions we can find a call to `func_238` which calls other relevant functions like saving stats. Among those calls we can find `func_241` that in turn calls `func_242` which among other things does this:

```
...
Global_91486 = 0;
StringCopy(&(Global_91486.f_3), sParam0, 32);
Global_91486.f_11 = iParam1;
Global_91486.f_2 = Global_91523;
...
```

Here, we:
1) Set `MISSION_FAILED_STATE` to 0 triggering `replay_controller`
2) Save current mission script name to `Global_91486.f_3` or `MISSION_FAILED_SCRIPT_NAME`
3) Save mission fail type to `Global_91486.f_11` or `MISSION_FAIL_TYPE`
	Mission Fail types are:
	```
	0-2 - Most Main Missions, depends on bitfield f_15 in MISSIONS_ARRAY 
	3 - carsteal4, fbi3
	4 - me_amanda1, me_jimmy1, me_tracey1
	5 - Trafficking and Towing missions
	6 - S&F
	```
4) Save `CURRENT_CHECKPOINT` to `Global_91486.f_2` or `MISSION_FAILED_CHECKPOINT`

### replay_controller

Now with `replay_controller` triggered let's look at MF states:

```
0 - MF before MF screen appears
1 - MF screen appeared, Main State
2 - MF screen appeared, CP > 0 with Tab to restart the mission, Special Main State (most likely only for Prologue, F&L, Jewelry Store Heist, Mr. Philips)
3 - Pressed Restart on state 2, Confirm Restart state
4 - Pressed Exit Mission, Confirm Exit state
5 - Pressed Skip, Confirm Skip state
6 - Main State for main missions with fail type == 2. Not sure what it's for and the code looks weird (presumably shows no message and just one button OK which exits the mission), is it failsafe\unused code of some kind?
7 - Pressed Retry\Restart\Yes on Confirm Skip\
8 - Pressed Yes on Confirm Exit screen
9 - State finished setup for restarting mission after Retry\Restart\Skip
10 - Finished Retry\Restart\Skip
11 - State after Retry\Restart\Skip for special missions: jewelry_prep2A, jewelry_prep1B, fbi4_prep1, agency_prep1, rural_bank_prep1 + State after Exit pressed during Mission Replay
12 - State after replay_controller termination 
13 - Default state after starting replay_controller + state to terminate replay_controller after exiting the mission.
```

Since we are interested in retrying the mission, we will look at state 9 which is represented by `func_19`:

```
void func_19()//Position - 0x27CF
{
	...
	if (func_18()) //if (MAIN_MISSION)
	{
		func_230(); //sets some bit for main missions and performs some stuff for few special missions
	}
	else
	{
		switch (Global_91486.f_11) //MISSION_FAIL_TYPE
		{
			case 5:
				func_229(); //stuff for towing and trafficking
				break;
			
			case 6:
				func_63(); //relaunch scripts for S&F
				break;
			
			default:
				break;
			}
	}
	if (func_257() == 0) //should always be true for S&F and most main missions (see state 11 for list of exceptions)
	{
		if (func_62() != 0) //if checkpoint != 0
		{
			func_61();
		}
		func_60();
		func_59();
	}
	func_20();
	system::wait(0);
	if (func_257() == 0) //should always be true for S&F and most main missions (see state 11 for list of exceptions)
	{
		Global_91486 = 10;
	}
	else
	{
		system::wait(500);
		cam::do_screen_fade_in(800);
	}
}
```

That's it, we check `MISSION_FAIL_TYPE` to determine if we should restart S&F script using `MISSION_FAILED_SCRIPT_NAME` or change bitfield for Main Missions and proceed to state 10. `replay_controller` will stay in this state unless you MF again.

## Restoring checkpoints 

Finally, we need to restore the state of the mission depending on the checkpoint we saved. Both S&F and Main Missions do this the same way. Let's search for `Global_91486.f_2` or `MISSION_FAILED_CHECKPOINT` in `rural_bank_setup`.
We quickly come up with this function:

```
int func_779()//Position - 0x83431
{
	if (!Global_91486 == 10 && !Global_91486 == 9) //MF state
	{
		return 0;
	}
	return Global_91486.f_2; // MF checkpoint
}
```

Basically we check if we came from MF screen and if we did - return `MISSION_FAILED_CHECKPOINT`.

Now let's see how it's used.

```
iVar0 = func_779(); //checkpoint 
if (Global_85996 == 1)
{
	iVar0++;
}
if (iVar0 > 3) //if checkpoint > 3 start final cutscene
{
	if (func_178() != 0)
	{
		while (!func_414(0, 1))
		{
			system::wait(0);
		}
	}
	func_784(38, 1);
	func_784(37, 0);
	func_778(1393.542f, 3580.138f, 33.9722f, 353.4546f, 1, 0);
	func_722(0, -1, 1);
	func_187();
}
else
{
	switch (iVar0) //change mission state depending on checkpoint
	{
		case 0:
			vLocal_196 = { vLocal_127 };
			fLocal_199 = fLocal_130;
			gameplay::clear_area(vLocal_108, 500f, 1, 0, 0, false);
			func_28(1);
			break;
		
		case 1:
			...
		
		case 2:
			...
		
		case 3:
			...
	}
	iLocal_234 = true;
}
```

That's it, the mission restored it's state according to the CP that was saved on MF and with that we now know the general Mission Fail and Retry flow.