# Switch OC Suite

[![License: GPL v2](https://img.shields.io/badge/License-GPL_v2-blue.svg)](https://www.gnu.org/licenses/old-licenses/gpl-2.0.en.html) [![Join the chat at https://gitter.im/Switch-OC-Suite/community](https://badges.gitter.im/Switch-OC-Suite/community.svg)](https://gitter.im/Switch-OC-Suite/community?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

Overclocking suite for Nintendo Switch™ Horizon OS (HOS) running on Atmosphere CFW.

This project will not be actively maintained or regularly updated along with Atmosphere CFW.


## DISCLAIMER: USE AT YOUR OWN RISK!

- Overclocking in general (often combined with overvolting and overheating) will _degrade internal components_ - SoC, VRM(Voltage Regulator Module), Battery, etc. - _faster_ than you and the manufacturer have expected.

- Higher RAM clocks without proper timings could be UNSTABLE and cause graphical glitches / instabilities / filesystem corruption. **Always make backup before usage.**

- Why no CPU/GPU OC for Erista?
  <details>

  - Tegra X1 on Erista is on TSMC 20nm HPM node, consumes much more power (~2x) and generates much more heat, compared to Tegra X1+ on Mariko (TSMC 16nm FinFET).
    - Erista Switch uses lower speedo (=== lower quality === higher voltage required) SoC from NVIDIA. You will NOT get comparable performance to NVIDIA Shield TV no matter what.
    - Snapdragon 810 (4 x A57 @ 2.0GHz + 4 x A53) also uses 20nm HPM, see how it plagued Android phones in 2014.

  - The board power supply is quite limited, even if you've done cooling mod.
    - You could spot battery draining at higher clocks under stress test, even with official 39W PD charger.
    - CPU / GPU performance at max clocks will be worse if power supply is not enough.

  - CPU OC (up to ~ 2.1 GHz, depending on your CPU bin) is available mainly for emulation, but it does NOT work out of the box.

  </details>


## Features

- **DRAM Overclock**

  - Most games are **bottlenecked by RAM bandwidth**

  - Safe:
    - Mariko: 1996.8 MHz has been tested stable for all (Samsung / Micron / Hynix), with built-in timing auto-adjustment.
    - Erista: 1862.4 MHz.

  - Unsafe: higher than 1996.8 MHz or overvolting
    <details>

    - Timing:
      - Timing parameters could be auto-adjusted (default) or overwritten with user-provided mtc table.
      - Customization: No GUI tool, requires [rebuilding](#Build).

    - DRAM bus overvolting (Erista Only).
      - Mariko: [use this to set DRAM bus voltage](https://gist.github.com/KazushiMe/6bb0fcbefe0e03b1274079522516d56d).

    </details>

- **[System Settings (Optional)](https://github.com/KazushiMe/Switch-OC-Suite/blob/master/system_settings.md)**

- **TinyMemBenchNX**: DRAM throughput and latency test based on [tinymembench](https://github.com/ssvb/tinymembench)

- **MemTesterNX**: A userspace utility for testing DRAM faults and stability based on [memtester](https://pyropus.ca/software/memtester/)
  - Now with multi-thread support and "stress DRAM" option, it should be able to test DRAM stability with adjusted timings.


### Mariko Only

- **CPU/GPU Overclock**

  - Safe: CPU/GPU @ 1963/921 MHz
    - It has been proved safe without charger (not reaching battery power draw threshold)

  - Unsafe: CPU/GPU up to 2397/1305 MHz
    - **Disabled by default**, toggle "Allow Unsafe Frequencies" on in overlay or add `allow_unsafe_freq=1` to `config.ini`
    <details>

    - Without chargers, CPU/GPU would be capped @ 1963/921 MHz or -/460 MHz (Emulator mode).

    - Without official chargers, GPU would be capped @ 1267 MHz.

    - Why **NOT RECOMMENDED**?
      - See `Current Flow` in sys-clk-OC overlay `Miscellaneous` (on battery) or measure power draw from charger yourself.
      - Currently, there are no protective measures from heavy power draw.

    - CPU: 2397 MHz @ 1220 mV (overvolting from 1120 mV)
      - NVIDIA Official Maximum: 1963.5 MHz
      - Maximum performance depends on CPU speedo.
        - You'd get somewhere between 2360 to 2390 MHz performance for real.
      - This is where floating point performance maxed out.
      - ≥ 2193 MHz will _ENABLE OVERVOLTING_.

    - GPU: 1305 MHz (no overvolting, less than official threshold 1050 mV)
      - NVIDIA Official Maximum: 1267.2 MHz
      - ~~Tested with deko3d compute shaders converted from Maxwell SASS assembly. Single-precision floating point (FP32 FFMA) performance maxes out at 1305 MHz.~~

    </details>

- **Modded sys-clk and ReverseNX**(-RT)

  - Permanent global clock override
    - Expected usage: set maximum DRAM clocks for all games and profiles.

  - Miscellaneous
    - Auto CPU Boost: For faster game loading
      - Enable CPU Boost (1785 MHz, could be configured higher) when CPU Core#3 (System Core) is stressed, especially when the game is loading assets from eMMC/SD card (I/O ops).
      - Auto-Boost will be enabled only when charger is connected.

    - Sync ReverseNX Mode: No need to change clocks manually after toggling modes in ReverseNX

    - View charger & battery info, toggle charging/fast-charging(2A) in overlay
      - Extend battery life expectancy by maintaining battery charge at 40% - 60% and disabling fast charging if possible.
      - Known issue: Fast charging toggle will be reset in-game.



## Installation

1. Download latest [release](https://github.com/KazushiMe/Switch-OC-Suite/releases/latest).

2. Mariko Only: Copy all files in `SdOut` to the root of SD card.
  - Erista: Use official sys-clk instead. Only `loader.kip` and some benchmark homebrew are available.

3. Grab `x.x.x_loader.kip` for your Atmosphere version, rename it to `loader.kip` and place it in `/atmosphere/kips/`.

4. Customization
    <details>

    | Defaults   | Mariko        | Erista       |
    | ---------- | ------------- | ------------ |
    | CPU OC     | 2397 MHz Max  | Disabled     |
    | CPU Boost  | 1785 MHz      | N/A          |
    | CPU Volt   | 1220 mV Max   | Disabled     |
    | GPU OC     | 1305 MHz Max  | N/A          |
    | RAM OC     | 1996 MHz Max  | 1862 MHz Max |
    | RAM Volt   | N/A           | Disabled     |
    | RAM Timing | Auto-Adjusted | Disabled     |

    </details>

  - Loader configurator
    - Grab [ldr_config.py](https://github.com/KazushiMe/Switch-OC-Suite/raw/master/ldr_config.py) and modify values in `cust_conf` dict.
    - `python ldr_config.py loader.kip -s` will save your configuration in-place.

5. **Hekate-ipl bootloader**
   - Rename the kip to `loader.kip` and add `kip1=atmosphere/kips/loader.kip` in `bootloader/hekate_ipl.ini`
   - Erista: Minerva module conflicts with HOS DRAM training. Recompile with frequency changed is recommeded, although you could simply remove `bootloader/sys/libsys_minerva.bso`.

   **Atmosphere Fusee bootloader:**
   - Fusee will load any kips in `/atmosphere/kips/` automatically.


### Patching sysmodules manually

This method is only served as reference as it could damage your MMC file system if not handled properly.

Patched sysmodules would be persistent until pcv or ptm was updated in new HOS (normally in `x.0.0`).

<details>

  Tools:
  - Lockpick_RCM
  - TegraExplorer
  - [hactool](https://github.com/SciresM/hactool)
  - [nx2elf](https://github.com/shuffle2/nx2elf)
  - elf2nso from [switch-tools](https://github.com/switchbrew/switch-tools/)
  - [hacpack](https://github.com/The-4n/hacPack)

  1. Dump `prod.keys` with Lockpick_RCM
  2. Dump HOS firmware with TegraExplorer
  3. Configure and run `test_patch.sh` to generate patched pcv & ptm sysmodules in nca
  4. Replace nca in `SYSTEM:/Contents/registered/` with TegraExplorer
  5. `ValidateAcidSignature()` should be stubbed to allow unsigned sysmodules to load (a.k.a. `loader_patch`)

</details>



## Build

### Loader KIP

Grab necessary patches from the repo, then compile sys-clk, ReverseNX-RT and Atmosphere loader with devkitpro.

If you are to install nro forwarders, stub `ValidateAcidSignature()` with `R_SUCCEED();` in `Atmosphere/stratosphere/loader/source/ldr_meta.cpp` to make them work again.

Uncompress the kip to make it work with config editor: `hactool -t kip1 Atmosphere/stratosphere/loader/out/nintendo_nx_arm64_armv8a/release/loader.kip --uncompress=./loader.kip`


- GPU OC (Higher than 921) Does not result in preformance improvements on Ersita, instead just producing more heat and drawing more power without any benefits. 

## Acknowledgement

- CTCaer for [Hekate-ipl](https://github.com/CTCaer/hekate) bootloader, RE and hardware research
- [devkitPro](https://devkitpro.org/) for All-In-One homebrew toolchains
- masagrator for [ReverseNX-RT](https://github.com/masagrator/ReverseNX-RT) and info on BatteryChargeInfoFields in psm module
- Nvidia for [Tegra X1 Technical Reference Manual](https://developer.nvidia.com/embedded/dlc/tegra-x1-technical-reference-manual)
- RetroNX team for [sys-clk](https://github.com/retronx-team/sys-clk)
- SciresM and Reswitched Team for the state-of-the-art [Atmosphere](https://github.com/Atmosphere-NX/Atmosphere) CFW of Switch
- Switchbrew [wiki](http://switchbrew.org/wiki/) for Switch in-depth info
- ZatchyCatGames for RE and original OC loader patches for Atmosphere
