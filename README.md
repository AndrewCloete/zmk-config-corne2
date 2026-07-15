# ZMK Config — Corne

ZMK firmware config for Corne (crkbd) keyboards.

## Builds

| Board | Shield | Notes |
|---|---|---|
| `nice_nano//zmk` | `corne_left` / `corne_right` | BLE split via nRF52840 |

> **Note:** All board identifiers require the `//zmk` variant suffix as of Zephyr 4.1.
> `nice_nano` → `nice_nano//zmk`, `sparkfun_pro_micro_rp2040` → `sparkfun_pro_micro_rp2040//zmk`, etc.

---

## crkbd v4.1.1 (foostan, RP2040 integrated on PCB)

### Use QMK, not ZMK

The crkbd v4.1.1 has an RP2040 soldered directly onto the PCB with a direct GPIO matrix (no row/column scanning — each of the 42 keys has its own dedicated GPIO pin). ZMK can handle the key matrix via `zmk,kscan-gpio-direct`, but **wired split between halves does not work**.

**Why split fails in ZMK:**  
The TRRS cable carries a single data wire. QMK uses the RP2040's PIO (Programmable I/O) to implement half-duplex single-wire serial on GP12. ZMK's wired split transport requires full-duplex UART (separate TX and RX lines), and Zephyr has no RP2040 PIO UART driver. ZMK's experimental `half-duplex` mode is for RS-485 transceivers with a direction GPIO — not single-wire PIO.

**Recommendation:** Use [QMK](https://docs.qmk.fm/) — officially supported, split works out of the box.

→ QMK keyboard path: `keyboards/crkbd/rev4_1`  
→ Vial fork: https://get.vial.today (drag-and-drop keymap editing, no compile needed)  
→ crkbd v4 QMK source: https://github.com/qmk/qmk_firmware/tree/master/keyboards/crkbd/rev4_1

### RP2040 GPIO pin mapping (for reference)

Source: QMK `keyboards/crkbd/rev4_1/info.json`

**Left half** (outer→inner column order per row):

| Position | Key | GP pin |
|---|---|---|
| Top row | col0→col5 | GP22, GP20, GP23, GP26, GP29, GP0 |
| Middle row | col0→col5 | GP19, GP18, GP24, GP27, GP1, GP2 |
| Bottom row | col0→col5 | GP17, GP16, GP25, GP28, GP3, GP9 |
| Thumb row | col3→col5 | GP14, GP15, GP11 |

**Right half** (inner→outer column order per row):

| Position | Key | GP pin |
|---|---|---|
| Top row | col6→col11 | GP27, GP1, GP2, GP3, GP9, GP8 |
| Middle row | col6→col11 | GP26, GP28, GP0, GP4, GP14, GP11 |
| Bottom row | col6→col11 | GP22, GP20, GP29, GP5, GP18, GP15 |
| Thumb row | col6→col8 | GP19, GP17, GP16 |

All pins are `GPIO_ACTIVE_LOW | GPIO_PULL_UP` (key press pulls to GND).

Serial split pin: **GP12** (single-wire half-duplex, PIO-based in QMK).
