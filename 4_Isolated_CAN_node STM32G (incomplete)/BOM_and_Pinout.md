# Isolated CAN FD Node — Parts List (Phase 01) & Pinout Reference (Phase 02)

## Phase 01 — Locked BOM & LCSC Reference

> **Important Corrections Applied**
> - **U2 (`ISO1042DW` vs `ISO1042DWV`):** Your 16-pin wide-body SOIC footprint (`SOIC-16W_7.5x10.3mm_P1.27mm`) corresponds to TI's **16-pin `ISO1042DW` / `ISO1042DWR`**, not `ISO1042DWV` (which is an 8-pin wide-body SOIC). The schematic symbol and BOM use `ISO1042DW`.
> - **U2 Logic-Side Supply (`VCC1`):** `ISO1042` `VCC1` is wired to **`+3.3V`** (not `+5V`) because `VIH(min) = 0.7 × VCC1` (`2.31 V` at `3.3 V` vs `3.5 V` at `5 V`), matching the `3.3 V` STM32G431 I/O levels.
> - **Crystal Load Caps (`Y1`):** `X322516MLB4SI` (`C13738`) specifies `CL = 9 pF`. Assuming ~3 pF stray PCB/pin capacitance, `C14 = C15 = 2 × (9 pF − 3 pF) = 12 pF` (C0G/NP0).

| Ref | Part / Value | Package / Footprint | Verified LCSC / MPN Note | Qty (+Spares) |
| :--- | :--- | :--- | :--- | :--- |
| `U1` | `STM32G431CBT6` | `LQFP-48_7x7mm_P0.5mm` | ST `STM32G431CBT6` (check LCSC stock at order time) | 1 (+2 spares) |
| `U2` | `ISO1042DW` | `SOIC-16W_7.5x10.3mm_P1.27mm` | TI `ISO1042DWR` (`C1848126`) | 1 (+2 spares) |
| `U3` | `B0505S-1WR3` | `Converter_DCDC_Murata_MEE1SxxxxSC_THT` (SIP-4) | Mornsun `B0505S-1WR3` (`C84643` / alt `C2992394`) | 1 (+2 spares) |
| `U4` | `AP2112K-3.3TRG1` | `SOT-23-5` | Diodes Inc `AP2112K-3.3TRG1` (`C51118`) | 1 (+2 spares) |
| `Y1` | `16 MHz` crystal (`CL = 9 pF`) | `Crystal_SMD_3225-4Pin_3.2x2.5mm` | YXC `X322516MLB4SI` (`C13738`) | 1 (+2 spares) |
| `D1` | `SM712` (+12 V / −7 V TVS) | `SOT-23` | Bourns `CDSOT23-SM712` (`C404012`) | 1 (+2 spares) |
| `FL1` | `51 µH` CM choke (or `0 Ω` bypass) | `L_CommonMode_Wurth_WE-CNSW-1206` (1206-4) | Populate 1206-4 CAN CM choke later or bridge `1-2` & `3-4` with `0 Ω` | 1 |
| `J1` | USB-C 16-pin receptacle | `USB_C_Receptacle_HRO_TYPE-C-31-M-12` | HRO `TYPE-C-31-M-12` (`C165948`) | 1 |
| `J2` | DB9 male THT | `DSUB-9_Pins_Horizontal_P2.77x2.84mm_EdgePinOffset7.70mm_Housed_MountingHolesOffset9.12mm` | Standard right-angle PCB DE9 male | 1 |
| `J4` | 3-pin screw terminal (5.08 mm) | `TerminalBlock_Phoenix_MKDS-1,5-3-5.08_1x03_P5.08mm_Horizontal` | 3-pin 5.08 mm THT terminal block | 1 |
| `J3` | 4-pin SWD header | `PinHeader_1x04_P2.54mm_Vertical` | 1×4 2.54 mm header (`SWDIO`, `SWCLK`, `NRST`, `GND`) | 1 |
| `JP1` | 2-pin termination jumper | `PinHeader_1x02_P2.54mm_Vertical` | 1×2 2.54 mm header + shunt | 1 |
| `D2`, `D3` | Indicator LEDs (Power, Status) | `LED_0603_1608Metric` | Green (`D2`), Blue (`D3`) 0603 LEDs | 2 |
| `R1`, `R7` | `1 kΩ` (LED current limit) | `R_0603_1608Metric` | 0603 5% or 1% | 2 |
| `R2`, `R3` | `5.1 kΩ` 1% (USB-C CC1/CC2) | `R_0603_1608Metric` | 0603 1% | 2 |
| `R4` | `4.7 kΩ` (PB8/BOOT0 pulldown) | `R_0603_1608Metric` | 0603 1% or 5% | 1 |
| `R5`, `R6` | `60 Ω` 1% (split termination) | `R_0603_1608Metric` | 0603 1% (or `60.4 Ω` 1% E96 standard) | 2 |
| `C5` | `4.7 nF` (split termination) | `C_0603_1608Metric` | 0603 ≥50 V X7R/C0G | 1 |
| `C14`, `C15` | `12 pF` (HSE crystal load) | `C_0603_1608Metric` | 0603 C0G/NP0 50 V | 2 |
| `C6–C9, C11, C13, C18–C20` | `100 nF` (decoupling + NRST) | `C_0603_1608Metric` | 0603 X7R/X5R ≥16 V | 9 |
| `C2, C3, C12` | `1 µF` (LDO + VDDA) | `C_0603_1608Metric` | 0603 X5R/X7R ≥10 V | 3 |
| `C1, C4, C10, C16, C17` | `4.7 µF` (bulk decoupling) | `C_0603_1608Metric` | 0603 X5R/X7R ≥10 V | 5 |

---

## Phase 02 — STM32G431CBT6 Pinout & CubeMX Configuration

### CubeMX Setup Order
1. **SYS → Debug:** `Serial Wire` (`PA13` = `SWDIO`, `PA14` = `SWCLK`)
2. **RCC → HSE:** `Crystal/Ceramic Resonator` (`PF0` = `RCC_OSC_IN`, `PF1` = `RCC_OSC_OUT`, 16 MHz)
3. **USB → Device (FS):** Enabled (`PA11` = `USB_DM`, `PA12` = `USB_DP`)
4. **FDCAN1 → Mode:** `Classic + FD`, remapped off `PA11/PA12` onto **`PB8` (`FDCAN1_RX`)** and **`PB9` (`FDCAN1_TX`)**
5. **GPIO Output:** `PC13` = `LED_STATUS`
6. **Clock Tree:** 16 MHz HSE → PLL → `SYSCLK` (up to 170 MHz), **`48 MHz` USB clock** (via PLLQ or HSI48+CRS; use PLL from 16 MHz HSE for deterministic timing)
7. **Option Bytes (Critical for `PB8/BOOT0` trap):** Clear `nBOOT0` / `nSWBOOT0` (set `nSWBOOT0 = 0`, `nBOOT0 = 1` in STM32CubeProgrammer so MCU boots from main Flash regardless of `PB8` state; `R4` `4.7 kΩ` pulldown protects the pre-option-byte reset window).

### Pin Assignment Table (`U1: STM32G431CBT6`)

| Pin # | Pin Name | Schematic Net | Function |
| :--- | :--- | :--- | :--- |
| `1` | `VBAT` | `+3.3V` | Tied to 3.3 V + `100 nF` (`C6`) |
| `2` | `PC13` | `LED_STATUS` (via `R7`/`D3`) | Firmware status LED |
| `5` | `PF0` | `/Logic/OSC_IN` | 16 MHz HSE crystal (`Y1.1`, `C14` `12 pF`) |
| `6` | `PF1` | `/Logic/OSC_OUT` | 16 MHz HSE crystal (`Y1.3`, `C15` `12 pF`) |
| `7` | `PG10` | `/Logic/NRST` | Reset (`C13` `100 nF` to `GND`, `J3.3`) |
| `19` | `VSSA` | `GND` | Analog ground |
| `20` | `VREF+` | `+3.3V` | Analog reference (`C11`, `C12`) |
| `21` | `VDDA` | `+3.3V` | Analog power |
| `23, 35, 47` | `VSS` | `GND` | Logic ground |
| `24, 36, 48` | `VDD` | `+3.3V` | Logic power (`C7–C10`) |
| `33` | `PA11` | `/Logic/USB_DM` | USB FS D− (`J1.A7/B7`) |
| `34` | `PA12` | `/Logic/USB_DP` | USB FS D+ (`J1.A6/B6`) |
| `37` | `PA13` | `/Logic/SWDIO` | SWD data (`J3.1`) |
| `38` | `PA14` | `/Logic/SWCLK` | SWD clock (`J3.2`) |
| `45` | `PB8` | `/Bus/CAN_RXD` | `FDCAN1_RX` + `4.7 kΩ` (`R4`) to `GND` (`U2.RXD`) |
| `46` | `PB9` | `/Bus/CAN_TXD` | `FDCAN1_TX` (`U2.TXD`) |
