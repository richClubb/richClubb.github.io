---
title: "Spending way too much time solving one problem"
subtitle: "Solving Advent of Code 2023 Day 5 too many times"
date: 2025-12-05
tags:
  - programming
---

I have officially spent too much time solving the Advent of code 2023 day 5 problem. I've solved it in 10 languages in a bunch of different ways. Functionally, with GPUs and CPU multiprocessing, being "smart" and just brute forcing the problem, and here is what I've learned.

# Overview

This Advent of Code (AoC) problem has been an ongoing thing for me for a few years. Initially I solved it with a group of people at work as a technical discussion and it's been a bit of a test-bed for me as it's a simple problem but it complex enough that you can get your teeth into it. It's possible to solve in a single file (and yes I've been nasty and done that) if you just want to solve it, but you can be nice and do it modularly with good unit tests and coverage. I'd encourage you to find a problem or a set of problems that you can go back to, [Low Level](https://www.youtube.com/@LowLevelTV) uses an HTTP server to act as his litmus test for a language. I'd recommend having 3: 
* a very easy one (but harder than hello-world) so you can get to grips with the syntax, functions, variables etc.
* a medium one to get used to package management / structure
* a harder one if you really want to get your teeth into the language and put it through it's paces.

# What is the AoC 2023 Day 5 problem

The official site page is [here (official)](https://adventofcode.com/2023/day/5) requires logging in and completing Part A for full information, or [here](https://github.com/richClubb/adventofcode/blob/main/2023/day05/docs/problem.md) is the text copied out.

The simple explanation is that you have an input value and a set of maps to transform it through to get an output.

```
     |-------|
     | Input |
     |-------|
         |
    |---------|
    | Layer 1 |
    |---------|
         |
       .....
         |
    |---------|
    | Layer 7 |
    |---------|
         |
    |---------|
    | Output  |
    |---------|
```

Each layer consists of a set of maps which have a `source` a `target` and a `size`. If the input value is between `source` and `source + size`, it translates to the corresponding place in `target` to `target + size`. If the value isn't in any of the maps then it passes through unchanged. E.g. if you have a map with; source 5, target 10, size 2. Then:
* f(4) = 4
* f(5) = 10 
* f(6) = 11
* f(7) = 7

It's a pretty simple problem, basic math and logic.

```
if (value >= source) and (value < source + size) then (value = value - source + target)
```

There are 2 main datasets, the [sample data](https://github.com/richClubb/adventofcode/blob/main/2023/day05/sample_data.txt) and the [full data](https://github.com/richClubb/adventofcode/blob/main/2023/day05/full_data.txt). As you can see the sample data is a much smaller dataset, and the values are all < 255 where the full data is a much larger dataset with much larger values.

## Part A

In Part A of the problem you have a set of inputs 20 for the full data and 7 layers with about 250 maps in total for the full dataset. The sample data has a much reduced set of seeds (79 14 55 13) and maps (about 30)

## Part B

Part B throws a spanner in the works. The simple input turns the simple set of values into a set of ranges with a `start` and a `size`, so our initial values of `79 14 55 13` become:

* 79 start, 14 size = 79 to 92 (inclusive)
* 55 start, 13 size = 55 to 67 (inclusive)

So our original input set has gone from 4 values to 27 values (or maybe it hasn't.... foreshadowing). But that's only 23 more values? It can't be that bad? For the sample data, no. But for the full data it goes from 20 values.. to 1,975,502,102. Just to put into perspective. If calculating each seed took 1 second. Part A would take 27 seconds, part B would take 60+ years.

# How to Solve this?

Unless you're an AoC veteran (and frankly even if you are you might not foresee this) whatever way you came up for solving this for part A is probably going to take a very long time to compute the final value, so with all the information at hand on both problems, lets see what ways we can solve this.

## Depth First

This is probably the simplest method, and the one most people I've talked to come up with. You start with a minimum value, you run each value through the layers and check it against the minimum, if it's smaller, update and continue, and then output the minimum value at the end. 

It's simple, succinct and for a programmer of any experience you'll have it done in an hour (depending on your language, but we can come to that later)

## Bredth First

This is still pretty easy, you translate each value from the input set through the layers and at the end find the minimum from the final set.

This problem actually parallelises well if you have a massive number of cores, each core can take a single value and the maps and calculate it's output and then at the end you find the minimum.

It does have one pretty significant flaw, it will take a LOAD of memory for large input sets. (I ran into this problem later)

## Translate the whole ranges of values

This is a trickier one to conceptualise but isn't hard once you've got your head around it. Instead of breaking each of the ranges in part B into discreet values, we transform the whole range at once and depending on how the range falls into the map you could translate part of it or the whole thing. I wrote out a [mapping of 13 cases](https://github.com/richClubb/adventofcode/blob/main/2023/day05/docs/Range%20Analysis%20Notes.pdf) (which can be reduced to fewer).

E.g.

```
Map: source 10, target 20, size 5

Case 1 (outside )
Range 5 - 7
Output: 5 - 6

Case 2 (partial overlap)
Range 5 - 13
Output (5 - 9), (20 - 23)
```

Won't put all the cases here, you get the idea.

This would take our ~2,000,000,000 inputs down to 10, which might be faster.....(foreshadowing)

# Optimisations to the solutions

Donald Knuth said "premature optimisation is the root of all evil" and he's right. Don't try to optimise until you know where the problems are. There is a bit of a caveat here though, you should have "designed" your problem properly first.

Optimisation can get you local maxima, but maybe not the global. A bit of information from later, if you started with a brute force C implementation, you can get it down to under 2 minutes with compiler optimisations and profiling, but I can get Python down to less than 1 second by translating the ranges rather than brute forcing things without ever optimising the Python solution.

But, lets think about designs, analysing the problem and optimisation.

## Profile your code to identify the bottleneck

Pretty much every language has tools to profile usage, if we're starting with Python we have `cProfile`. If we start with the `day5_classes.py` script.

```
python3 -m cProfile day5_classes.py ../../input_reduced_1M.txt part_b_forward
...
   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
      7/1    0.000    0.000   62.958   62.958 {built-in method builtins.exec}
        1    0.000    0.000   62.958   62.958 day5_classes.py:1(<module>)
        1    0.442    0.442   62.951   62.951 day5_classes.py:474(part_b_forwards)
  1000000    1.016    0.000   62.018    0.000 day5_classes.py:436(map_seed)
  7000000   14.710    0.000   61.002    0.000 day5_classes.py:341(map_seed)
183000000   34.014    0.000   46.292    0.000 day5_classes.py:156(map_seed)
345000001   11.983    0.000   11.983    0.000 day5_classes.py:26(value)
  1000001    0.304    0.000    0.352    0.000 day5_classes.py:128(__next__)
  8000000    0.343    0.000    0.343    0.000 day5_classes.py:12(__init__)
  1000000    0.137    0.000    0.137    0.000 day5_classes.py:46(__lt__)
```

Three calls stand out: `day5_classes.py:341(map_seed)`, `day5_classes.py:156(map_seed)` and `day5_classes.py:26(value)`

For `day5_classes.py:341(map_seed)`
```
    def map_seed(self, seed: Seed):
        for mapping in self.__mappings:
            if result := mapping.map_seed(seed):
                return result
        return None
```
At a quick glance there isn't much we can do here

For `day5_classes.py:156(map_seed)`
```
    def map_seed(self, seed: Seed):
        if (seed.value >= self.__src) and (seed.value <= self.__src + self.__size - 1):
            return Seed(self.__dest + (seed.value - self.__src))
        return None
```
Similar to the last function call there isn't alot to do here.

For `day5_classes.py:26(value)` it's a little more complex
```
    @property
    def value(self):
        return self.__value
```

This is a getter method for the `Seed` class which was an experiment in being overly object oriented. By removing this class and using just an integer we could speed up the code by about 20%. So that might be a good start.

There might be some other things we could do, but I haven't gone any further with profiling python, and I started looking into other (more drastic) solutions.

## Change your language

One of the initial investigations was into how ChatGPT could translate code to different languages, after we saw how slow our python implementation was. We used it to convert the Python to Rust to see how it could speed up the situation, it did a pretty good first job which was really helpful.

Python
```
vscode ➜ .../2023/day05/python/src (main) $ time ./day5_classes.py ../../input_reduced_10M.txt part_b_forward
part b (forward depth first): 2804662518

real    2m42.835s
user    2m42.128s
sys     0m0.011s
```

Rust
```
vscode ➜ .../adventofcode/2023/day05/rust (main) $ time ./target/release/day5 ../input_reduced_10M.txt part_b_forward
path: "../input_reduced_10M.txt", run: "part_b_forward"
Part B forward brute force: 2804662518

real    0m0.668s
user    0m0.628s
sys     0m0.000s
```

I chose to use a reduced dataset as the python implementation takes SOOOOO long to complete but the performance difference is staggering, 162 times faster (using a release build in Rust)

C was slightly faster still
```
vscode ➜ .../2023/day05/c/build-x86 (main) $ time ./day5 -i ../../input_reduced_10M.txt -r part_b
2023 - Day 5
Running Part B
Result is: 2804662518

real    0m0.543s
user    0m0.508s
sys     0m0.000s
```

The difference between the C, Rust and Python implementations is pretty minimal in terms of logic. Rust has pretty nice string parsing functions so it's pretty easy to get the input file, but C was more complex for parsing the file but overall it wasn't too much harder, but rewriting the whole problem in a different language might be a bit too complex, which is where Foreign Function Interfaces come in.

## Foreign Function Interface (FFI)

Another way to deal with this would be writing speed code in a faster language and the calling out to it. I'd [played around with this before](https://github.com/richClubb/devlog/tree/main/2025-02-28/language_interop) and never finished the whole grid I was trying to fill out.

Ctypes in Python is a really interesting way to solve problems, it provides an FFI interface to native C code. [I wrote an example](https://github.com/richClubb/adventofcode/tree/main/2023/day05/experiments/C%20and%20python%20FFI) which uses python to parse the code from the file, and then calls out to a dynamically linked lib written in C. In total the C code has 4 structs and 4 functions, less than 100 lines and the python is 150 lines, most of which is either argparsing or extracting the file. I played around with optimising the C library with what I'd learned from my C experiments and when optimised the speed difference was insignificant in comparison to the main C example. I also spent the time to use the same lib in the same way using C, and it had a very similar execution time but took 5x as much code to sort out and was much more frustrating in terms of memory management.

## Use MORE cores

Most modern CPUs have multiple cores (unless you're on an embedded system) so why not leverage them? The majority of languages have some method for concurrency, and learning to see a problem and breaking it up into it's constituent parts is a valuable skill.

In this case, the problem parallelises pretty well on the seeds (part a), or seed ranges (part b). 

Python gives us the `multiprocessing` library which has `Pool`. This allows us to split the processing over multiple CPUs and have them simultaneously compute their local result and then check the minimum of the parallelised process.

In other languages, C has `OpenMP` and `OpenCL`, Rust has the `rayon` crate, some languages automatically deal with parallelisation like F# will just automatically use as many CPUs as it can if you're using functional methods as they can naturally scale and grow. You've also got languages like Cuda which are specifically designed for parallel computation. Multiprocessing, OpenMP and rayon work really well alongside traditional languages, and as long as you can break your problem down into a parallelisable way it's very easy. Because of how I coded my Rust implementation I only had to change a single line to get rayon parallelisation to work

Single threaded
```Rust
let min_value = seed_ranges.iter().map(|a| a.get_lowest_seed_in_range(&map_layers)).min().unwrap();
```

Multiprocess
```Rust
let result = seed_ranges.par_iter().map(|s| s.get_lowest_seed_in_range(&map_layers)).min().unwrap();
```

This gives a pretty extreme speed boost. Rust completes all the 1,900,000,000 values in just under 10 seconds with a 28 core CPU, so for 1 line this is a nice speed boost.

Cuda and OpenCL are a different beast as you've got to do a lot more boilerplate. These are useful to know and learn but make sure your code is modular enough to be able include these as necessary.

## Be clever and change your design

If you look at part A to part B in a really naive way, it goes from 20 to 2,000,000,000 values. But you can also see it as going from 20 values, to 10 ranges as we covered earlier. The seed maps are able to be translate entire ranges or parts of ranges, but it just makes the logic more complex. I did an analysis on paper and came up with 13 cases where the seed range and the seed map interact and have to be dealt with.

While the logic is more complex the actual computation is not, and recoding the problem in python got a correct solution in 0.6s compared to hours and hours brute force. Even with faster languages like Rust or C it is 10 times faster.

A really key thing to understand is that no solution is perfect, so don't get attached. They might be good for a while but all it takes is 1 small piece of information can force you to completely change your approach. "Kill your darlings" is a common literary phrase to mean don't get attached to your works or characters unnecessarily. Don't get personally invested in your solutions. If there is a better one, use it.

# Language review

## C

[Code Link](https://github.com/richClubb/adventofcode/tree/main/2023/day05/c)

I spend most of my time in C, C++ or Rust these days. Work has me in C++ and Rust and most of the embedded stuff I'm doing is in C, so this was ok. I set myself the challenge to learn more about memory safety in C so I learned to use `valgrind` and `asan` to find memory leaks and errors. I also tried out a source code arrangement where the include and unit tests life alongside the code to make it more modular. I think it makes for a nicer more manageable codebase than having separate `include/` `test/` and `src/` directories in the root of the directory.

The hardest part of this was mostly the file parsing, extracting the data into a sensible format, the rest was pretty easy.

It still reigns supreme in terms of speed, It was marginally faster than both Rust and Zig, but only by 10 seconds (1:30 to 1:40 on average)

### Parallelisation

This was interesting. I started with [OpenMP](https://github.com/richClubb/adventofcode/blob/main/2023/day05/c/part_b/part_b_openmp.c) (look for the `#pragma` directive) and it got the execution time down from 1:30 to 0:30, but performance-wise it wasn't actually very impressive in comparison to OpenCL. The time to implement was about 10 seconds though....

[OpenCL](https://github.com/richClubb/adventofcode/blob/main/2023/day05/c/part_b/part_b_opencl.c) was a different beast however. It's really similar to Cuda in that you have to prepare the environment beforehand and once it was sorted you can execute the code. Debugging was hard, the help available is a bit spotty and difficult to understand, it's not beginner friendly, but once it worked it was great. Got the execution down to 10s with 28 cores. It did take a good few evenings to get right.

Definitely worth looking at both of these if you use C.

### Optimisations

I did have some issues with C when it comes to optimisations. I used the default parameters to optimise the code, but I was stuck at 3:00, while Rust and Zig's best release versions were down at 1:40. I couldn't figure out why. I joined a few Discords and someone on the Zig discord pointed out that it might be the absence of the `-flto` compiler option that means the code isn't inlining functions. Turned this on and it immediately went down to 1:20. I'm going to have to remember this for the future.

## C++

[Code Link](https://github.com/richClubb/adventofcode/tree/main/2023/day05/c%2B%2B)

This was probably one of the most interesting in terms of the side-effects. I chose to use recent version of C++ and used some of the more modern options, specifically the `optional` type, this is something about Rust that I really like. I also used a more OO design which was similar to most other languages.

Initially i wrote the `map_seed` function like so

```
std::optional<uint64_t> SeedMap::map_seed_opt(uint64_t input)
{
    if (
        (input >= this->source) &&
        (input < this->source + this->size)
    )
    {
        return input - this->source + this->target;
    }
    return std::nullopt;
}
```

This allows the calling function to know that the result has been mapped. However with my initial tests this showed a massive difference in comparison to `C`:
* C version - 0:17:06.342
* C++ (optional) - 2:29:25.373

This is nearly a 10x speed difference! Why? I started off by adding `-pg` to the compiler options to allow profiling, this allows the use of gprof to profile the code.

I won't include it here but the majority of the calls were to the optional type or associated function calls which lead me to believe this was the main source of problems. As a reference, the `-pg` compiled version took 18.5s to complete 1,000,000 values.

I decided to write a version which used pointers instead, this took 3s. That's a massive difference.

Why is this? There can't be that much of a difference using optionals, nobody would use them. I plugged this into [compiler explorer](https://godbolt.org/) to play around and realised that depending on the compiler options it completely changes the code that is generated. This tool is great for playing around with your code to see how it optimises.

In the end, with the compiler optimisations turned on both the optional and pointer versions were basically the same, and consistently 20s slower than the pure C version.

Overall this was more pleasant than C as there are niceties that mean you don't have to handle as much memory management, the `vector` type is just so much more pleasant.

### Parallelisation

I parallelised this with OpenMP and it was just as easy as C. I didn't bother to do the OpenCL version in C++ as it wasn't going to be significantly different than the C version.

## Cuda

[Code Link](https://github.com/richClubb/adventofcode/tree/main/2023/day05/cuda)

I have wanted to learn how to use Cuda for a while and this was a good reason to do so. Initially it was a headache to get over the memory management and code setup and how the kernel worked.

It was also necessary for me to write code to split the seed ranges into smaller ranges to maximise the use of the Cuda cores. In total I have 3072 on the RTX 2000 series graphics card on my laptop, with this it was complete in under 10 seconds. I did hear from someone back in 2023 that with a full fat 3080 it was solved in less than a second, I don't know how to quantify this with my version to see if I'm doing anything wrong. I asked people in several communities and they weren't particularly helpful :(

In terms of the code it wasn't appreciably harder than C or C++, but the setup was more annoying to learn, and there were some pitfalls that were hard to debug. In all I'd need to spend some time learning more about the tools to be properly happy.

Definitely worth doing and I'd recommend trying out Cuda if you have an Nvidia graphics card.

## C#

[Code Link](https://github.com/richClubb/adventofcode/tree/main/2023/day05/c_sharp)

This was pretty straightforward. By the time I got to C# I'd done it in 7 other languages so I had the problem pretty down. The main reason I did it was a colleague wanted ammunition in a holy war he was having with his colleague over C# and Java... I definitely didn't want to fan the flames on that one..... :)

The main issue I had was that I wanted to create a nullable type and found it to be a bit frustrating, in the end I just reverted to using the builtin `UInt64` type which is nullable. Oh and the namespace / file naming conventions.... F*** that.

## F#

[Code Link](https://github.com/richClubb/adventofcode/tree/main/2023/day05/f_sharp)

This was an experiment with functional language and had a few unexpected surprises. At the time I started this I was working on [interop between different languages](https://github.com/richClubb/devlog/tree/main/2025-02-28/language_interop) and thought it would be good to experiment with the dotnet frameworks and how they might interact together. Functional programming has some advantages over imperative or OO and being able to farm out tasks to functions with zero / low cost was an appealing idea. But.... it was a pain to learn, the syntax is not friendly and I didn't like it, this might improve with time but initially it was just a bit awkward and didn't pull me in.

The first surprise was that if I tried to solve the problem in a functional way, it ate all of my available memory. My assumption is that it was creating a vector of 1,900,000,000 values for both the input and output vector. F# isn't purely functional, you can use imperative code, by changing the code to use a more imperative design it was able to solve the function without consuming all the available memory.

```F#
    member this.FindMinSeedInRange( mappingLayers : List<MappingLayer> ) = 

        let seeds = [|this.SeedStart .. this.SeedEnd|]

        let processed_values = Array.map (fun seed -> (seed, mappingLayers) ||> List.scan (fun s v -> v.TranslateSeedForward(s))) seeds
        Array.map (fun a -> List.last a) processed_values |> Array.min

    // This works by only keeping track of the minimum value, rather than trying to store the list of all values
    // by using a while loop rather than other iterators it means that it doesn’t eat loads of memory
    member this.FindMinSeedInRangeMutable( mappingLayers : List<MappingLayer> ) = 

        let mutable min_value = Int64.MaxValue
        let mutable seed = this.SeedStart
        while seed <= this.SeedEnd do 
            let seed_val = (seed, mappingLayers) ||> List.scan (fun s v -> v.TranslateSeedForward(s)) |> List.last
            if seed_val < min_value then min_value <- seed_val
            seed <- seed + int64(1)
        min_value
```

There might be ways to get around this but honestly I don't care enough to spend the time figuring it out.

The second interesting thing is that it automatically parallelised the code without having to do anything. This is a nice consequence of using a functional design, which is what I found with Rust and the rayon crate.

I'm unlikely to come back to this unless I have some burning urge. I'd be more tempted to use something like Erlang if I wanted to try another functional language.

## Go

[Code Link](https://github.com/richClubb/adventofcode/tree/main/2023/day05/go)

I have mixed feelings on Go. The language and syntax is good, very Pythonic and I can see why people are so happy with it. It's high enough language so you have a lot of the benefits and it's garbage collected so you don't have to worry about memory management. It's MUCH faster than Python, and it seems to be geared to web services which is 90% of what people do these days.

The coding was easy enough, but the thing that frustrated me was the idioms and the language rules for structure. The fact that capitalising the first letter of a function denotes the visibility of the function is not a nice smell to me, and I found it complaining about how I was naming my packages frustrating. I had a similar frustration in C#.

Overall good, would come back for more.

## Java

[Code Link](https://github.com/richClubb/adventofcode/tree/main/2023/day05/java)

This was basically the same as C# and I only did it to help pour fuel on someone else’s argument, but I have similar complaints.

Although choosing Maven for the package management was a frustration. It took me longer to find information on how to use Maven than it did actually coding the solution, and figuring out the commands to build and package the dependencies with the `jar` file. Bleh.

It was slower than C# and as much as I hate Microsoft with every fibre of my being..... I'd probably still go back to C#, but I'd probably try to do it in Rust first before either of these two.

## Python

[Code Link](https://github.com/richClubb/adventofcode/tree/main/2023/day05/python)

This was my first implementation and again there isn't much to say here. It was fast to develop but fell apart quickly due to performance problems, but this isn't what Python is used for.

It was very quick to prototype new things, and I think it was probably my second strongest language when I started doing this back in 2023, but now I've done more in Rust and C++ I'd probably gravitate to those first. It was the first language I tried parallelising which only took a little tweaking to get working, and handling the range calculation was pretty easy after I got the logic down.

While doing this writeup I actually decided to try and learn more about `ctypes` and it was a really good experience. I wrote a basic library in C which did the heavy computation and python was used to just extract and format the data from the file and pass it into C. It was pretty flexible, I was able to use pointers, custom structs and it only took a few hours to get my head around.

The solution is a bit of a mess and I really need to tidy it up, but it's unlikely to happen. Poetry solves a lot of the package management woes but I never got around to setting it up. It would also be good to try using any of the GPU parallelisation.

## Rust

[Code Link](https://github.com/richClubb/adventofcode/tree/main/2023/day05/rust)

I'm a bit of a Rust fanboy so I really enjoyed this. Cargo is a great system for managing the packages (of which I used none) and it's one of the main points that makes Rust more attractive than C or C++. I was quite new to Rust when I started so I set myself a challenge of doing this in as 'Rusty' a way as possible, minimising mutability and leaning into the functional style that seems to be popular.

The learning curve was pretty short initially, but as I wasn't doing anything complex so I wasn't surprised. Parallelisation was a dream, but this might have been a bit emergent considering how I'd started programming it, if I'd chosen a different pattern then it might have been harder.

I'd recommend learning some Rust, it's fun.

## Zig

[Code Link](https://github.com/richClubb/adventofcode/tree/main/2023/day05/zig)

I also have mixed feelings about Zig. The learning curve is pretty steep as to do anything of any substance and the flux in the interfaces can be a pain. I found some notes on 0.14 but I was using 0.15 and the interfaces had changed so none of the information was relevant.

`build.zig` is very powerful but also a bit difficult to grasp initially. I think more study on my part would make this better, and I like how granular you can be. I like C and C++ as CMake allows you to access quite a lot (depending on how you arrange your CMake) but it also can lead into spaghetti code. With the specificity of `build.zig` you can intuitively see how interconnected your program is as you have to code it in directly.

The allocator behaviour is a bit frustrating and I really think it could do with some examples of how best to use it. Several people in the Zig discord suggested using global variables for the allocators but this seems like a crutch and not a proper solution. It's a cool concept in general but I think it's a bit overkill, and I think it would benefit from the ability to switch out the allocator based on your build type (debug allocator for debug builds and the c allocator for release). It might be possible to do as you can create your own allocator but this is a whole other project.

The general language syntax is... ok, a bit verbose for me but I'm not going to complain too much. This is one of those muscle memory things that will come with time.

Overall I'd suggest looking into it, it's fast and interesting and the community seems to be pretty active. I think it's a worthy successor to C in a lot of ways, and something you should keep your eyes on if you do low-level programming.

# Conclusion

This was a lot of fun and I'd highly recommend doing it. As I said earlier having a set of "pet" problems that you can use as a litmus test for a language is a great idea. Advent of code ia a great way to push yourself and set random challenges to really stretch your skill; try different languages, try a new technique, or just try and complete the whole thing and have that under your belt.

Most people won't have to ever do any crazy computation like this. Typically most problems are web-based dealing with basic input and only a few of them ever hit the "oh s***, we've got to do something millions or billions of times, how do we make it faster!". Search and sorting algorithms are so good that even with millions or billions of entries we can get results fast, and we've got techniques to cache, index and store data to make retrieval almost instant. So what I've done here is probably irrelevant, right? Well no. Everything I've done here has been in service of learning "how" to solve these types of problems and getting that "purposeful practice" that we all sorely need, especially in an industry that forces us to do different things so often (but this is a topic for another day). I've documented as much as I can so that hopefully in the future I can come back to this and say "Oh yea, I've solved that problem before"

I revisited this because I'm trying to complete 12 projects in 12 months, and finishing this off and completing it in a few more languages and techniques seemed like a good idea, but it just spiralled into a whole other thing which I'm pretty glad about. I've got to play around with languages I've not used before, and have a good comparison point between them. I learned a lot about performance profiling, language interop, parallelisation, and I finally got to do something with Cuda! It got me taking a more active role in some computing communities and I think in general made me a better programmer.

I could talk about this for hours.... but I'll stop here :)


