---
title: Sc2 for Mac OS X and memory reading/writing
tags:
  - mac os x
  - memory hacking
  - memory manipulation
  - ReadProcessMemory
  - sc2
  - starcraft 2
  - vm_read_overwrite
  - vm_read()
  - vm_write
  - WriteProcessMemory
url: sc2-for-mac-os-x-and-memory-readingwriting.html
id: 364
categories:
  - android
date: 2010-08-24 13:50:25
---

So recently, I a heard from a few friends that the Starcraft II had some pretty interesting anti-cheating methods in place. I figured I'd try to check them out - though I only have a mac now. So after firing up the game, getting distracted and playing about half way through the single player -- I remember why I actually installed it, to check out the anti-cheating protection. Sadly, but not surprisingly, it doesn't appear that there is any anti-cheating protection on the mac client. It's not really surprising because to implement something like the Warden they would need root, or some sony style rootlet for the mac ;) 

Any who, it reminded me of the good old days of using `ReadProcessMemory` and `WriteProcessMemory` making quick and dirty memory hacks for Windows games. Making stuff for Diablo 2 consumed a lot of my free periods back in high school - great times! Back on topic though, it got me thinking of just dumping some sc2 memory and doing some quick memory hacks, though I never really did any of this on a mac before.

Turns out it's also pretty easy to do some memory hacking in Mac OS X - you just need to know where to look. First of, if you found this site by googling "mac starcraft2 hacks", then stop read -- just go download "The Cheat" and use that, since it'd be easier than compiling your own program. Though basically The Cheat uses these same functions I stumbled upon.

Basically we're going to use `vm_write` instead of the old `WriteProcessMemory` and `vm_read_overwrite` (or `vm_read`) instead of `ReadProcessMemory`. There's some documentation out there but it's pretty simple stuff to use. Below I've pasted an example of how easy a sc2 trainer would actually be to make;
```
/*
 * [ s2trainer.c ]
 * strazz@gmail.com
 * 2010
 */

#include <mach/mach.h>
#include <stdint.h>
#include <stdlib.h>
#include <stdio.h>

#define MINERALS 0x03200D80

int main(int ac, char **av) {
  mach_port_t sc2_task;
  kern_return_t err;
  long value = 0;

  if(ac != 2)
    return 0;

  // Make sure we're root
  if(getuid() && geteuid())
    printf("error: requires root.");

  value = atoi(av[1]);
  err = task_for_pid(mach_task_self(), value, &sc2_task);

  if ((err != KERN_SUCCESS) || !MACH_PORT_VALID(sc2_task))
    printf("error: getting sc2 task.");

  // Write values to stack.
  if(vm_write(sc2_task, (vm_address_t) MINERALS, (vm_address_t)&value, sizeof(value)))
    printf("error writing new mineral value");

  printf"done!\n");
  return 0;
}
```
Is this pretty? No - it could be much prettier than this. Is this safe? For the latest patch as of today, yes - it works fine. I wouldn't recommend running it on any other version since the offset will change and it could lead to bad things. Will it work on a window machine? Heck no - the title says MAC OS X. 

Also, yes I could have tossed SC2 into vm fusion, but thats far to much work just to mess around with a game :)