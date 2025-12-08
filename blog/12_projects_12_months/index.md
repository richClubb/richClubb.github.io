---
title: "12 projects in 12 months"
subtitle: "12 projects in 12 months"
date: 2025-12-05
tags:
  - programming
  - making
  - electronics
---

I've got a list of projects as long as my entire body, woodworking, electronics, programming, so many different things I've started or been interested in playing around with but never really started or finished. This idea is to try and motivate myself to do things with them, to finish them off or to just put them away.

There are a couple of guidelines to this:
* Spend 1 month
* Scope the ideas well
* Plan what you're going to do, but don't be too worried about the plan
* At the end of the month, take a good look and assess if you're going to finish them, or put them down.

One thing I'd like to note here is that generally I don't have a lot of time to work on these types of projects. It's mostly 1 - 2 hours in the evening depending on how much stuff I've got to do, and most of my weekends are spent with my family or doing stuff around the house.

# Month 1 - LED Clock

* Repo Link - https://github.com/richClubb/led-clock
* Time spent: ~30 - 40 hours
* Date started - 2025-09-04 (but some stuff done before then)

This was a pretty simple project to finish off the code for an LED clock that I'd started a few years ago and had just been decaying in a drawer for a while.

TLDR: Made a 2 (but can use 3) ring LED clock which has hour, minute and second "hands". The leds can be set on the different rings, the colours are configurable, and it also has a timer that can be used as a count-up clock but still displays the time even while the timer is active. Uses an RTC to keep accurate time even when off, and has a custom serial interpreter to be able to program the settings. Everything written in the Arduino IDE using an OTS ESP32C3 dev board and Neopixel led displays, so anyone can use this.

[Blog Page](./month-01-led-clock.md)

# Month 2 - 2023 Advent of code, Day 05

* Repo Link - https://github.com/richClubb/adventofcode/tree/main/2023/day05
* Time spent: ? (didn't measure, but way longer than I should have)
* Date started - 2025-09-16

Completing the AoC 2023 Day 05 problem in a bunch of different ways, For the detailed analysis I covered this in a [separate blogpost](https://richclubb.github.io/blog/spending_way_too_much_time/) as I spent way too much time doing this but had a lot of fun doing it.

TLDR: Solved it in 10 different languages (C, C++, C#, Cuda, F#, Go, Jave, Python, Rust, Zig) in about 20 ways including mutliprocesing and FFI implementations. Got the brute force execution down to > 10 seconds (Rust with rayon, Cuda, and C OpenCL CPU multiprocessing) and the fastest time solving the problem was > 1 s. Compared writing the problem in different languages and speed comparisons between them and found some interesting things.

[Blog Page](./month-02-aoc-2023-day5.md)

# Month 3 - Learning Microcontrollers

* Time spent - 4 hours
* Date started - 2025-12-06

[Blog Page](./month-03-learning-microcontrollers.md)


