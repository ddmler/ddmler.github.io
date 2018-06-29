---
layout: post
title:  "Profile-guided optimization"
date:   2018-06-29 18:00:00 +0100
categories: compiler
description: A look at compiler optimization techniques beyond -o3 or how Google Chrome got an up to 15% performance increase by just changing their compilation process.
---

Profile-guided optimization (PGO) also known as feedback-directed optimization (FDO) is a compiler optimization technique that provides even better performance gains than the well known -o3 optimization flag in gcc. It is however a bit more work than just setting a flag in the compilation. 

[Google Chrome uses PGO][chrome] since version 53 and they saw an up to 15% performance increase for Chrome on Windows. Broken down in different parts of Chrome they got a 14.8% faster new tab page load time, 5.9% faster page load and 16.8% faster startup time. All these performance gains not from optimizing their millions of lines of code, but from using a compiler optimization. Pretty impressive and this works for every program. 

Likewise the [GCC wiki][autofdo] shows a simple bubble sort example which shows a performance increase of 3.46% of PGO over -o3 and more complex code experiments show an increase of almost 9%. They also have a tutorial on PGO and AutoFDO.

As we will see there are two ways to use PGO. The first one is instrumentation based and works as follows: First build the program with an instrumentation flag. The resulting program will gather runtime data that will be used in a second build to optimize the parts of the program that are often used. The other way to use PGO uses perf to gather runtime profile data, which makes the process of gathering data a lot simpler (see below at the AutoFDO section).

One important thing to keep in mind is that the runtime data on which the binary will get optimized has to be a somewhat representative of the actual usage to improve performance. If this is not the case it can also harm the performance. This is also the reason why AutoFDO was created, since it can collect the runtime data from your normal end users instead of a special instrumented build.

[Microsofts Build Reference][optimizations] has a list of optimizations that are performed by PGO for example reordering if and else blocks depending on which is more likely to be true.


#### How to use it

This example uses GCC, but your favorite compiler probably supports the same features.

The first step as we said before is to create an instrumented version of our binary that will gather the runtime data:

`gcc program.c -o instrumented -fprofile-generate`

The `-fprofile-generate` flag will instrument our binary. The next step is to run the program to gather some profile data.

`./instrumented`

To get started we just run it once, but normally you would run it multiple times with different input and so on to create a somewhat representative sample of your actual usage patterns. This will create a new gcda file in this case it's called `program.gcda`. We will use this in our next step as the profile:

`gcc program.c -O3 -o fdo -fprofile-use=program.gcda`

The resulting binary should be a bit more efficient than the normal -O3 version. Next let's have a look at how to use real usage data from your end users as the profile instead of having to simulate the usage.

#### AutoFDO

[AutoFDO][autofdo_github] is a project aiming to make the needed work for PGO simpler by removing the need for instrumenting the program to gather runtime data. Instead it uses perf and converts that data with a tool to the correct format. The main difference between AutoFDO and normal PGO is, that AutoFDO collects its runtime data on an optimized binary (for example in production).

The first step in this case is to collect our perf data, since we already have a binary running in production. For this we can either use perf or ocperf from the [pmu-tools project][pmu]. The reason to use the latter is that we need to capture the `BR_INST_RETIRED:TAKEN` event in the processor which changes with different CPU architectures. ocperf is a wrapper that will take care of these differences so we don't have to worry about it. A recording with ocperf could look like this:

`ocperf.py record -b -e br_inst_retired.near_taken:pp -- ./program`

The next step is to use the create_gcov tool provided by the [AutoFDO project][autofdo_github]. This will convert the perf data into a gcov file that we can use as our profile for the compiler:

`create_gcov --profile=perf.data --gcov=program.gcov --binary=./program`

Now just use the gcov file as our profile (this time with another option called `fauto-profile`):

`gcc program.c -O3 -o autofdo -fauto-profile=program.gcov`

And we're done. For the next version/build of your program just collect perf data from production and on your next build you will just have to create a new gcov file and use it as the new profile.


#### Improving PHP performance with PGO?

So can we also improve PHP performance with PGO? Yes. We can instrument (or with AutoFDO collect perf data) our PHP binary and let it run our PHP project to collect data. With instrumentation it would work like this: First instrument the binary. Next execute `php index.php` or multiple different pages. Try to execute the parts of your PHP project that are executed most often in production. Lastly use that data to get a PHP binary that is optimized for your PHP app.

With AutoFDO you can collect perf data while clicking your way through your website like a real user would, or let the perf data be collected in production.


[chrome]: https://blog.chromium.org/2016/10/making-chrome-on-windows-faster-with-pgo.html
[autofdo]: https://gcc.gnu.org/wiki/AutoFDO/Tutorial
[autofdo_github]: https://github.com/google/autofdo
[optimizations]: https://docs.microsoft.com/en-us/cpp/build/reference/profile-guided-optimizations
[pmu]: https://github.com/andikleen/pmu-tools
