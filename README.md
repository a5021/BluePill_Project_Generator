# BluePill Project Generator

[![MIT License](https://img.shields.io/badge/License-MIT-green.svg)](https://choosealicense.com/licenses/mit/)

This is a simple shell script to generate a bare-metal Makefile project for the "Blue Pill" STM32F103C8 development board.

Run the script in any convenient way:

  ```console
  sh create_stm32f1_project
  ```

The script will fetch all necessary CMSIS files from the STMicro repository, generate main.h and main.c source files, and build them using the GNU ARM toolchain. Both Linux and Windows (with Cygwin) are supported.

Optionally, the script can be run with the parameter mdk to generate a Keil MDK-ARM project.

  ```console
  sh create_stm32f1_project mdk
  ```

If you specify the 'pdf' parameter at the command line, the script will retrieve the description and datasheet from the STMicro website.

  ```console
  sh create_stm32f1_project pdf
  ```

After the sources have built successfully, to upload firmware to the 'Blue Pill' microcontroller, simply type:

  ```console
  make program
  ```
  
*Note: An ST-Link programmer and the ST-Link utility are required.
                                                                 
If it is necessary to rebuild the sources with debug information, just run the make utility as follows:

  ```console
  make debug
  ```
  
Configuration macro names (those from the top of main.h) can be used as command line parameters for the make utility:

  ```console
  make USE_ALL=1 USE_HSE=0 USE_PLL=0
  ``` 
  
In the example above, all configuration macros are enabled except for USE_HSE and USE_PLL, which are disabled.

## Hardware references

* [STM32F103C8 datasheet](https://www.st.com/resource/en/datasheet/stm32f103c8.pdf)
* [RM0008: STM32F1 reference manual](https://www.st.com/resource/en/reference_manual/cd00171190.pdf)
* [PM0056: Cortex-M3 programming manual](https://www.st.com/resource/en/programming_manual/cd00228163.pdf)
* [PM0075: STM32F1 flash programming manual](https://www.st.com/resource/en/programming_manual/cd00283419.pdf)

