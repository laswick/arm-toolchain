# README

## ARM Toolchain Project

Rob Laswick  
May 10 2013  
[www.laswick.net](http://www.laswick.net)

---

> **Project Status: Inactive**

> This repository will remain available but is no longer maintained.

---

## Welcome!

This project simply includes a set of **easily customizable** scripts that automatically download, configure, patch, build, and install the entire ARM toolchain (including `binutils`, `gcc`, `newlib`, `gdb`, and `openocd`). 

Currently the scripts only support the Cortex M3, Cortex M4, and Cortex M4F.  However, support for other ARM cores can be added relatively easily.

`build_toolchain` is the top level "master" script that calls all the other scripts, although the individual scripts can be run stand alone as well. 

Before running `build_toolchain` there are 2 parameters at the top of the file you'll want to edit:

`WORKING_DIR` is the location where the toolchain will be built.  Once it's built and you're happy with it, this directory can be deleted _(it's actually quite large, ~2.6GB)_.

`PREFIX` is the location where the final toolchain will be installed.

You can also specify the exact version of each of the tools you want your toolchain to use, which is pretty cool.

If your development host is setup correctly you can simply run `./build_toolchain` and walk away!  A full build takes approximately 2 hours.


***
## So what _exactly_ does this toolchain get me?

You get all the standard GNU tools and utilities with support for multilibs, a multilib and reentrant C library, the gdb debugger, and openocd (an awesome open source gdb server with support for a large number of hardware interfaces).


***
## So what doesn't this toolchain get me?

Like most non-commercial toolchains, you don't get any processor specific header files and start code, or any hardware specific linker scripts.

It's important to note, that as long as you know what you're doing, _or have the drive and curiosity to find out_, these _key_ files are _not_ impossible to write yourself _(despite what your mother told you)_.  In fact, I provide **all** of these files for all the projects on [laswick.net](http://www.laswick.net) _(take that mom!)_.


***
## Using the toolchain

#### Adding the tools to your PATH
The first thing you'll likely want to do is add the location of the tools to your PATH environment variable.  This is typically done by adding a line similar to the following to your _shell profile_ config file (i.e. `.bash_profile` or `.profile` depending on the distro you're running):

        PATH="$PATH:$HOME/opt/arm-toolchain/bin:$HOME/opt/arm-toolchain/openocd/bin"

You'll need to run `. ~/.profile` (or `. ~/.bash_profile`) in each shell for this update to take effect _(or log out and back in again)_.


#### Makefile Considerations

There are Makefiles provided for all the projects on [laswick.net](http://www.laswick.net), but a few key things to keep in mind when using Makefiles are:

- I typically reserve the file `Makefile` _(note the uppercase 'M')_ as a _project template_ Makefile.

- I typically reserve the file `makefile` _(note the lowercase 'm')_ to be used as a  `symlink` to the specific Makefile being used (i.e. flashLed.mak).  Since `make` searches for `makefile` and then `Makefile` _(in that order)_, your `symlinked` Makefile will be used.  This allows you to simply run `make` to build your project, rather than having to type out `make -f flashLed.mak` everytime.

- Ensure you pass the `-nostartfiles` flag when linking to prevent the tools from trying to include generic `start code` which you don't want.

- **Do not** link the core libraries (`libgcc` and `libc`) explicitly.  Link using `gcc` instead of `ld` to let `gcc` determine which _version_ of each library to pull in _(you'll need to pass in your `CPU_FLAGS` (i.e. -mcpu=cortex-m4 -mthumb, etc) to `gcc` when linking for this to work correctly)_.

Also, for convenience, I **highly** recommend `symlinking` the key files produced by a successful build as `out.*`, i.e.:

        out.s   : disassembly listing
        out.hex : hex record
        out.elf : full image
        out.map : map file
   

#### GDB Init Scripts

It's often a good idea to use a `.gdbinit` script whenever you're using `gdb`.  The `.gdbinit` script is simply a small text file, _generally_ located in the directory you're working in, and _typically_ used to automate a few tasks such as:

- connecting to the gdb server (`openocd`).

- setting breakpoints on the standard default locations (i.e. the `assert()` stub, and your `default_interrupt_handler`).

- defining a `reset` command for your target.

A `.gdbinit` script could look similar to:

        target remote localhost:3333
        b _deafult_fault_handler
        b __assert_func
        define reset
        monitor reset init
        end


#### Flash Memory Programming with gdb

gdb's `load` command programs an image into memory on the target board.

In some cases when using `gdb` with `openocd`, gdb's `load` command is unable to program the FLASH memory on the target.  One way to work around this annoyance is to override the `load` command by adding the following to the end of your `.gdbinit` script:

        define load
        monitor flash write_image erase "out.axf" 0 elf
        reset
        end

This works quite well, but keep in mind this now binds the `load` command to exclusively programming the FLASH.  If this is an issue for you _(i.e. you need support for RAM builds)_, don't redefine `load`, simply use another name, like `prog`, etc.





