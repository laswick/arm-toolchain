# README

## ARM Toolchain Project

Rob Laswick  
May 10 2013  
[www.laswick.net](http://www.laswick.net)


***
## Welcome!

This project simply includes a set of easily customizable scripts to automatically download, patch, build, and install the entire ARM toolchain (including `binutils`, `gcc`, `newlib`, and `openocd`). 

Currently the scripts only support the Cortex M3, Cortex M4, and Cortex M4F.

Support for other ARM cores can be added relatively easily.

`build_toolchain` is the top level "master" script that calls all the other scripts.  However, the individual scripts can be run stand alone as well.  
  If you're development host is setup correctly you can simply run `./build_toolchain` and walk away!  A full build takes approximately 2 hours.



