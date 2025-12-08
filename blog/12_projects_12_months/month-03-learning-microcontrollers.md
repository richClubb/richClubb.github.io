# 12 Projects in 12 Months - Project 3 - Learning Microcontrollers

I'd wanted to create a good resource for deep learning on how to use and implement microcontrollers. I've been working with various micros in different capacities for years and there are still areas that I don't fully understand so this was part of the "see one, do one, teach one" philosophy. I've seen them used, I've used them, but I've never really tought people how to use them.

## Starting from scratch

A lot of the time people are already starting from a pretty established position before they touch the code. Arduino gives you a very easy point and click method of starting with micros, but there are some downsides in that you don't fully know how things work under the hood and it will set things up in ways that can be affect you later. For example, `delay` in arduino uses one of the timer interrupts, but what if you want to use that timer interrupt for your own things? What emergent behaviour might creep up if you do.

The idea is to be able to start from as "bare metal" as possible. The reference manual and some open-source tools, and get a basic blinky up and running. Then try and get a serial interface working with a basic request / response.

## RTFM!!!

If you want to do bare metal, you have to know your processor, you have to be able to read the reference manual and understand how to find out what you need.