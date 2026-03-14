# ZMK Zephyr Environment

A Nix environment for building [ZMK](https://zmk.dev/) firmware, configured specifically for the `nice_nano` (using the `nrf52840` MCU) and the Sofle keyboard.

## Getting Started

Because this environment is managed by Nix using `flakes`, you have everything you need out of the box.

### Prerequisites

* [Nix](https://nixos.org/download.html) with Flakes enabled
* Optional: `direnv` and `nix-direnv` for automatic activation

### Usage

Enter the development environment:

```shell
nix develop
```

Or, if you use `direnv`, simply run:

```shell
direnv allow
```

This will automatically drop you into a shell with:
* The Zephyr SDK (version 0.16.x) configured for `arm-zephyr-eabi`
* The python environment with `west` and all dependencies
* Standard tools: `cmake`, `ninja`, `dfu-util`, `ccache`

### Building ZMK Firmware

Once you are in the environment, you can use standard `west` commands. If you are starting fresh:

1. **Initialize the workspace**
   ```shell
   # Only required once:
   nix develop --command west init -m https://github.com/zmkfirmware/zmk.git --mf app/west.yml zmk-workspace
   cd zmk-workspace && nix develop --command west update
   ```

2. **Build the firmware**
   From the `zmk-workspace` directory:
   ```shell
   # Build for Sofle left half on nice_nano
   nix develop --command west build -s zmk.git/app -b nice_nano -- -DSHIELD=sofle_left

   # Build for Sofle right half on nice_nano (into separate build dir)
   nix develop --command west build -s zmk.git/app -b nice_nano -d build/right -- -DSHIELD=sofle_right
   ```
   The `.uf2` files will be in `build/zephyr/zmk.uf2` and `build/right/zephyr/zmk.uf2` respectively.
