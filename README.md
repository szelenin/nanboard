# nanboard

Working through the [nandland Go Board tutorials](https://nandland.com/go-board-tutorials/) on an Apple Silicon Mac.

The tutorials use Lattice iCEcube2 and Programmer Standalone, which only run on x86 Windows/Linux.
This repo uses the open-source toolchain via [Apio](https://github.com/FPGAwars/apio) instead
(yosys, nextpnr-ice40, icestorm), as the tutorial author recommends for Mac. No Lattice license is needed.

Verified on: macOS 15.7 (arm64), Apio 1.6.0.

## Setup (macOS)

```bash
brew install pipx
pipx ensurepath          # then open a new terminal so ~/.local/bin is on PATH
pipx install apio
apio packages install    # downloads yosys, nextpnr, icestorm, iverilog, etc. into ~/.apio
```

Check that Apio knows the board:

```bash
apio boards | grep go-board   # go-board ... ICE40HX1K-VQ100 ... iceprog
```

No USB driver is needed on macOS (`apio drivers install ftdi` reports
"No driver installation is required on this platform").

## Repo layout

- `Go_Board_Constraints.pcf` — pin map for the whole board using the tutorial's signal names
  (`i_Clk`, `i_Switch_1`, `o_LED_1`, `o_Segment1_A`, `i_UART_RX`, `o_VGA_HSync`, `io_PMOD_1`, ...).
  Each project symlinks to it.
- `project1-switches-leds/` — Project 1: each switch drives an LED.

Use the **Verilog** version of each tutorial; the open-source flow does not support VHDL out of the box.

## Building a project

```bash
cd project1-switches-leds
apio build
```

This replaces iCEcube2's "Run All" (synthesis, place & route, bitstream) and writes `_build/default/hardware.bin`.

## Raspberry Pi (64-bit) differences

Verified on: Raspberry Pi running Debian 13 (trixie), aarch64.

**Apio does not work here.** Apio 1.6.0 only supports `darwin-arm64`, `linux-x86-64` and
`windows-amd64`. `pipx install apio` succeeds, but `apio packages install` fails with
`Error: Unknown platform id: [linux-aarch64]`.

Instead, install the same tools Apio uses (YosysHQ OSS CAD Suite) directly. No sudo is needed:

```bash
mkdir -p ~/projects && cd ~/projects && git clone https://github.com/szelenin/nanboard.git
# Pick the latest linux-arm64 asset from https://github.com/YosysHQ/oss-cad-suite-build/releases
curl -fL -o /tmp/oss-cad-suite.tgz \
  https://github.com/YosysHQ/oss-cad-suite-build/releases/download/2026-11-01/oss-cad-suite-linux-arm64-20261101.tgz
tar -xzf /tmp/oss-cad-suite.tgz -C ~ && rm /tmp/oss-cad-suite.tgz
```

Build by running the same commands `apio build` runs on the Mac, after loading the suite into the shell:

```bash
source ~/oss-cad-suite/environment     # once per shell
cd ~/projects/nanboard/project1-switches-leds
mkdir -p _build
yosys -q -p "synth_ice40 -top Switches_To_LEDs -json _build/hardware.json" Switches_To_LEDs.v
nextpnr-ice40 -q --hx1k --package vq100 --pcf Go_Board_Constraints.pcf \
  --json _build/hardware.json --asc _build/hardware.asc
icepack _build/hardware.asc _build/hardware.bin
```

For another project, change `-top` to that project's top module and the `.v` file name.

## Starting a new project

```bash
mkdir projectN-name && cd projectN-name
ln -s ../Go_Board_Constraints.pcf Go_Board_Constraints.pcf
cat > apio.ini <<'EOF'
[env:default]
board = go-board
top-module = <Your_Top_Module>
EOF
```

Then add your `.v` file with port names that match the `.pcf`, and run `apio build`.
