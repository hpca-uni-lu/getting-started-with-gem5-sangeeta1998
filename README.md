[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-24ddc0f5d75046c5622901739e7c5dd533143b0c8e959d652212380cedb1ea36.svg)](https://classroom.github.com/a/E0ftO9Jx)
[![Open in Codespaces](https://classroom.github.com/assets/launch-codespace-7f7980b617ed060a017424585567c406b6ee15c891e84e1186181d67ecf80aa0.svg)](https://classroom.github.com/open-in-codespaces?assignment_repo_id=13926400)
# Introduction to the gem5 computer system simulator
Mats Brorsson

## Acknowledgments
This assignment has been adapted from the original assignment published for [UC Davis course in computer architecture](https://jlpteaching.github.io/comparch/) with permission from [Prof Jason Lowe-Power](https://cs.ucdavis.edu/directory/jason-lowe-power).

## Table of Contents

- [Introduction](#introduction)
- [gem5 and gem5’s standard libary](#gem5-and-gem5s-standard-library-gem5-stdlib)
- [Writing your own configuration script](#writing-your-own-configuration-script)
- [Workload and gem5-resources](#workload-and-gem5-resources)
- [Final step: simulate](#final-step-simulate)
- [Invoking gem5](#invoking-gem5)
- [Hints](#hints)

# Introduction

In this assignment, we will use gem5’s standard library to simulate the Hello World program in C++ on gem5. You will learn how to write your own configuration scripts to describe the computer system you need to simulate and pass your workloads and benchmark to the simulator.

## gem5 and gem5’s standard library (gem5-stdlib)

gem5 is a well known event based simulator for computer architecture research. You will learn about simulation and its different techniques in class and during the discussions. The gem5 simulator is designed so that is very powerful in describing widely varying systems, but this means it is also quite complex. With the purpose of simplifying system description, gem5’s standard library has been developed to allow users to use ready made components and user friendly interfaces to describe their system. The gem5 standard library (stdlib) is the digital world equivalent of going to your local BestBuy and picking the different components you need to build your computer system. Although, the components along with their names do not match the commercial products of Intel, AMD, and Nvidia.

## `gem5.components.boards`

Each system in gem5 is described as a `board`. You can think of this board as the motherboard in a computer system. In this assignment we are going to be using a ready made board in gem5’s standard library that we have renamed to `HW0RISCVBoard`. This board will use a RISC-V-based You can find the definition of `HW0RISCVBoard` in this repo in `components/boards.py`. You can see that it is based on SimpleBoard from the standard library. You can find the SimpleBoard source code and documentation in `gem5/src/python/gem5/components/boards/simple_board.py`.

Similar to a real system, a system in gem5 is not complete with only a board. However, the board is the platform through which all the other components communicate with each other. In order to build a complete system we need 3 other components: a processor, a cache hierarchy, and a memory. This is not exactly the same way you would build a real computer system. For real computers, cache hierarchy comes pre-packaged with the processor and you need other components such as a power supply and a storage drive to complete a real system. However, there are many research focuses on cache hierarchy design that has motivated this separation of processor and cache hierarchy.

In addition, you need to specify the clock frequency for the processor and cache hierarchy to the board.

## `gem5.components.processors`

Ready made processor objects in the standard library represent the processing cores in a real CPU. In this assignment, we are going to use `HW0TimingSimpleCPU`. You can find its source code in this repo in `components/processors.py`. This processor is based on `SimpleProcessor` from the standard library. Whenever instantiated, it will create a `SimpleProcessor` with one `TimingSimpleCPU` core with RISC-V instruction set architecture (ISA). You can find the source code to SimpleProcessor in `gem5/src/python/gem5/components/processors/simple_processor.py`. In addition, it is highly recommended that you learn more about gem5’s different CPU models in the link below.

- [SimpleCPU](https://www.gem5.org/documentation/general_docs/cpu_models/)
- [MinorCPU](https://www.gem5.org/documentation/general_docs/cpu_models/minor_cpu)
- [O3CPU](https://www.gem5.org/documentation/general_docs/cpu_models/O3CPU)

## `gem5.components.cachehierarchies`

The cache hierarchy objects in standard library are designed to implement cache coherency protocol for multi-core processors and model the interconnect between multiple cores and multiple memory channels. In this assignment we are going to use `HW0MESITwoLevelCache`. You can find its source code in `components/cache_hierarchies.py`. This cache hierarchy is based on `MESITwoLevelCacheHierarchy` from the standard library. You can find the source code to `MESITwoLevelCacheHierarchy` in `gem5/src/python/gem5/components/cachehierarchies/ruby/mesi_two_level_cache_hierarchy.py`. Whenever instantiated, a    `HW0MESITwoLevelCache` creates a two level cache hierarchy with the MESI protocol for cache coherency[^1]. It has a 64KiB 8-way set associative L1 instruction cache, 64KiB 8-way set associative L1 data cache, 256KiB 4-way set associative L2 unified cache with 16 banks.

[^1]: MESI and cache coherence will be introduced in a later assignment, don't worry too much about it now.

## `gem5.components.memory`

The memory objects in standard library are designed to implement various types of single and multi-channel DRAM based memories. In this assignment we are going to use `HW0DDR3_1600_8x8`. You can find its source code in `components/memories.py`. This memory is based on `ChanneledMemory` from the standard library. You can find the source code to `ChanneledMemory` in `gem5/src/python/gem5/components/memory/memory.py`. Whenever instantiated, a `HW0DDR3_1600_8x8` creates a single channel DDR3 DRAM memory with 1GiB of capacity and a data bus frequency of 1600MHz.

# Writing your own configuration script

We will write configuration scripts that describe our desired system to be simulated in python. We will go through the following steps to complete our configuration script. Before that, let’s create a script called **run.py** in the assignment’s base directory. We will keep adding to this script until it is complete. Then we will pass this script to the gem5 binary for simulation.

## Import models

We will need to import the models for the different components that we aim to simulate. Here is the code that imports all the mentioned models for HW0.

```
from components.boards import HW0RISCVBoard
from components.processors import HW0TimingSimpleCPU
from components.cache_hierarchies import HW0MESITwoLevelCache
from components.memories import HW0DDR3_1600_8x8
```

## Instantiate an object of the model

In this step we will create an object of each model. We will first create a processor and name it **cpu**, then we will create a cache hierachy and name it **cache**, then we will create a memory and name it **memory**, and then we will create a board and name it **board**. We will need cpu, cache, and memory to create a board. Here is the code that does that for us.

```
if __name__ == "__m5_main__":
    cpu = HW0TimingSimpleCPU()
    cache = HW0MESITwoLevelCache()
    memory = HW0DDR3_1600_8x8()
    board = HW0RISCVBoard(
        clk_freq="2GHz", processor=cpu, cache_hierarchy=cache, memory=memory
    )
```

# Workload and gem5-resources

So far, we have written a configuration script that describes the system we would like to simulate. However, our simulation setup is not complete. We still need to describe what software needs to be executed on the hardware that we just described. In computer architecture research frequently used programs and kernels (small pieces of code essential to many programs e.g. quick sort) are used to evaluate the performance of a computer system. These programs usually come in a package and are referred to as benchmarks. There are many benchmarks available. SPEC2017 and PARSEC are among the popular benchmarks in computer architecture research.

gem5 resources is a project aiming to offer ready made resources compatible with the gem5 simulator. You can download and use compiled binaries from many benchmarks and small programs from gem5-reosources.

In this assignment, as mentioned before, we are going to use an already compiled Hello World binary for the RISC-V ISA from gem5-resources. You can find the source code to `HelloWorldWorkload` in `workloads/hello_world.py`.

## Importing workload

We need to import our `HelloWorldWorkload` to our configuration script. To do that add the following line to your import section of the code.

```
from workloads.hello_world_workload import HelloWorldWorkload
```

## Setting the workload for simulation

Next, we will need to create an object of the workload we just imported and describe that this workload object is the object that we want to use for the software on our specified hardware. You can do that by calling `set_workload` function from `HW0RISCVBoard`. Here are the lines of code that does that.

```
workload = HelloWorldWorkload()
board.set_workload(workload)
```

Here is how the configuration script looks like so far.

```
from components.boards import HW0RISCVBoard
from components.processors import HW0TimingSimpleCPU
from components.cache_hierarchies import HW0MESITwoLevelCache
from components.memories import HW0DDR3_1600_8x8

from workloads.hello_world_workload import HelloWorldWorkload

if __name__ == "__m5_main__":

    cpu = HW0TimingSimpleCPU()
    cache = HW0MESITwoLevelCache()
    memory = HW0DDR3_1600_8x8()
    board = HW0RISCVBoard(
        clk_freq="2GHz", processor=cpu, cache_hierarchy=cache, memory=memory
    )
    workload = HelloWorldWorkload()
    board.set_workload(workload)
```

# Final step: simulate

The last step before our configuration script is complete is to create a simulator object. The simulator object allows us to give directions to gem5 on specific things we need gem5 to do for simulation. You will see examples of interacting with the simulator later on. We can create a simulator object through an internal python package from gem5. The simulate package allows user to easily set up the simulation environment for the experiments. NOTE: Most of gem5’s internal python packages work exclusively with gem5. gem5 has an internal modified python interpreter that these packages use to communicate with gem5. 

The following steps show how to import the simulator package and instantiate a simulator object.

## Import simulator

In order to import the simulator package, add the following line to you configuration script.

```
from gem5.simulate.simulator import Simulator
```

## Create a simulator object

Next, we will create a simulator object. In order to create a simulator object we need to specify what the system to be simulated is and what mode of simulation should be used. We have already described the system to be simulated in the previous steps. We will need to only pass **board** as the representative for the whole system. In regards to simulation modes, gem5 works in two simulation modes. The two modes are referred to as _Full System_ (FS) mode and _Syscall Emulation_ (SE) mode. FS mode is like “bare metal” simulation which requires a kernel and disk image to boot Linux and extra script to then run some application. In SE mode, gem5 “fakes” the system calls and they take 0 time. However, SE mode doesn’t require a disk or kernel image, it only requires a binary (usually statically compiled). So SE mode is a little easier to get going than FS mode.

We will be using SE mode for this assignment. This means that we need to pass value False as the `full_system` argument to our simulator. Here is the piece of code that creates a simulator object.

```
simulator = Simulator(board=board, full_system=False)
```

The last action item is to tell gem5 to run the simulation. Make a call to `run` function from the simulator to do that. Here is the piece of code that calls this function.

```
simulator.run()
```

Let’s add a print statement to show the simulation is over. Add the following line to your configuration script.

```
print("Finished simulation.")
```

Here is how the script looks like after adding all the necessary statements.

```
from components.boards import HW0RISCVBoard
from components.processors import HW0TimingSimpleCPU
from components.cache_hierarchies import HW0MESITwoLevelCache
from components.memories import HW0DDR3_1600_8x8

from workloads.hello_world_workload import HelloWorldWorkload

from gem5.simulate.simulator import Simulator

if __name__ == "__m5_main__":

    cpu = HW0TimingSimpleCPU()
    cache = HW0MESITwoLevelCache()
    memory = HW0DDR3_1600_8x8()
    board = HW0RISCVBoard(
        clk_freq="2GHz", processor=cpu, cache_hierarchy=cache, memory=memory
    )
    workload = HelloWorldWorkload()
    board.set_workload(workload)
    simulator = Simulator(board=board, full_system=False)
    simulator.run()

    print("Finished simulation.")
```

# Invoking gem5

As discussed earlier, gem5 has a modified internal python interpreter. Please note that passing your configuration scripts to python will result in numerous confusing errors. You will need to pass your configuration script to the gem5 binary to run your simulation. To do that you will need to use the command line. Now, open a terminal and run the command below. Remember: We initially named our configuration script as `run.py`. If you have not done so, use any name you gave your configuration script instead of `run.py` in the command line.

```
gem5 run.py
```

Note> the `gem5` command is defined for the Codespace in github. If you have downloaded gem5 and built it locally, the name of the command will be different.

After running this command in the terminal, you will see an output like below:

```
gem5 Simulator System.  https://www.gem5.org
gem5 is copyrighted software; use the --copyright option for details.

gem5 version [DEVELOP-FOR-22.1]
gem5 compiled Nov 22 2022 23:54:40
gem5 started Jan 11 2023 22:17:47
gem5 executing on codespaces-50d4d9, pid 14760
command line: gem5 run.py

Resource 'riscv-hello' was not found locally. Downloading to '/root/.cache/gem5/riscv-hello'...
Finished downloading resource 'riscv-hello'.
warn: The simulate package is still in a beta state. The gem5 project does not guarantee the APIs within this package will remain consistent across upcoming releases.
Global frequency set at 1000000000000 ticks per second
warn: failed to generate dot output from m5out/config.dot
warn: failed to generate dot output from m5out/config.board.cache_hierarchy.ruby_system.dot
build/ALL/mem/dram_interface.cc:690: warn: DRAM device capacity (8192 Mbytes) does not match the address range assigned (1024 Mbytes)
build/ALL/base/statistics.hh:280: warn: One of the stats is a legacy stat. Legacy stat is a stat that does not belong to any statistics::Group. Legacy stat is deprecated.
0: board.remote_gdb: listening for remote gdb on port 7000
build/ALL/sim/simulate.cc:197: info: Entering event queue @ 0.  Starting simulation...
build/ALL/mem/ruby/system/Sequencer.cc:613: warn: Replacement policy updates recently became the responsibility of SLICC state machines. Make sure to setMRU() near callbacks in .sm files!
build/ALL/sim/syscall_emul.hh:1014: warn: readlink() called on '/proc/self/exe' may yield unexpected results in various settings.
      Returning '/root/.cache/gem5/riscv-hello'
build/ALL/sim/mem_state.cc:443: info: Increasing stack size by one page.
Hello world!
Finished simulation.
```

You can see that our print statement has produced the output that we exptected. If you see an output like the one above, congratulations. You just completed your first simulation with gem5.

Now, if you run ls in your terminal you will see the following output.

```
LICENSE  LICENSE.code  LICENSE.content  README.md  components  gem5  m5out  requirements.txt  run.py  util  workloads
```

You might notice that a directory named `m5out` has been created after invoking gem5. This directory includes all the simulator output files. You can find the statistics output under `m5out/stats.txt`. This file includes many statistics about your simulated hardware. The file is human readable. Please take the time to take a look at the first 15 lines of that file and understand what each statistic means. NOTE: host statistics refer to statistics about the actual machine that the simulation was run on and guest/simulated statistics refer to the statistics of the simulated computer system.

## Redirecting simulator output

Like every other program gem5 uses standard out stdout and standard error stderr to communicate some of it information with the user. However, you can tell gem5 to not dump stdout and stderr to your terminal and output those to a file. To do this add `-r` flag to your previous command you used to run your simulation. Make sure to add this flag before the name of your configuration script as the flag should be passed to gem5 and not your configuration script. Below is how the command looks like after adding `-r` flag to it. Remember: We initially named our configuration script as `run.py`. If you have not done so, use any name you gave your configuration script instead of `run.py` in the command line.

```
gem5 -r run.py
```

This is what you will see in your terminal after running the command above.

```
Redirecting stdout and stderr to m5out/simout
```

Now, if you look at `m5out`, you will see that there is a new file name `simout`. Let’s print the content of that file and compare to our previous output in Invoking gem5. To do that, run the following command in your terminal.

```
cat m5out/simout
```

Below you can see the output after running the command above.

```
gem5 Simulator System.  https://www.gem5.org
gem5 is copyrighted software; use the --copyright option for details.

gem5 version [DEVELOP-FOR-22.1]
gem5 compiled Nov 22 2022 23:54:40
gem5 started Jan 11 2023 22:31:34
gem5 executing on codespaces-50d4d9, pid 19657
command line: gem5 -r run.py

warn: The simulate package is still in a beta state. The gem5 project does not guarantee the APIs within this package will remain consistent across upcoming releases.
Global frequency set at 1000000000000 ticks per second
warn: failed to generate dot output from m5out/config.dot
warn: failed to generate dot output from m5out/config.board.cache_hierarchy.ruby_system.dot
build/ALL/mem/dram_interface.cc:690: warn: DRAM device capacity (8192 Mbytes) does not match the address range assigned (1024 Mbytes)
build/ALL/base/statistics.hh:280: warn: One of the stats is a legacy stat. Legacy stat is a stat that does not belong to any statistics::Group. Legacy stat is deprecated.
0: board.remote_gdb: listening for remote gdb on port 7000
build/ALL/sim/simulate.cc:197: info: Entering event queue @ 0.  Starting simulation...
build/ALL/mem/ruby/system/Sequencer.cc:613: warn: Replacement policy updates recently became the responsibility of SLICC state machines. Make sure to setMRU() near callbacks in .sm files!
build/ALL/sim/syscall_emul.hh:1014: warn: readlink() called on '/proc/self/exe' may yield unexpected results in various settings.
      Returning '/root/.cache/gem5/riscv-hello'
build/ALL/sim/mem_state.cc:443: info: Increasing stack size by one page.
Hello world!
Finished simulation.
```

You can see that apart from the date and time of simulation and the missing gem5-resources statement for download `riscv-hello`, the two outputs look similar. However, this file includes both stdout and stderr. You separate stdout and stderr into two files. To do that pass `-re` instead of `-r` to gem5. This is what the command will look like after passing the new flags. 

```
gem5 -re run.py
```

After running the command above, this is what you will see in your terminal.

```
Redirecting stdout to m5out/simout
Redirecting stderr to m5out/simerr
```

Compare the content of `m5out/simout` and compare that to your previous output. 

Then  look at the content of m5out/simerr. To do that, run the following command in your terminal.


# Changing the output directory

Since gem5 uses the same name `m5out` for its output directory every time you run gem5, the simulator output will be overwritten every time you run gem5. However, gem5 allows you to manually change this directory to a directory of your liking. To do that, you have to add `--outdir=[path to your desired directory]` to your command that you use to invoke gem5. Make sure to pass it to gem5 and not your configuration script. This means you have to put `--outdir=[path to your desired directory]` before the path to your configuration script. This is what an example command with `--outdir` will look like.

```
gem5 --outdir=test-run0 run.py
```

After running the command above, you will see that a directory with the name `test-run0` has been created. Let’s confirm that by running `ls`. This is what you will see in your terminal after running `ls`.

```
LICENSE  LICENSE.code  LICENSE.content  README.md  components  gem5  m5out  requirements.txt  run.py  test-run0  util  workloads
```

If you look at the content of `test-run0`, you will see that it includes all the simulator outputs that you could find in `m5out` before.

## Passing input arguments to configuration script

gem5 allows you to pass input arguments to your configuration script. This way you will not need to create a configuration script for every system you want to simulate. You can pass your input arguments to your configuraion script following the path to your configuration script. gem5 will then capture that part of the command line and pass it to your configuration script. You will then have to parse that part of the command line to read your input arguments from the command line. Below is how you can invoke gem5 with the flags we discussed before and input arguments to your conifuration script.

```
gem5 [-re] {path to your configuration script} [input arguments to your configuration script]
```

I highly recommend you to read up on [argparse](https://docs.python.org/3/library/argparse.html). It is a feature rich python package that allows you to parse the command line.

# Hints
Start early and ask questions in the appropriate Moodle forum 
