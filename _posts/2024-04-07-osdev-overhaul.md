---
title: "osdev: overhaul"
categories:
  - blog
tags:
  - learn
  - osdev
---

# Prelude
This is the my first blog on osdev, my journey of operating system development.
I started osdev on the winter holiday of 2023, and it has been 4 months now.
Indeed, as warned by [The Hard
Truth](https://wiki.osdev.org/Getting_Started#The_Hard_Truth), osdev is a
time-consuming and **hard** project.\
In the past 4 months, I have carried this project with me as a nighttime hobby.
I managed to set up a basic framework from using [Meaty
Skeleton](https://wiki.osdev.org/Meaty_Skeleton), and enabled GDT, IDT by
following many useful resources. I'm currently working on
[paging](https://wiki.osdev.org/Paging).\
Two important things that I *experienced* (I was going to say *learned*, but I
still feel unsure about whther I have learned the gist) so far:
1. Working with uncertainties. 
* It's amazing how uncertain osdev is. There are less resource, less nice
  carry-you-all-the-way YouTube videos, less bootcamp courses. Osdev trains me
  to read documentation (the Intel manual), dig into ancient 2000s forums, and
  read other project's code (in this case, xv6 served me well). There are less
  "best practices", at least less that I can search for. Every decision is a
  *choice*, that may or may not lead me to a brighter path. Honestly, I haven't
  even set a scope or goal for this project, but I'm starting to enjoy this
  uncharted ocean.
2. How to get things moving.
* This is related to the uncertainties. Because there are too many paths I can
  potentially take, I kept spending time on deciding one, and less on actually
  writing code. After a while, I realized that it is way more important to get
  my hands dirty and start to write something. The answer won't just pop up
  when I'm sitting and thinking. Write something, anything, even some trashy spaghetti,
  that can serve as a base that I can work on top of. Overthinking is useless.


# Overhaul
[Overhaul](https://github.com/ffyuanda/osdev/tree/overhaul) is a series that tries to flatten the project structure of [Meaty
Skeleton](https://wiki.osdev.org/Meaty_Skeleton) (will reference as MS). 
## Motivation
I liked
AS, because of its nice separation between userspace and kernel, and the
consideration of hardware compatibility and future extendability.  
However, I
loathed it because of the nesting directories and overall complex organization
brought by its well-rounded design. I'm not saying its design is bad, but it is
stirring my brain. I have to keep spending energy understanding the project
structure and its implications.  
I believe MS will serve its value when the project is more mature, but I need to
focus on writing code right now, instead of walking that directory maze.  
So, I decided to flatten the whole directory, a bit like [how xv6 does
it](https://github.com/mit-pdos/xv6-public). I reaffirmed my decision after
reading [Should your source code be in one folder or should it be
nested](https://breckyunits.com/code/should-your-source-code-be-in-one-folder-or-should-it-be-nested.html),
and [Favour flat code file
folders](https://blog.ploeh.dk/2023/05/29/favour-flat-code-file-folders/).
## Action
Well, this is simple, let's move all the regular files to the root directory of the
project, done! And, not surprisingly, boom! Okay, the build system is not happy
because I just flipped its world upside down.  
To make the build system happy again, I need to read
the Makefile and trace back how it was doing things, then I need to adjust all
the Makefile variables, build path, shell scripts, etc. to satisfy my current
flattend view. I'll save the actual details from being listed here, simply
because it is very tedious and complex.  
## Onward
One result of having a flat directory is that the kernel code is no longer
separate from the libc, or libk(ernel) in this case. I can't see an urgent need
to separate them apart, could be that I'm very inexperienced, but I'll leave it
this way until there is a need to extract out libk.  
I learned a lot about Makefile and the whole MS build system
after finishing overhaul, and I can see these knowledge immediately in use
continuing down my osdev journey. Once again, this series is not a denial of the
value of directories and more structured codebase, but a makeshift change that
helps me get through this turbulent starting phase by elminating the overhead of
maintaining the MS structure. I can finally focus on
writing the code now.
