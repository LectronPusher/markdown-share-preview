# EECS 470 Early Labs/Projects

This readme will be included in the first few labs/projects, and is a
beginner's guide and quick reference for getting started in EECS 470.

It starts with a guide to accessing the CAEN Linux environment, gives a
quick refresher on using the terminal and shell, introduces GitHub and
autograder usage in EECS 470, and ends by introducing our general build
tool and Makefile workflow.

## Accessing the CAEN Linux Environment

[CAEN's Linux environment](https://caen.engin.umich.edu/software/clse/)
is our workspace for verilog programming in 470. It gives us access to
the Synopsys build tools and the Verdi debugger and is the only location
we can use these tools from. Luckily, there are many options for
accessing the environment:

#### [Lab computers!](https://caen.engin.umich.edu/software/clse/)

Lab computers are the simplest way to access CAEN Linux - and they don't
require two-factor authentication (2FA).

Go to any CAEN lab (try searching at
[this link](https://its.umich.edu/computing/computers-software/campus-computing-sites/computer-labs-map)
by checking "CAEN Workstations") and open a computer:

- If your login screen is for Linux, you're done!

  Use your uniqname and password to log in and access the Linux desktop,
  then open a terminal with Ctrl+T or by right-clicking the desktop and
  selecting "Open Terminal"

- Windows computers will require a few more steps:

  - Most lab computers are dual boot, so you can often reboot straight
    into the Linux desktop:

    Press Ctrl+Alt+Del at the Windows login screen and select restart,
    then use the arrow keys to choose Linux when the option to choose an
    OS comes up

  - Or you can connect to a remote Linux desktop inside Windows by using
    CAEN VNC:

    Log in to Windows, then search for "CAEN VNC" in the "appsanywhere"
    page that automatically opens. Click the green launch button, then
    the second big green launch button in the cloudpaging player window.
    It will open a small window asking for your log in. Enter it, then
    log in again to access the Linux desktop!

#### [Remote login!](https://caen.engin.umich.edu/connect/linux-login-service/)

You can still log into the CAEN Linux environment if you're not on
campus, however you will need to use two-factor authentication with Duo
every single time you access, which gets annoying. These tutorials will
show you how to access either the desktop through VNC or the terminal
through SSH.

- [Log in to a desktop via VNC](https://teamdynamix.umich.edu/TDClient/76/Portal/KB/ArticleDet?ID=4999)
- [Log in to a terminal via SSH](https://teamdynamix.umich.edu/TDClient/76/Portal/KB/ArticleDet?ID=5002)
  - And don't forget to set up an ssh config file (see below)

#### [Visual Studio Code!](https://code.visualstudio.com/docs/remote/ssh#_installation)

VS Code is a common editor and offers a great way to access projects in
470. You can use the Remote-SSH extension to access CAEN over SSH from a
VS Code instance running on your local machine. If you follow the
[linked tutorial](https://code.visualstudio.com/docs/remote/ssh#_installation)
above, CAEN is already ready for ssh with the hostname
`login-course.caen.umich.edu` and your username will be your uniqname,
so connect to the remote: `YOUR_UNIQNAME@login-course.caen.umich.edu`.
Unfortunately this too requires 2FA on every login, but you can set up
ControlPersist in your ssh config (see below) to speed up reconnecting.

### Configuring SSH

Most students will eventually want to access CAEN remotely, and will
probably use ssh to do so. Setting up an ssh config file can save you a
lot of hassle over the semester. Below is a good default ssh config for
CAEN, it lets you type `ssh caen` instead of
`ssh YOUR_UNIQNAME@login-course.caen.umich.edu` and also keeps your ssh
connection alive so you don't have to reconnect and redo 2FA if you
connect multiple times in a row. You can use it by copying it to the
`~/.ssh/config` file, or if you're using VS Code, search for "remote-ssh
configuration" and add it there.

```
Host caen login-course.engin.umich.edu
    HostName login-course.engin.umich.edu
    User YOUR_UNIQNAME
    ControlMaster auto
    ControlPath ~/.ssh/_%r@%h:%p
    ControlPersist yes
    ForwardX11 yes
```

## Using the Terminal and Shell

Once you can access the CAEN Linux environemnt, you can open a terminal
and run some shell commands. Open the terminal by pressing Ctrl+T or
right-clicking the desktop and selecting "Open Terminal"

We assume you have basic terminal and shell knowledge already, but very
quickly, here's what to remember:

- Each line is a command with space separated arguments, use quotes for
  arguments that contain spaces:  
  `grep EECS 470 Makefile` vs `grep "EECS 470" Makefile`
- Press the up arrow to reuse previous commands
- Press tab to auto-complete typed filenames (`grep 470 Makef<TAB>`)
- Use `pwd` to print your working folder/directory, `cd new_dir` to
  change your directory, `mkdir new_dir` to make new directories, and
  `touch new_file`to make new files
- Output files raw with `cat`, view them with `less`:  
  `cat README.md` vs `less README.md`
- Redirect command output to files with `>`:  
  `echo EECS 470 rules! > file.txt`; `cat file.txt`
- Pipe the output of one command as the input to another with `|`:  
  `echo "EECS 370 was a lot of work" | sed -e s/3/4/ -e s/wa/i/`
- Press Ctrl+C to Cancel a running command  
  Press Ctrl+\\ to force Quit a running command if Ctrl+C doesn't work
- Finally, read manuals on any command with `man command`, and get quick
  help with `command --help`

The specific build tools used in 470 (Makefile usage, running
testbenches) are gone over in detail below.

## GitHub Guide

TODO

Once you have shell access, clone your repos and get started!

## Build Tools in EECS 470

In EECS 470 we're generally executing one of three types of commands in
CAEN:

1.  Compiling verilog testbenches with simulated and synthesized modules
2.  Running the compiled executable
3.  Debugging in Verdi, our verilog debugging environment

#### Here's our general workflow:

We start with a verilog testbench and module(s) it tests.

We compile these under simulation to an exectuable file that runs the
testbench and prints whether the module passes or fails. If it fails, we
use Verdi or display statements to inspect the output and iterate on
either the testbench or the module until we're satisfied.

Once the testbench and module work in simulation, we change our
compilation step. We synthesize the verilog modules against actual
device cells representing structural hardware connections. We only
synthesize the module, not the testbench, as the testbench may include
features like *printing*, or *reading files* that don't exactly
translate to raw silicon.

In synthesized modules, we generally focus on timing rather than area,
and use the **slack** metric: the clock period minus the longest signal
path from one register to another. If slack is positive, a signal might
not reach its next register by the clock edge, leaving incorrect values
floating and causing potential *undefined behavior* (scary!).

So when we synthesize modules, we check the slack of the compilation,
and either increase the clock period or change the design if we find
violations. When we're satisfied with the synthesis, we can compile the
synthesized module with the original testbench to produce a new
executable that we can debug again until it passes.

### Using Makefiles in EECS 470

To manage this workflow, we use *GNU Make* as a build automation tool.
We specify targets with dependencies and set variables in the special
file "`Makefile`". At the shell, we run `make my_target` to compile or
run a target and all of its dependencies. Make also only rebuilds
targets or dependencies if their source files are newer than the
previous build.

From the workflow above, we name our normal simulation executable
`simv`, and our synthesis executable `syn_simv`. To build these, we need
to specify our source and output files in these Make variables:

- `TESTBENCH` `= and8_test.sv` Our verilog testbench for a module
  (`and8` in this case)
- `SOURCES` `= and8.sv and4.sv` The source files for the modules in the
  testbench
- `SYNTH_FILES` `= and8.vg` The file(s) we'll use when compiling for
  synthesis

For simulation we compile `simv` directly from `SOURCES` and
`TESTBENCH`. For synthesis we first synthesize `SOURCES` to
`SYNTH_FILES` and then compile `syn_simv` from `SYNTH_FILES` and
`TESTBENCH`.

We won't go over the specific build commands in detail, but our verilog
compiler is `vcs`, which we run with many many arguments, and our
synthesizer is `dc_shell`, which we run with a synthesis script:
`470synth.tcl`. The script reads environment variables for source files
and other configuration and defines the constraints that set up our
timing requirements.

To use the Makefile, run `make my_target` in the shell when in a
directory with a "`Makefile`" file. Here is the reference table of all
standard targets in EECS 470 makefiles:

```
# make sim       <- execute the simulation testbench (simv)
# make simv      <- compiles simv from the testbench and SOURCES

# make syn       <- execute the synthesized module testbench (syn_simv)
# make syn_simv  <- compiles syn_simv from the testbench and *.vg SYNTH_FILES
# make *.vg      <- synthesize the top level module in SOURCES for use in syn_simv
# make slack     <- a phony command to print the slack of any synthesized modules

# make verdi     <- runs the Verdi GUI debugger for simulation
# make syn_verdi <- runs the Verdi GUI debugger for synthesis

# make clean     <- remove files from testbench runs and compilations (but not synthesized modules)
# make nuke      <- remove all files created by the makefile
# make clean_run_files <- remove per-run output files
# make clean_exe       <- remove compiled executable files
# make clean_synth     <- remove generated synthesis files
```
