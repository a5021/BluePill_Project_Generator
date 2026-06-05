# BluePill Project Generator

[![MIT License](https://img.shields.io/badge/License-MIT-green.svg)](https://choosealicense.com/licenses/mit/)

**Bare-metal STM32F103 (Blue Pill) project generator** — one bash script, one `make`, zero bloat. Generates a complete C project with CMSIS headers, startup code, linker script, and Makefile, ready to compile and flash.

## Features

- **One-command project** — `bash create_stm32f1_project`
- **14 MCU peripherals** pre-configured (RCC, SysTick, USART, I2C, SPI, ADC, DMA, RTC, IWDG, TIM2, EXTI, CRC, FLASH, SWD)
- **HCLK-independent timings** — SysTick, USART baud, I2C timing, TIM2 prescaler all computed from `HCLK`; works at 8/64/72/112 MHz unchanged
- **DMA** — circular for TIM2→GPIO (LED breathing), one-shot for USART TX
- **RTC** — selectable LSE/LSI source, second counter mode
- **I2C** — repeated START, timeouts, bus recovery on error (PE cycle)
- **OZONE debugger config** — `project.jdebug` with SWD, SVD, ELF auto-setup
- **RNG** — entropy from multiple analog sources (ADC noise, RTC phase, TIM2 jitter)
- **BTN** — PB0/PB1 (BUTTON 1/2, input with pull-up, debounced via 32-sample shift register)
- **`lock_firmware()`** — sets RDP Level 1 to disable debug access
- **Optimized startup** (`bash create_stm32f1_project opt`) — block-transfer `.data`/`.bss` (ldmia/stmia × 8 words), minimal footprint (7.4 KB text)
- **0 warnings** at `-Wall -Werror -Wextra -Wpedantic -O3 -flto`

## Requirements

| Tool | Version |
|------|---------|
| bash | 4+ |
| arm-none-eabi-gcc | ≥ 12 (tested with 15.2.1) |
| curl | Any |
| make | Any |

## Quick start

```bash
git clone https://github.com/a5021/BluePill_Project_Generator.git
cd BluePill_Project_Generator
bash create_stm32f1_project
```

## Usage

```bash
bash create_stm32f1_project [opt] [mdk] [ses] [pdf]
```

| Argument | Effect |
|----------|--------|
| *(none)* | Generate project with CMSIS startup, build with `make` |
| `opt` | Use custom optimized startup (block-transfer .data/.bss) |
| `mdk` | Generate Keil MDK-ARM project (`project.uvprojx`) |
| `ses` | Generate SEGGER Embedded Studio project (`project.emProject`) |
| `pdf` | Download RM0008 reference manual + STM32F103C8 datasheet PDFs |

Arguments can be combined: `bash create_stm32f1_project opt mdk`

## Generated project structure

```
.
├── main.c                    # Application entry, init + demo loop
├── main.h                    # All peripheral configuration macros
├── Makefile                  # Build system (cross-platform)
├── startup_stm32f103xb.s     # Startup code (CMSIS or opt)
├── system_stm32f1xx.c        # SystemInit() stub
├── project.jdebug            # OZONE debugger configuration
├── STM32F103XB_FLASH.ld      # Linker script
├── build/
│   ├── project.elf           # ELF binary
│   ├── project.hex           # HEX (for flashing)
│   └── project.bin           # Binary (for flashing)
└── cmsis/                    # CMSIS headers (downloaded)
```

## Peripherals

| Peripheral | Configuration |
|------------|---------------|
| **HSE/PLL** | 72 MHz system clock (overclock to 128 MHz via `OVRCLK`) |
| **SysTick** | 1 ms period |
| **USART1** | TX (PA9) / RX (PA10), 115200 baud |
| **SPI1** | SCK (PA5) / MISO (PA6) / MOSI (PA7), full-duplex master |
| **I2C1** | SCL (PB6) / SDA (PB7), 100 kHz standard mode |
| **ADC1** | CH0 (PA0), continuous scan |
| **DMA1** | CH2 (TIM2 UP) circular, CH4 (USART1 TX) normal |
| **TIM2** | 100 Hz update, DMA-triggered BSRR for LED breathing |
| **RTC** | LSE or LSI, second counter |
| **IWDG** | ~4 s timeout (LSI 40 kHz / 256 prescaler) |
| **EXTI** | PA0 falling edge |
| **CRC** | Default polynomial |
| **FLASH** | 2 wait states (72 MHz), prefetch enabled |
| **SWD** | PA13/PA14 preserved |

Additionally:
- **RNG** — software entropy from ADC noise, RTC phase, and TIM2 jitter
- **BTN** — PB0/PB1 (BUTTON 1/2, input with pull-up, debounced via 32-sample shift register)

## Build system

```bash
make              # Build with minimal configuration (all peripherals disabled)
make test         # Build with -DUSE_ALL=1 (enable all peripherals)
make debug        # Build with debug symbols (-Og -g3) and SWD enabled
make program      # Flash via ST-Link
make jprogram     # Flash via J-Link (requires SEGGER J-Flash)
make clean        # Remove build artifacts
make gccversion   # Show compiler version
```

### Make variables

Peripherals and features can be toggled from the command line:

```bash
make USE_ALL=1 USE_I2C=0 USE_SPI=0   # All peripherals except I2C and SPI
make USE_USART=1 USE_I2C=1           # Minimal: only USART and I2C
make OVRCLK=1                        # Overclock to 112 MHz (HSE × 14, ADCCLK = 14 MHz)
```

Available variables: `USE_ALL`, `USE_PLL`, `USE_HSE`, `USE_LSE`, `USE_ADC`,
`USE_USART`, `USE_I2C`, `USE_SPI`, `USE_BTN`, `USE_RTC`, `USE_TIM2`,
`USE_DMA`, `USE_CRC`, `USE_RNG`, `USE_IWDG`, `USE_WWDG`, `USE_EXTI`,
`USE_FLASH`, `USE_SWD`, `OVRCLK`.

Set a variable to `1` to enable (e.g. `USE_I2C=1`) or `0` to disable
(e.g. `USE_HSE=0`). Command-line values override `#ifndef` defaults from
`main.h`; they do **not** override explicit `#define` written directly into
`main.h`.

Extra compiler flags can be passed via `EXT`:

```bash
make EXT="-save-temps -ftime-report"
```

## Programming

Connect ST-Link to the Blue Pill:

| ST-Link | Blue Pill |
|---------|-----------|
| SWCLK   | PA14 (SWCLK) |
| SWDIO   | PA13 (SWDIO) |
| GND     | GND |
| 3.3V    | 3.3V |

```bash
make program   # ST-Link utility
```

## Hardware references

* [STM32F103C8 datasheet](https://www.st.com/resource/en/datasheet/stm32f103c8.pdf)
* [RM0008: STM32F1 reference manual](https://www.st.com/resource/en/reference_manual/cd00171190.pdf)
* [PM0056: Cortex-M3 programming manual](https://www.st.com/resource/en/programming_manual/cd00228163.pdf)
* [PM0075: STM32F1 flash programming manual](https://www.st.com/resource/en/programming_manual/cd00283419.pdf)

## Other useful links

- STM32CubeF1 (CMSIS headers) — [Repo](https://github.com/STMicroelectronics/STM32CubeF1)
- cmsis_device_f1 (device headers, startup, system) — [Repo](https://github.com/STMicroelectronics/cmsis_device_f1)
- SVD (device description) — [Repo](https://github.com/cmsis-svd/cmsis-svd-data)
