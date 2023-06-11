# BluePill Project Generator

Simple shell script to generate bare-metal Makefile project for "Blue Pill" STM32F103C8 devboard.

Run the script in any convenient way.

`sh create_stm32f1_project`

It will fetch all necessary CMSIS files from STMicro repo, generate main.h/main.c sources and build them with GNU ARM toolchain. Linux and Windows (cygwin required) are supported.

*Optionally the script can be run with parameter `mdk` to generate Keil MDK-ARM project.*

To upload firmware to the "BluePill" microcontroller simply type:

  `make program`
  
*Note: st-link programmer and st-link utility are required*


If it is nessesary to rebuild sources with debug information supplied just run make utility as follows:

  `make debug`
  
Configuration macro names (those from top of main.h) can be used as command line parameters for the make utility.

  `make USE_ALL=1 USE_HSE=0 USE_PLL=0` 
  
In the example above all configuration macros are enabled except for USE_HSE and USE_PLL which are disabled.

## Hardware references

* [STM32F103C8 datasheet](https://www.st.com/resource/en/datasheet/stm32f103c8.pdf)
* [RM0008: STM32F1 reference manual](https://www.st.com/resource/en/reference_manual/cd00171190.pdf)
* [PM0056: Cortex-M3 programming manual](https://www.st.com/resource/en/programming_manual/cd00228163.pdf)
* [PM0075: STM32F1 flash programming manual](https://www.st.com/resource/en/programming_manual/cd00283419.pdf)

