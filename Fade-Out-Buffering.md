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
	}
}
```

Let's call this piece of code `SCRIPT_1`.

## Fade_Out buffering

### What is Fade_Out buffering?

Fade_Out buffering means that we can buffer the code under `if(IS_SCREEN_FADED_OUT())` to execute later.

[Example Video](https://youtu.be/NexXs8VTJTY)

### How does it work?

The thing is, our example code is actually bad. I mean, it's alright if we are the only script running but that's not the case for GTA V where a lot of scripts run in parallel during gameplay.
We can easily end up in a situtation like this: 

Let's say we start our `SCRIPT_1`, it executes `DO_SCREEN_FADE_OUT(800);` and the screen starts to fade out. Now let's say some `SCRIPT_2` calls `DO_SCREEN_FADE_IN(0)` 400 ms after our fade out.
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

## Unanswered questions

How do we determine which script will consume `IS_SCREEN_FADED_OUT()` first? It seems that the example video is reproducable every time. However, if we try to chain property buying cutscene (Trevor's airfield, taxi service, towtruck) from `main` script, we will fail 
because `main` will be the first to consume `IS_SCREEN_FADED_OUT()`.

Related code from `main`:

```
if (func_1233() && gameplay::get_game_timer() >= Global_100364.f_43 + 1000) //check if we skipped the cutscene + some timer check
{
	cam::do_screen_fade_out(800);
	while (!cam::is_screen_faded_out())
	{
		system::wait(0);
	}
	func_1200(iParam0); //fade in
}
```

The only guess I have is that it depends on the execution speed of a particular script. Since `main` will be constantly checking `!cam::is_screen_faded_out()` in a loop, it will execute fade in faster than for example `vehicle_gen_controller`.
But that's only a guess, I'm not sure about this.