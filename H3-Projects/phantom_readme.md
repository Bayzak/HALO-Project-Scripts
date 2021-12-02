<h1>PHANTOM.HSC WHITEPAPER</h1>
With how HSC files work, I figured having a readme for the file outside of the actual HSC would make more sense.

To understand how the script works, you'll need to understand the two parts of it - The Actual Work and The "Director"

<h1>THE DIRECTOR</h1>
This one isn't nearly as hard to understand, and makes the actual "dropship" function much easier to understand.

Quite literally, take this in two parts: The check to see if the phantom is alive, and if it is then ignore the rest of the script.

```
(script continuous respawn_Director
  (if
  (< (ai_living_count phantom_Dropship) 1)
  (. . .)
  (print <expression>)
  )
)
```

And, the check to see if the phantom is dead, but the squads are alive.

How this works is that four squad groups were made, and each squad group consisted of four squads.<br>
To randomly select what groups spawn, we implement a "simple" check, like so:

We first grab a random integer between 0 and 30, and we check if that number is 14 or less.<br>
If it is, then we follow Clause A, otherwise we follow Clause B.<br>
`(if (< (random_range 0 30) 15) (Clause A) (Clause B) )`

In the instance of the script, both Clause A and Clause B follow the same random integer check; <br>however, their Clause A and Clause B are picking which of the two squads to spawn from the squad group.

Now, to ensure that we do not spawn any additional units or pull units from where they are currently fighting and putting them back onto the Dropship, we will use the following:<br>
`(< (ai_living_count SQUAD_GROUP) 1)`

As long as there is a single living unit out of the four possible squads in the squad group, then the squad group is still technically alive.<br>
This may cause the Phantom to take longer to respawn, since it waits for all 16 individual units that make up the four squads to die, but it allows the spawn cycle to work without issue -- more over, without it pulling enemies away from the player.

So, in the end, we spawn four groups (16 units) out of 16 possible groups (64 units), and the Phantom Dropship itself.

```
(if
  (and
    (< (ai_living_count phantom_SQ1) 1)
    (< (ai_living_count phantom_SQ2) 1)
    (< (ai_living_count phantom_SQ3) 1)
    (< (ai_living_count phantom_SQ4) 1)
  )
  (begin
    (ai_place phantom_Dropship)
    (if (< (random_range 0 30) 15)
      (if (< (random_range 0 30) 15)
        (ai_place phantom_Grunts_SQ1)
        (ai_place phantom_Brutes_SQ1)
      )
      (if (< (random_range 0 30) 15)
        (ai_place phantom_Jackals_SQ1)
        (ai_place phantom_Jackal_Snipers_SQ1)
      )
    )
    (if (< (random_range 0 30) 15)
      (ai_place phantom_Grunts_SQ2)
      (ai_place phantom_Brutes_SQ2)
    )
    (if (< (random_range 0 30) 15)
      (ai_place phantom_Grunts_SQ3)
      (ai_place phantom_Brutes_SQ3)
    )
    (if (< (random_range 0 30) 15)
      (ai_place phantom_Grunts_SQ4)
      (ai_place phantom_Brutes_SQ4)
    )
    (cs_run_command_script phantom_Dropship/phantom_Spawn phantom_AI)
  )
)
```

<h1>THE ACTUAL WORK</h1>

So, beginning the command script, the Phantom will pick up the four squads and immediately load them into the four passenger parts of the ship.<br>
It will then immediately fly to the first dropsite.<br>
Sleep for one second, face a point, sleep for three seconds, open the doors and begin the "troop drop" segment.

```
(ai_vehicle_enter_immediate phantom_SQ1 (ai_vehicle_get ai_current_actor) "phantom_p_mr_f")
(ai_vehicle_enter_immediate phantom_SQ2 (ai_vehicle_get ai_current_actor) "phantom_p_mr_b")
(ai_vehicle_enter_immediate phantom_SQ3 (ai_vehicle_get ai_current_actor) "phantom_p_rf")
(ai_vehicle_enter_immediate phantom_SQ4 (ai_vehicle_get ai_current_actor) "phantom_p_rb")
(cs_fly_to phantom_Drops/phantom_p0_DropSite 1)
(sleep 30)
(cs_face TRUE phantom_Drops/phantom_p0_Face)
(sleep 90)
(unit_open (ai_vehicle_get ai_current_actor))
(sleep 90)
```

Troop drop. Straight forward, just drops the troops from the given points.<br>
We call `ai_current_actor` as the dropship. The dropship is evicting the troops; the troops are not technically being told to leave.

```
(begin
    (vehicle_unload (ai_vehicle_get ai_current_actor) "phantom_p_rf")
    (sleep 30)
    (vehicle_unload (ai_vehicle_get ai_current_actor) "phantom_p_rb")
    (sleep 30)
    (vehicle_unload (ai_vehicle_get ai_current_actor) "phantom_p_mr_f")
    (sleep 90)
    (vehicle_unload (ai_vehicle_get ai_current_actor) "phantom_p_mr_b")
    (sleep 90)
  )
```

After which it closes its doors, unlocks the face target, moves to another location to loiter, and eventually leaves to unspawn.

```
(unit_close (ai_vehicle_get ai_current_actor))
(sleep 90)
(cs_face FALSE phantom_Drops/phantom_p0_Face)
(cs_fly_to phantom_Drops/phantom_p0_Face 1)
(cs_face TRUE phantom_Drops/phantom_p0_DropSite)
(sleep 900)
(cs_face FALSE phantom_Drops/phantom_p0_DropSite)
(cs_fly_to phantom_Drops/phantom_Death)
(sleep 30)
(object_destroy (ai_vehicle_get ai_current_actor))
```

<h1>THE END</h1>

It was an interesting project to work on; and it definitely needs to be cleaned up.<br>
This scripting language is not the most fun to work in because of how specific you need to be with spaces, values, and parenthesis.<br>
But, I won't lie. Having troops successfully drop from the phantom was a dopamine rush.

This code was taken and refitted from a reddit post from another user -- if I do manage to come across the user again then I would be more than happy to link their work.<br>
Because this wouldn't have made it this far without that code to jump from.

https://www.youtube.com/c/fryingpanproductions/videos<br>
Also, go check out fryingpanproductions, specifically his work on<br>
https://www.youtube.com/watch?v=6UpjWyGyJHY - Tutorial 13 - Flying Phantom & Pelican<br>
https://www.youtube.com/watch?v=ffEsuiEyW7A - Tutorial 15 - Respawning AI<br>
https://www.youtube.com/watch?v=r22L8hbOfWw - Tutorial 20 - Pelican Troop Dropoff















