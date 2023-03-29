---
layout: post
title: 'The weirdest compiler bug'
date: "2021-01-26T00:00:00.000Z"
description: "When the same code produces different results in a thread..."
categories: [mingw64, compiler, C++, threading, IEEE-754]
---

There are approximately 7.5x10^18 grains of sand on Earth. This story is about finding changes in an equation that has a difference of approximately 1e-18 out of hundreds of billions of calculations. That is 7 grains of sand that are different to what we expect across the entire planet Earth.

After spending days generating gigabytes of debug logs and GDB breakpoints, I finally discovered a very peculiar bug in the compiler. I thought this would be an interesting story to tell.

**Update: Thanks to gus_massa and wiml @ HackerNews for pointing out I had used associative instead of commutative**

## Background

Back in 2008 I started developing a scientific modelling platform called the [Spatial Population Model (SPM)](https://niwa.co.nz/fisheries/tools-resources/spatial-population-modelling). This software is designed to model or simulate and ocean environments to approximate the health of fish stocks. The output of SPM is used to create scientific reports that are given to Governments for the setting of commercial fishing quota. 

Due to the political nature of these reports, reproducibility and integrity of results is crucial. Scientists from around the world will use the software to re-run models across different Operating Systems and compilers to validate results.

Since it was 2008, it was decided that SPM would not be multi-threaded. This is because of: no C++ standard threading, average computer had 2 cores, incompatibility with auto-differentiation libraries) 

Fast forward to 2020/21 where many-core systems are commonplace. I had recently acquired for myself an AMD 5950X 16c/32t CPU and was keen to apply it to some modelling work. SPM is still in use as a world leading spatial modelling platform and has received continual updates. I've had a long interest in bringing concurrency to SPM to invent new methods of scientific modelling that evolve the underlying mathematics. As a proof of concept, I wanted to start working through threading the internal gradient calculation. Models in SPM are user defined and can be enormous (>100k lines of input text) so we're unable to analytically determine the gradient function through auto-differentiation. We use an iterative approach tweaking model parameters to calculate the gradient. These tweaks are independent and therefore can be parallelised. 

This seemed like a relatively straight forward piece of work. I would need to:
1. Move all of the classes to be children of a new Model class
2. Remove all singletons
3. Keep the floating-point operations in the exact same order when calculate on the main thread (IEEE-754)
4. Support an arbitrary number of threads
5. Produce identical results regardless of the number of threads, Operating System or compiler

## IEEE-754 or the floating-point nightmare

The development of scientific modelling platforms like SPM requires a significant amount of thought and effort around reproducibility. Given the same input files, any user must be able to re-run your model and get the exact same answer out. This seems incredibly obvious, but it's not....

[IEEE-754](https://en.wikipedia.org/wiki/IEEE_754) is the standard for floating-point arithmetic. Floating point math in computers is an approximation. Whenever you see a floating point value it is highly unlikely to be completely accurate. Floating-point arithmetic is commutative, but not associative. So for floating-point numbers,
A + B = B + A always, but
A + (B + C) != (A + B) + C in many cases.
The order in which operations occur within your model will influence the output. While this is inconsequential for small programs or games; this continual adding of errors for scientific models where the number of operations is in the hundreds of billions is consequential. 

Every thread we spawn must run the same operations in the exact same order. When the threads give their results back to the main thread, they must do so in a way that ensures that all future operations happen in the exact same order. This is regardless of the number of threads or number of parameters in the model.

_Side Note: Within scientific models, we use double precision numbers (double) and not single precision (float). Single precision does not have enough precision to handle the number of calculations required. At the end of a model the error added by the approximation of floating-point math is very significant. This limits us to using CPUs over GPUs as the double precision performance of GPUs is not that much better than a modern CPU when you have to factor in the added complexity of writing GPU specific code._

## The first signs of trouble

With my task list ready, I started to work through re-factoring the code. Everything was modified to use a central model class as the parent for the system. This allows me to spawn as many model classes as threads. Everything was compiling and running without crashes. Time to check the output..
```
1999.818926297566804 // original score running no threads
1999.8189264475995515 // my new score running 32 threads
```
A slight difference, nothing to be concerned about as I had probably changed the order of execution for some of the equations by threading them. I was testing with a small model that ran in <5 seconds, so working through each of the instantiated classes to check the code won't take too long. Another day down and nothing obvious was found. Time to run the application through GDB and the [Sanitizers](https://en.wikipedia.org/wiki/AddressSanitizer) looking for issues.... nope nothing there either.

A simple model will have recruitment/breeding, ageing and death. I started to reduce and simplify the processes in the model to see if I could identify any key process or parameter. I removed the random number generate and commented out a large amount of complex math.... but still no luck.

## Down the rabbit hole of debug logging and GDB

After three days of trying to find the issue, I had added a LARGE amount of debug logging and had resorted to doing step throughs in GDB looking for a point at which I would notice a change in the result. The log files were 20MBs+ and the model would only show issues after a few iterations... why not immediately?

Was I calling a function later in the model that had the issue? Or was the issue starting earlier but at a greater precision that I was printing. I was printing my debug output with a [precision](https://en.cppreference.com/w/cpp/io/manip/setprecision) of 15. Time to increase the precision to 20. This is basically the maximum amount of precision you can print from a double with accuracy. I did a few more model runs and saw that the variation in results had moved to much earlier in the model... almost immediately.

The model would load the configuration file, then construct all of the user defined objects in memory. As part of constructing these objects a bunch of calculations would be run to build caches. To save on moving data between threads each thread would repeat the same process. Every new thread would get the configuration file that had been loaded, create all of the objects and run the initial calculations. GDB showed that even these initial calculates were having different results.

My working folder was littered with files named "fuck", "fuck1" and "fuckN" each with more than 20MBs of debug output for a small model run. Diff was run across files to see where changes were occurring and how small they were... But still no luck.. everything was going wrong almost immediately but I had no idea why.

## Surely it's not a compiler bug?

SPM uses the GCC and MinGW64 compilers. This allows us to have near-identical code for Windows and Linux with common makefiles. I have always preferred [TDM-GCC](https://jmeubank.github.io/tdm-gcc/) as this has a current release. In this instance, TDM-GCC 9.2.0 from March 2020. Running the sanitizers was done on OpenSuSe Tumbleweed Linux using GCC 10.2.1. These are both very modern versions of the GCC compiler suite. I have never been one to blame tools for a bug in my code.

At this point, I am way down the rabbit hole trying to find the cause of this bug. Some testing had shown me that:
- The original binary would produce the same result on every run
- My new binary would produce the same result on every run, but different to the original
- My new binary would produce the same result regardless of the number of threads.. from 1 to 100

That last observation made me curious. Why did my new code produce a different result when I ran it with only one thread? What would make it different from the original binary. Having only 1 thread that was executing would eliminate any race conditions.

In the original binary, we do not create any threads. We load the model, build and run it. In the new code, even with one thread we create that thread, load the model, build and run it. What was I doing during the thread creation process that would cause this issue? Time for GBs of log files and GDB.

Stepping through every operation to validate the output lead me into what we call a selectivity. This is a static piece of code that takes an input, calls a standard simple piece of math and returns an output. These few lines of code had returned different results in a thread and not. Time to analyse the following code:
```c++
double CLogisticSelectivity::calculateResult(int Age) {
    double dRet = 0.0;
    double dTemp = (dA50-Age)/dAto95;

    if(dTemp > 5.0)
      dRet = 0.0;
    else if (dTemp < -5.0)
      dRet = dAlpha;
    else
      dRet  = dAlpha/(1.0+pow(19.0,dTemp));

    return dRet;
}
```
If I gave the code Age = 1 then ran it I got the following results:
```
Local  = 0.0010370292068795884059
Thread = 0.0010370292068795879722
```

What gives? There is nothing in here that should be different in a thread. I am not passing in anything but an int with the value of 1. I edited the code to make dA50 and dAto95 local variables to remove any potential race conditions or thread wonkiness and still had the same error in output. Everything was initialized within the same function and the only external value was an Integer. Weird....

## Compiler bug? But how...

Looking at the code, the only things that could create the bug were the operators and pow() call. These are part of the C++ standard offering so it's highly unlikely that either of these would have an error. What next? Off to [GodBolt](https://godbolt.org/) to try some other compilers to see if I get wonky results on them. MSVC++... nope, Clang... nope, GCC-Trunk on Linux... nope. I am only getting this weird behaviour in TDM-GCC. Time to try some other MingW64 compilers. [Nuwen](https://nuwen.net/mingw.html)... Yes, [Mingw64](http://mingw-w64.org/doku.php).. yes, TDM-GCC... yes. 

Confirmed: It's a bug in MinGW64.

When I create a new thread and run floating point operations in that thread, I get slightly different answers. 

## Getting it fixed.. well soon...

Off to SourceForge I went to log a [bug](https://sourceforge.net/p/mingw-w64/bugs/867/). After waiting a month and getting no response I jumped on IRC to ask the same question. Within 24 hours my issue had a comment indicating that it had been [fixed](https://sourceforge.net/p/mingw-w64/mingw-w64/ci/295fafcf584ce105426045b03d17aa90d105808d/). This had not yet picked up by any of the pre-built distributions. 

New threads were not calling [\_fpreset()](https://docs.microsoft.com/en-us/cpp/c-runtime-library/reference/fpreset?view=msvc-160) when they were constructed, which "reinitializes the floating-point math package". This would mean that the default values used within floating point arithmetic would be slightly different when you were inside a thread.

As there have not been any downstream versions of MinGW64 that have this fix, I have had to resort to calling \_fpreset() within the threads myself. A dirty but, but effective fix.

All n all. I spent around 5 days chasing this bug through my code. I generated Gigabytes of log files and had to get down to the precision of 7.5 grains of sand on the planet Earth. The compiler missing a key function call turned out to be the cause of the issue. Many times, while trying to find the root cause I found myself questioning my ability to write code, diagnose bugs and remain sane. I'm glad I found an answer and have a way forward.

When the code was tested with an AMD 5950X and 32 threads, solving a model went from 23.5 hours to 90 minutes. This was a substantial improvement and this work alone provides significant new opportunities for iterating through models when developing scientific reports.
