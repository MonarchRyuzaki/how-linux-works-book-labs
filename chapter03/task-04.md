# Task 04: The GUI Freeze Out

## Scenario (Mode B: Production Outage)
You are physically in the datacenter (or using an out-of-band KVM console) looking at a monitoring workstation. A memory leak in the dashboard software has completely frozen the graphical user interface. The mouse won't move, and you cannot open a terminal emulator window. 

The machine is still running, but the GUI on the current screen is dead.

## Goal
Using your knowledge of Linux terminal devices (`/dev/tty*`) and virtual consoles:
1. What exact keyboard combination can you press to bypass the frozen GUI and drop into a raw, text-only virtual console?
2. If the keyboard combination fails due to an input capture issue, what command could you run remotely (if you SSH'd in from your laptop) to forcefully switch the monitor's display to virtual console 1 (`/dev/tty1`)?

## Solution
1. Some sort of `CTRL+ALT+F1/2/3` combination.
2. Or I can ssh from my laptop into the machine and type `chvt 1` which is equivalent to typing `CTRL+ALT+F1` 

**Correction:**
**Grade: 100% (Exactly right)**

1. `CTRL+ALT+F1` (or F2, F3) is the hardware interrupt to switch virtual consoles (TTYs). The GUI usually runs on one of them (often F7 or F1 on modern systems), and the others are pure text-mode consoles waiting for a login prompt.
2. `chvt 1` is a lifesaver command. It tells the kernel to forcefully switch the physical monitor output to Virtual Terminal 1 (`/dev/tty1`), regardless of whether the GUI has locked up the keyboard input. 
