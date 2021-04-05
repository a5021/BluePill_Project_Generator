# BluePill Project Generator

Shell script to generate simple Makefile project for "BluePill" STM32F103C8 devboard.

Run the script in any convenient way.

`sh create_stm32f1_project`

It will fetch all necessary files from STMicro CMSIS repo, generate main.h/main.c sources and build them with GNU ARM toolchain. Linux and Windows (cygwin required) are supported.

*Optionally the script can be run with parameter `mdk` to generate config for Keil MDK-ARM.*

To upload firmware to the "BluePill" microcontroller simply type:

  `make program`
  
*Note: st-link programmer and st-link utility are required*



If it is nessesary to rebuild sources with debug information supplied just run make utility as follows:

  `make debug`
  
  

It is possible to compile firmware with macros predefined. It can be done with Makefile variable named EXT. For instance run

  `make EXT="-DUSE_PLL=1 -DUSE_HSE=1"`
  
to build firmware with HSE generator switched ON and PLL selected as system clock. The list of macros can be found at the top of main.h

