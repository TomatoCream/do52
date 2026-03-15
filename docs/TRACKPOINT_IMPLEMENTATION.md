# Trackpoint Implementation Reference Summary

This document summarizes the technical differences and advanced configurations found in the `references/zmk-config-do52pro` project for implementing a PS/2 Trackpoint on the `do52pro` keyboard.

## 1. Core Dependencies
- **ZMK Fork:** `infused-kim/zmk`, revision `pr-testing/mouse_ps2_module_base`.
- **Driver Module:** `infused-kim/kb_zmk_ps2_mouse_trackpoint_driver`.
- **Helper Modules:** `urob/zmk-helpers` and `dhruvinsh/zmk-tri-state`.

## 2. Advanced Hardware Configuration (Interrupt Tuning)
To prevent cursor jitter/stuttering caused by Bluetooth or system tasks, the reference project demotes all non-essential system interrupts to a lower priority (`3`), allowing the Trackpoint (`0`) to always take precedence.

```dts
&gpiote { interrupts = < 6 0 >; }; // Highest priority
&clock { interrupts = < 0 3 >; };
&power { interrupts = < 0 3 >; };
&radio { interrupts = < 1 3 >; };
// ... (Apply to all peripherals: uart, i2c, spi, timer, etc.)
```

## 3. Driver Strategy: UART vs. GPIO
- **Preferred:** PS/2 UART driver (bit-banged). It is more efficient and stable for NRF52 chips (nice!nano).
- **Pin Muxing:** Requires complex `pinctrl` definitions to "borrow" UART pins for PS/2 data while using a standard GPIO for the clock.

### Reference Pin Assignments (do52pro_tp)
- **SCL (Clock):** `<&pro_micro 15>` (D15)
- **SDA (Data):** `<&pro_micro 16>` (D16)
- **Reset (Optional):** `<&pro_micro 9>` (D9)

## 4. Software Features & Layers
- **Modular Includes:** Behaviors and settings are split into `includes/mouse_tp.dtsi` and `includes/mouse_keys.dtsi`.
- **Runtime Tuning:** A dedicated `MouseTP_Set` layer allows on-the-fly adjustment of:
    - Sensitivity (Incr/Decr)
    - Drift/Thresholds
    - Resetting the trackpoint
- **Input Listeners:** Uses `zmk,input-listener-ps2` for advanced data handling and potential auto-layer features.

## 5. Build Configuration
- **Central Side:** Right side is set as the Bluetooth Central (`CONFIG_ZMK_SPLIT_ROLE_CENTRAL=y`) since the trackpoint is typically connected to the right half.
- **Target:** Reference uses `nice_nano_v2`.
