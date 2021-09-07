##### This research was done on build 944. Not exactly sure which 1.xx version it is.

# Mission Failing and Retrying in GTA V

Some research on mission failing (MF from now on), retrying and checkpoints (cp).

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

Now that `replay_controller` is running, we need to fail the mission to trigger it. But before we look at how it triggers, let's first look at how cps work since we would want to save current checkpoint when we MF.

Once again, nothing extraordinary, we just look for native `playstats_mission_checkpoint` in our mission script. Most likely we'll find this piece of code (`rural_bank_setup` or Paleto Score Setup will be used here):

```
void func_736(int iParam0, char* sParam1, int iParam2, int iParam3, int iParam4, int iParam5)//Position - 0x7E55D
{
	iParam0 - new cp,
	sParam1 - cp description,
	iParam2 - final checkpoint,
	iParam3 - ignore if new cp < current cp,
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
	iVar0 = 0;	//should change current cp
	if (iParam3 == 1) //if (ignore if new cp < current cp)
	{
		if (iParam0 != Global_91523)
		{
			iVar0 = 1;
		}
	}
	else if (iParam0 > Global_91523) // if(new cp > cur cp)
	{
		iVar0 = 1;
	}
	if (iVar0 == 1) //if (should change current cp)
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
		Global_91523 = iParam0; //saving current cp
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

Now we know that `Global_91523` is `CURRENT_CHECKPOINT` and `Global_85997` is `IS_FINAL_CHECKPOINT` (the latter one isn't really interesting and is used to determine whether the game should diplay "Skip Mission" or "Skip Section" on MF screens).

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

MF states:

```
0 - MF before MF screen appears
1 - MF screen appeared, main state
2 - Pressed No on Mission Restart screen
3 - Pressed Restart when CP > 0 (cut feature to restart the mission independent of CP?)
4 - Pressed Exit Mission, Confirm Exit state
5 - Pressed Skip, Confirm Skip state
6 - Some state for main missions with type == 2 (examples are : drf1, drf2, drf5). Gets set when No is pressed on Skip Checkpoint\Mission or Confirm Exit screens. Not sure what it's for and the code looks weird, is it failsafe\unused code of some kind?
7 - Pressed Retry\Restart\Yes on Confirm Skip\
8 - Pressed Yes on Confirm Exit screen
9 - State finished setup for restarting mission after Retry\Restart\Skip
10 - Finished Retry\Restart\Skip
11 - State after Retry\Restart\Skip for special missions: jewelry_prep2A, jewelry_prep1B, fbi4_prep1, agency_prep1, rural_bank_prep1 + State for when benchmark is running
12 - State after replay_controller termination 
13 - Default state after starting replay_controller + state to terminate replay_controller after exiting the mission.
```

We can quickly check, that after failing the mission this Global is equal to 0.

So, let's check what sets `MISSION_FAILED_STATE` to 0. Here main missions and S&F go separate ways. Logic is mostly the same but for S&F missions MF is triggered from the script itself while for main missions it's triggered from `flow_controller`.

For main missions algorithm's like this:

```
1) Mission Fail
2) Change some state varialbles
3) Cleanup and Terminate the script
4) flow_controller restarts the script
```

Let's look at flow_controller. The logic here happens in one of functions that were discusses in mission triggering research: `Flow_Do_Mission_Now`, `Flow_Do_Mission_At_Blip` or `Flow_Do_Mission_At_Switch`.

At the end of each of those functions we can find a call to `func_238` which in turn will call other relevant functions like saving stats. Among those calls we can find `func_241` 





Let's look at what state 13 does:

```
case 13:
	if (!(((((func_1(0) || func_1(3)) || func_1(2)) || func_1(4)) || func_1(9)) || func_1(10)))
	{
		func_737();
	}
```

And `func_1` should be familiar to us:

```
int func_1(int iParam0)//Position - 0x1FB
{
	return Global_35777 == iParam0;
}
```

We saw this function already in `main.c`. It checks if current `GAME_STATE` is equal to `iParam0`.
In this case, we use it to do exactly the opposite from `main.c`, we check if `GAME_STATE` is not equal to any of the mission states that require `replay_controller` and call `func_737` if it's the case:

```
void func_737()//Position - 0x76A70
{
	func_738();
	Global_91486 = 12;
	script::terminate_this_thread();
}
```

Alright, now we know 