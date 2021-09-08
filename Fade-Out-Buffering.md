##### This research was done on build 944. Not exactly sure which 1.xx version it is.

# Fade Out buggering in GTA V

Explanation for fade out buffering.

## Fade_Out & Fade_In

There are 2 native functions responsible for fading in and out:

```
void DO_SCREEN_FADE_IN(int duration);
void DO_SCREEN_FADE_OUT(int duration);
```

`duration`: The time the fade should take, in milliseconds. 

Once the screen is faded in\out, you usually want some actions to take place. 
That's what `BOOL IS_SCREEN_FADED_IN()` and `BOOL IS_SCREEN_FADED_OUT()` are for.

For example, you can do something like this:

```
func_1()
{
	DO_SCREEN_FADE_OUT(800); //call fade out
	func_2();
}

func_2()
{
	while(true)
	{
		if(IS_SCREEN_FADED_OUT()) //wait for it to fade out
		{
			...	//do something
			DO_SCREEN_FADE_IN(0); //fade in the screen
			break; //break the loop
		}
		system::wait(0);
	}
}
```

Let's call this piece of code `SCRIPT_1`.

## Fade_Out buffering

### What is Fade_Out buffering?

Fade_Out buffering means that we can buffer the code under `if(IS_SCREEN_FADED_OUT())` to execute later.

[Example Video](https://youtu.be/NexXs8VTJTY)

### How does it work?

The thing is, our example code is actually bad. I mean, it's alright if we are the only script running but that's not the case for GTA V where a lot of scripts run in [quasi-parallel](#how-to-determine-which-script-will-consume-fade-out) mode during gameplay.
We can easily end up in a situtation like this: 

Let's say we start our `SCRIPT_1`, it executes `DO_SCREEN_FADE_OUT(800)` and the screen starts to fade out. Now let's say some `SCRIPT_2` calls `DO_SCREEN_FADE_IN(0)` 400 ms after our fade out.
What will happen to our `while(true)` loop? `if(IS_SCREEN_FADED_OUT())` will not succeed because the screen never actually faded out thus leaving us waiting for the next successful fade out.
Voila, we succesfully buffered the fade out.

With `SCRIPT_1` stuck in a loop, let us start `SCRIPT_2` which is completely identical to `SCRIPT_1`.
Now `SCRIPT_2` will call `DO_SCREEN_FADE_OUT(800);` and after 800 ms `SCRIPT_1` will consume `if(IS_SCREEN_FADED_OUT())` and fade in the screen. Since the screen is faded in now, `SCRIPT_2` is now stuck in `while(true)` waiting for the next successful fade out.

With that, we successfully chained fade out buffering to the `SCRIPT_2`. 

That's exactly what happens in the [example video](https://youtu.be/NexXs8VTJTY).

`vehicle_gen_controller` initiates the fade out, few frames later mission failed screen fades the screen back in and finally we buffer `ob_huffing_gas` which we execute later by entering the garage and fading out again (we end up with buffered `vehicle_gen_controller` once again).

## The normal way scripts handle fade outs

GTA V scripts handle fade outs differently, some scripts have bad implementations while some have solid ones.

For example `mission_repeat_controller` handles it like this:

```
while (!cam::is_screen_faded_out())
{
	if (!cam::is_screen_fading_out())
	{
		cam::do_screen_fade_out(500);
	}
	system::wait(0);
}
```

The script will always fade out until the screen is faded out preventing fade out buffering from occuring. 

## How to determine which script will consume fade out

Scripts in GTA V aren't completely parallel, one script will continue executing until it reaches a `WAIT` native, then the game switches over to the next script. The order of script execution depends on the scripts array (from first to last).
What that means is: scripts that are started earlier (and thus added to the scripts array first) have precedence, provided no script has ended execution. If another script did end it's execution, new script can take it's place in the scripts array allowing it to consume fade out despite 
being launched later than some other script(s) that wants to consume it too. Thanks to Parik for this info.