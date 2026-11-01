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
