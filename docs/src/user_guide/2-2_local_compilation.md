# Local compilation

This sections describles all you need to compile RMK firmware on your local machine.

## Setup RMK environment

First, you'll setup the Rust development environment, install all needed components for compiling and flashing RMK.

### 1. Install Rust

RMK is written in Rust, so first you have to install Rust to your host. Installing Rust is easy, checkout [https://rustup.rs](https://rustup.rs) and follow the instructions.

[Here](https://doc.rust-lang.org/book/ch01-01-installation.html) is a more detailed guide for installing Rust.

### 2. Choose your hardware and install the target

RMK firmware runs on microcontrollers. By using [Embassy](https://github.com/embassy-rs/embassy) as the runtime, RMK supports many series of microcontrollers, such as stm32, nrf52 and rp2040. Choose one of the supported microcontroller makes your journey of RMK much easier. In RMK repo, there are many examples, microcontrollers in examples are safe options. If you're using other microcontrollers, make sure your microcontroller supports [Embassy](https://github.com/embassy-rs/embassy).

The next step is to add Rust's compilation target of your chosen microcontroller. Rust's default installation include only your host's compilation target, so you have to install the compilation target of your microcontroller manually.

Different microcontrollers with different architectures may have different compilation targets, if you're using ARM Cortex-M microcontrollers, [here](https://docs.rust-embedded.org/book/intro/install.html#rust-toolchain) is a simple target list.

For example, rp2040 is a Cortex-M0+ microcontroller, it's compilation target is `thumbv6m-none-eabi`. Use `rustup target add` command to install it:

```bash
rustup target add thumbv6m-none-eabi
```

nRF52840 is also commonly used in wireless keyboards, it's compilation target is `thumbv7em-none-eabihf`. To add the target, run:

```bash
rustup target add thumbv7em-none-eabihf
```

### 3. Install `rmkit` and other tools

`rmkit` is a tool that helps you create your RMK project easily. You can use the following command to install `rmkit`:

```shell
cargo install rmkit
```

There are several other tools should be installed:

- `flip-link`: zero-cost stack overflow protection.

- `cargo-make`: used to automate uf2 generation.

- (optional) `probe-rs`: used to flash and debug your firmware via debug proble. [Here](https://probe.rs/docs/getting-started/installation/) is the installation instruction.


You can use the following commands to install them:

```bash
  # Install flip-link
  cargo install flip-link cargo-make

  # Install probe-rs using scripts
  # Linux, macOS
  curl --proto '=https' --tlsv1.2 -LsSf https://github.com/probe-rs/probe-rs/releases/latest/download/probe-rs-tools-installer.sh | sh
  # Windows
  irm https://github.com/probe-rs/probe-rs/releases/latest/download/probe-rs-tools-installer.ps1 | iex
  ```

Generating uf2 firmware also requires you have `python` command available. [Here](https://wiki.python.org/moin/BeginnersGuide/Download) is a guide for installing python.

## Create firmware project

You can use `rmkit` to create the firmware project:

```shell
rmkit init
```

This command would ask you to fill some basic info of your project, then a project will be created from RMK's templates: 

```shell
$ rmkit init                                                                
> Project Name: rmk-keyboard
> Choose your keyboard type? split
> Choose your microcontroller nrf52840
⇣ Download project template for nrf52840_split...
✅ Project created, path: rmk-keyboard
```

Now RMK has project templates for many microcontrollers, such as nRF52840, rp2040, stm32, esp32, etc. If you find that there's no template for your microcontroller, please feel free to add one.

## Update firmware project

The generated project uses `keyboard.toml` to config the keyboard. There're steps you have to do to customize your own firmware:

### Edit `keyboard.toml`

The generated `keyboard.toml` should have some fields configured from `rmkit init`. But there are still some fields that you want to fill, such as the pin matrix, default keymap, led config, etc.

The [Keyboard Configuration](../keyboard_configuration.md) section has full instructions of how to write your own `keyboard.toml`. Follow the doc and report any issues/questions at <https://github.com/HaoboGu/rmk/issues>. We appreciate your feedback!

### Update `memory.x`

`memory.x` is the linker script of Rust embedded project, it's used to define the memory layout of the microcontroller. RMK enables `memory-x` feature for `embassy-stm32`, so if you're using stm32, you can just ignore this step.

For other ARM Cortex-M microcontrollers, you only need to update the `LENGTH` of FLASH and RAM to your microcontroller.

If you're using **nRF52840**, generally you have to change start address in `memory.x` to 0x27000 or 0x26000, according to your softdevice version. For example, softdevice v6.1.x should use 0x00026000 and v7.3.0 should be 0x00027000

You can either checkout your microcontroller's datasheet or existing Rust project of your microcontroller for it.

### Add your own layout(vial.json)

> The layout should be consistent with the default keymap set in `keyboard.toml`

The next step is to add your own keymap layout for your firmware. RMK supports [vial app](https://get.vial.today/), an
open-source cross-platform(windows/macos/linux/web) keyboard configurator. So the vial like keymap definition has to be
imported to the firmware project.

Fortunately, RMK does most of the heavy things for you, all you need to do is to create your own keymap definition and
convert it to `vial.json` following **[vial's doc here](https://get.vial.today/docs/porting-to-via.html)**, and place it
at the root of the firmware project, replacing the default one. RMK would do all the rest things for you.

### (Optional) Update compilation target

For stm32 microcontrollers, the compilation target varies according to the series. If there's no project template for your specific stm32 model, a common template will be used. An extra step for the common template is to update `.cargo/config.toml`, change the project's default target:

```toml
[build]
# Pick ONE of these default compilation targets
# target = "thumbv6m-none-eabi"        # Cortex-M0 and Cortex-M0+
# target = "thumbv7m-none-eabi"        # Cortex-M3
# target = "thumbv7em-none-eabi"       # Cortex-M4 and Cortex-M7 (no FPU)
target = "thumbv7em-none-eabihf"     # Cortex-M4F and Cortex-M7F (with FPU)
# target = "thumbv8m.base-none-eabi"   # Cortex-M23
# target = "thumbv8m.main-none-eabi"   # Cortex-M33 (no FPU)
# target = "thumbv8m.main-none-eabihf" # Cortex-M33 (with FPU)
```

It's also welcome to submit and share your project template, please open an [issue](https://github.com/HaoboGu/rmk-template/issues) with your project attached. 

## Compile the firmware

To compile the firmware is easy, just run

```shell
cargo build --release
```

If you've done all the previous steps correctly, you can find your compiled firmware at `target/<your_target>/release` folder, whose name is your project's name or the name set in `Cargo.toml`'s `[[bin]]` section.

The firmware generated by Rust has no extension, which is actually an ELF file.

If you encountered any problems when compiling the firmware, please report it [here](https://github.com/HaoboGu/rmk/issues).

### Compile uf2 firmware

By default, Rust firmware is an ELF file, so we have to do some extra steps converting it to uf2 format.

RMK uses [cargo-make](https://github.com/sagiegurari/cargo-make) to automate the uf2 firmware generation. Generating uf2 firmware also requires you have `python` command available.

Then you should make sure the chip family argument(aka argument after -f) in `Makefile.toml` is correct. You can get your chip's family ID in `scripts/uf2conv.py`.

That's all you need to set. The final step is to run

```shell
cargo make uf2 --release
```

to generate your uf2 firmware.