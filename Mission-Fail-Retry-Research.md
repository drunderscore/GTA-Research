##### This research was done on build 944. Not exactly sure which 1.xx version it is.

# Mission Failing and Retrying in GTA V

Some research on mission failing\retrying (MF from now on).

## Scripts Responsible for MF

```
main: Launches replay_controller.
replay_controller: Displays MF screen, changes the MF state depending on player's input, restarts S&F missions.
flow_controller: Restarts main missions, saves checkpoint on MF for main missions, triggers replay_controller for main missions. 
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

As I said, everything is pretty straightforward, we check if `Global_35777` or `GAME_STATE` is one of mission states that require MF screen + check if `replay_controller` is not running already. If those conditions are satisfied, we launch `replay_controller`.

## Mission Failing

Now that `replay_controller` is running, all we need to do is fail the mission.


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