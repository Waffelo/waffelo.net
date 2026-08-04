+++
title = 'Fiddling_with_voron'
date = 2025-07-11T13:28:02+02:00
draft = false
+++

So in the recent days, I have been building a Voron. I even took days off work to give it attention.
I managed to build the entire Voron in like 3 days of nearly nonstop building. I slept around 3 hours
each night. Yeah I was kinda hyper focused on it, I just wanted to have it done.
Assembly was pretty fun, I tried to make it the closest to perfection, most difficult thing about it was
putting up the A and B belts. Then there was electronical wiring which was sort of easy, the only difficult
thing about it was making the cables fit the plastic channels for cable managment. I took some time flashing
the controller boards. I first flashed the main controller and then tried to flash the canbus controller. I
took me a day to understand it better and make it work all together.

I came to the point of first time powering the printer, it went all fine. No explosion happened! I have been
assigning pins in the klipper config and everything seemed to work. Except the motors, I kept getting errors
that they cannot connect via UART (IFCNT and GCONF error). I have swapped the controller board and this still
happens. So I instinctively started verifying all connections, jumpers, and now even schematics to see how
things are really connected. The motors just don't work.. Or perhaps the drivers. I have tried putting them to
the standalone mode, but that didn't work. They only locked in, and that's about it.

I might just replace the drivers now. I am pretty desparate in this. It feels like the controller board
documentation is completely missing. But they award it as "most performant spider board". I don't even try to
ask people for help because those retards tell me "check google", or "do you have the jumper in?" after I state 
what everything I have already tried.

So yeah, I made a [Github issue](https://github.com/FYSETC/FYSETC-SPIDER-H7/issues/7) if anyone wants to look at it...
