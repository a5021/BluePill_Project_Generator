# BluePill Project Generator

[![MIT License](https://img.shields.io/badge/License-MIT-green.svg)](https://choosealicense.com/licenses/mit/)

This is a simple shell script to generate a bare-metal Makefile project for the "Blue Pill" STM32F103C8 development board.

Run the script in any convenient way:

  ```console
  sh create_stm32f1_project
  ```

The script will fetch all necessary CMSIS files from the STMicro repository, generate main.h and main.c source files, and build them using the GNU ARM toolchain. Both Linux and Windows (with Cygwin) are supported.

Optionally, the script can be run with the parameter `mdk` to generate a Keil MDK-ARM project.

  ```console
  sh create_stm32f1_project mdk
  ```

If you specify the `pdf` parameter at the command line, the script will retrieve the description and datasheet from the STMicro website.

  ```console
  sh create_stm32f1_project pdf
  ```

After the sources have built successfully, to upload firmware to the 'Blue Pill' microcontroller, simply type:

  ```console
  make program
  ```
  
_Note: An ST-Link programmer and the ST-Link utility are required._
                                                                 
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

## Other useful links

- STM32CubeF1 (CMSIS headers)
  - Repo: https://github.com/STMicroelectronics/STM32CubeF1
  - Raw CMSIS include path used by the script:
    https://raw.githubusercontent.com/STMicroelectronics/STM32CubeF1/master/Drivers/CMSIS/Include/

- cmsis_device_f1 (device headers, startup, system files for STM32F1 series)
  - Repo: https://github.com/STMicroelectronics/cmsis_device_f1
  - Raw files used by the script (examples):
    - https://raw.githubusercontent.com/STMicroelectronics/cmsis_device_f1/master/Include/stm32f1xx.h
    - https://raw.githubusercontent.com/STMicroelectronics/cmsis_device_f1/master/Include/stm32f103xb.h
    - https://raw.githubusercontent.com/STMicroelectronics/cmsis_device_f1/master/Include/system_stm32f1xx.h
    - https://raw.githubusercontent.com/STMicroelectronics/cmsis_device_f1/master/Source/Templates/system_stm32f1xx.c
    - https://raw.githubusercontent.com/STMicroelectronics/cmsis_device_f1/master/Source/Templates/gcc/startup_stm32f103xb.s
    - https://raw.githubusercontent.com/STMicroelectronics/cmsis_device_f1/master/Source/Templates/gcc/linker/STM32F103XB_FLASH.ld

- SVD (device description) used by the script:
  - Repo: https://github.com/cmsis-svd/cmsis-svd-data
  - Raw file: https://raw.githubusercontent.com/cmsis-svd/cmsis-svd-data/refs/heads/main/data/STMicro/STM32F103xx.svd
