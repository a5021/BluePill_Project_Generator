# BluePill Project Generator

[![MIT License](https://img.shields.io/badge/License-MIT-green.svg)](https://choosealicense.com/licenses/mit/)
[![Build](https://img.shields.io/github/actions/workflow/status/a5021/BluePill_Project_Generator/build.yml?label=Build&logo=github)](https://github.com/a5021/BluePill_Project_Generator/actions/workflows/build.yml)
[![CodeQL](https://img.shields.io/github/actions/workflow/status/a5021/BluePill_Project_Generator/ci.yml?label=CodeQL&logo=github)](https://github.com/a5021/BluePill_Project_Generator/security/code-scanning)

**Bare-metal STM32F103 (Blue Pill) project generator** — one POSIX shell script, one `make`, zero bloat. Generates a complete C project with CMSIS headers, startup code, linker script, and Makefile, ready to compile and flash.

## Features

- **One-command project** — `sh create_stm32f1_project`
- **15 MCU peripherals** available (RCC, SysTick, USART, I2C, SPI, ADC, DMA, RTC, IWDG, WWDG, TIM2, EXTI, CRC, FLASH, SWD)
- **HCLK-independent timings** — SysTick, USART baud, I2C timing, TIM2 prescaler all computed from `HCLK`; works at 8/64/72/112 MHz unchanged
- **DMA** — circular for TIM2→GPIO (LED breathing), one-shot for USART TX
- **RTC** — selectable LSE/LSI source, second counter mode
- **I2C** — repeated START, timeouts, bus recovery on error (PE cycle)
- **OZONE debugger config** — `project.jdebug` with SWD, SVD, ELF auto-setup
- **PRNG (`USE_RNG`)** — xoshiro128** software generator; not a hardware or cryptographic RNG. ADC samples may contribute when ADC is enabled; TIM2 jitter is not used.
- **BTN** — PB0/PB1 (BUTTON 1/2, input with pull-up, debounced via 32-sample shift register)
- **`lock_firmware()`** — optional readout and write protection via STM32 option bytes
- **Optimized startup + linker script** (`sh create_stm32f1_project opt`) — block-transfer `.data`/`.bss` (ldmia/stmia × 8 words); computed `_estack` (8-byte AAPCS alignment), `ASSERT` checks against malloc/C++ misuse, explicit `.ARM.exidx` placement, zero-heap safety
- **0 warnings** at `-Wall -Werror -Wextra -Wpedantic -O3 -flto`

## Why this generator?

- No GUI dependency
- No HAL/LL abstraction
- No hidden project files
- Transparent Makefile
- Bare-metal startup and linker script
- Optional IDE configs when needed
- Peripheral code controlled by simple `USE_*` flags

## Requirements

| Tool | Version |
|------|---------|
| POSIX shell (`sh`) | Any |
| arm-none-eabi-gcc | ≥ 12 (tested with 15.2.1) |
| wget | Any |
| make | Any |

Optional tools: `st-flash` or `ST-LINK_CLI` for `make program`, SEGGER J-Flash for
`make jprogram`, and Doxygen for `make docs`.

## Quick start

```bash
git clone https://github.com/a5021/BluePill_Project_Generator.git
cd BluePill_Project_Generator
sh create_stm32f1_project
```

## Usage

```bash
sh create_stm32f1_project [opt] [mdk] [ses] [vsc] [pdf]
```

| Argument | Effect |
|----------|--------|
| *(none)* | Generate project with CMSIS startup, build with `make` |
| `opt` | Use custom optimized startup (block-transfer .data/.bss) |
| `mdk` | Generate Keil MDK-ARM project (`project.uvprojx`) |
| `ses` | Generate SEGGER Embedded Studio project (`project.emProject`) |
| `vsc` | Generate VSCode workspace (`tasks.json`, `launch.json`, `c_cpp_properties.json`) |
| `pdf` | Download RM0008 reference manual + STM32F103C8 datasheet PDFs |

Arguments can be combined: `sh create_stm32f1_project opt mdk vsc`

## Generated project structure

```
.
├── main.c                    # Application entry, init + demo loop
├── main.h                    # All peripheral configuration macros
├── Makefile                  # GNU Make build system
├── startup_stm32f103xb.s     # Startup code (CMSIS or opt)
├── system_stm32f1xx.c        # CMSIS system initialization
├── project.jdebug            # OZONE debugger configuration
├── STM32F103XB_FLASH.ld      # Linker script
├── cmsis_compiler.h          # CMSIS support headers (downloaded)
├── cmsis_gcc.h
├── cmsis_version.h
├── core_cm3.h
├── stm32f1xx.h               # Device header
├── stm32f103xb.h             # STM32F103xB register definitions
├── system_stm32f1xx.h
├── build/
│   ├── project.elf           # ELF binary
│   ├── project.hex           # HEX (for flashing)
│   └── project.bin           # Binary (for flashing)
└── STM32F103xx.svd            # SVD file (downloaded to the project root)
```

CMSIS, device-support, and SVD source files are fetched from GitHub at pinned
commit revisions; they are placed in the project root rather than a `cmsis/`
subdirectory.

The generator writes files into the **current directory** and overwrites files
with matching generated names. Run it in a directory intended for the project.

## Target memory and Flash capacity

The generated linker and IDE configurations target the STM32F103xB memory map
with 128 KiB of Flash. The official STM32F103C8 specification guarantees only
64 KiB. Some chips marked C8 may expose an additional 64 KiB, but ST does not
guarantee this capacity for the C8. Using addresses above 64 KiB on a C8 is
unsupported and at your own risk; images that use the extra area may not run on
all C8 devices. Keep the image within 64 KiB for guaranteed C8 compatibility.

## Peripherals

| Peripheral | Configuration |
|------------|---------------|
| **HSE/PLL** | 72 MHz by default; 112 MHz overclock is opt-in with `OVRCLK=1` |
| **SysTick** | 1 ms period |
| **USART1** | TX (PA9) / RX (PA10), 115200 baud |
| **SPI1** | SCK (PA5) / MISO (PA6) / MOSI (PA7), full-duplex master |
| **I2C1** | SCL (PB6) / SDA (PB7), 100 kHz standard mode |
| **ADC1** | Internal channels (Vrefint on CH17, continuous scan) |
| **DMA1** | CH2 (TIM2 UP) circular, CH4 (USART1 TX) normal |
| **TIM2** | ~33 Hz update, DMA-triggered BSRR for LED breathing |
| **RTC** | LSE or LSI, second counter |
| **IWDG** | ~1.6 s timeout (LSI 40 kHz / 16 prescaler) |
| **WWDG** | Window watchdog, early-wakeup interrupt, ~65 ms window at 72 MHz (PCLK/4096), disabled by default |
| **EXTI** | PA8 falling edge (external interrupt) |
| **CRC** | Default polynomial |
| **FLASH** | 2 wait states (72 MHz), prefetch enabled |
| **SWD** | PA13/PA14 preserved |

> **Flash demo warning:** With `USE_FLASH=1` (enabled by `make full` by default),
> the demo erases and rewrites the last Flash page once on every firmware startup.
> Repeated resets consume that page's erase-cycle endurance; do not store user
> data on this page. Disable the demo feature with `make full USE_FLASH=0`.

Additionally:
- **BTN** — PB0/PB1 (BUTTON 1/2, input with pull-up, debounced via 32-sample shift register)

`USE_RNG` provides a software pseudo-random generator (xoshiro128**), not a
cryptographic random-number source. The F103 has no hardware RNG; the generator
mixes peripheral/memory state, may sample ADC noise when ADC is enabled, and
may mix in the RTC counter when ADC is disabled and RTC is enabled. The current
implementation does not use TIM2 jitter as an entropy source.

## Flash protection (RDP/WRP)

`lock_firmware()` is an optional STM32 readout-protection operation, enabled at
compile time with `LOCK_FIRMWARE`. It programs option bytes for RDP Level 1 and
Flash write protection, then resets the MCU. RDP Level 1 restricts debug/readout
access; returning to RDP Level 0 requires a mass erase, which erases the
application and stored Flash data. Write-protected pages also cannot be updated
normally. Enable this only after validating the final firmware and accepting
that recovery requires erasing the device.

For example, `make full EXT="-DLOCK_FIRMWARE=1"` enables the operation in the
generated demo. It is not enabled by default.

## Build system

```bash
make              # Minimal build: HSI at 8 MHz; peripherals and SWD disabled
make full         # Build with USE_ALL defaults; 72 MHz by default
make debug        # Debug symbols (-Og -g3), SWD enabled; otherwise minimal config
make program      # Flash via st-flash (Unix) or ST-LINK_CLI (Windows)
make jprogram     # Flash via J-Link (requires SEGGER J-Flash)
make docs         # Generate Doxygen documentation
make clean        # Remove build artifacts
make gccversion   # Show compiler version
```

### Make variables

Peripherals and features can be toggled from the command line:

```bash
make USE_ALL=1 USE_I2C=0 USE_SPI=0   # All peripherals except I2C and SPI
make USE_USART=1 USE_I2C=1           # Minimal: only USART and I2C
make full OVRCLK=1                   # Opt in to 112 MHz overclock (HSE × 14)
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
make program   # st-flash (Unix) or ST-LINK_CLI (Windows)
```

## Hardware references

- [STM32F103C8 datasheet](https://www.st.com/resource/en/datasheet/stm32f103c8.pdf)
- [RM0008: STM32F1 reference manual](https://www.st.com/resource/en/reference_manual/cd00171190.pdf)
- [PM0056: Cortex-M3 programming manual](https://www.st.com/resource/en/programming_manual/cd00228163.pdf)
- [PM0075: STM32F1 flash programming manual](https://www.st.com/resource/en/programming_manual/cd00283419.pdf)

## Other useful links

- STM32CubeF1 (CMSIS headers) — [Repo](https://github.com/STMicroelectronics/STM32CubeF1)
- cmsis_device_f1 (device headers, startup, system) — [Repo](https://github.com/STMicroelectronics/cmsis_device_f1)
- SVD (device description) — [Repo](https://github.com/cmsis-svd/cmsis-svd-data)
