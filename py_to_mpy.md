# Simple Guide: Convert MicroPython `.py` to `.mpy`

Use this guide when you want to compile MicroPython `.py` files into `.mpy`
files for any MicroPython board.

Important rule:

```text
Board MicroPython firmware version must match mpy-cross version.
```

Example:

```text
Board firmware: MicroPython v1.27.0
mpy-cross:      MicroPython v1.27.0
```

## Step 1: Check or Download the MicroPython Repo

First check if the MicroPython repo already exists:

```bash
ls /home/ubuntu/micropython
```

If it exists, check the current version:

```bash
git -C /home/ubuntu/micropython describe --tags --dirty --always
```

If it does not exist, download it:

```bash
git clone https://github.com/micropython/micropython.git /home/ubuntu/micropython
```

Install build tools if needed:

```bash
sudo apt update
sudo apt install -y git build-essential python3
```

## Step 2: Check the MicroPython Firmware on the Board

Open the board REPL with Thonny, `mpremote`, PuTTY, or another serial terminal.

Run this on the board:

```python
import sys
print(sys.version)
print(sys.implementation)
print(sys.implementation._mpy)
```

Example from Olimex ESP32 ETH:

```text
3.4.0; MicroPython v1.27.0 on 2025-12-09
(name='micropython', version=(1, 27, 0, ''), _machine='Olimex ESP32 ETH with ESP32', _mpy=11014, _build='OLIMEX_ESP32_POE', _thread='GIL')
11014
```

Write down the firmware version:

```text
v1.27.0
```

Also find the board architecture flag. Run this on the board:

```python
import sys

sys_mpy = sys.implementation._mpy
arch = [
    None,
    "x86",
    "x64",
    "armv6",
    "armv6m",
    "armv7m",
    "armv7em",
    "armv7emsp",
    "armv7emdp",
    "xtensa",
    "xtensawin",
    "rv32imc",
    "rv64imc",
][(sys_mpy >> 10) & 0x0F]

print("mpy version:", sys_mpy & 0xFF)
print("mpy sub-version:", (sys_mpy >> 8) & 3)
print("mpy flags:", end="")
if arch:
    print(" -march=" + arch, end="")
print()
```

For this Olimex ESP32 ETH board, `_mpy=11014` gives:

```text
mpy flags: -march=xtensawin
```

## Step 3: Match Board Firmware to `mpy-cross`

Checkout the same MicroPython version as the board.

For board firmware `v1.27.0`:

```bash
git -C /home/ubuntu/micropython checkout v1.27.0
```

If Git says local changes would be overwritten, save them first:

```bash
git -C /home/ubuntu/micropython status --short
git -C /home/ubuntu/micropython stash push -u -m "save local changes before mpy-cross build"
git -C /home/ubuntu/micropython checkout v1.27.0
```

Build `mpy-cross`:

```bash
make -C /home/ubuntu/micropython/mpy-cross clean
make -C /home/ubuntu/micropython/mpy-cross
```

Check the compiler version:

```bash
/home/ubuntu/micropython/mpy-cross/build/mpy-cross --version
```

Expected result for this board:

```text
MicroPython v1.27.0 ...
mpy-cross emitting mpy v6.3
```

If it still shows another version, for example `v1.24.0`, rebuild again after
checking out the correct tag.

## Step 4: Make `.py` Files into `.mpy`

Basic command format:

```bash
/home/ubuntu/micropython/mpy-cross/build/mpy-cross -march=BOARD_ARCH -o OUTPUT_FILE.mpy INPUT_FILE.py
```

For this Olimex ESP32 ETH board, use:

```bash
-march=xtensawin
```

Create the output folder:

```bash
mkdir -p MPY_fridge/md
touch MPY_fridge/md/__init__.py
```

Compile files one by one:

```bash
/home/ubuntu/micropython/mpy-cross/build/mpy-cross -march=xtensawin  raw_micropython/md/var.py
```

```bash
/home/ubuntu/micropython/mpy-cross/build/mpy-cross -march=xtensawin  raw_micropython/md/psv.py
```

```bash
/home/ubuntu/micropython/mpy-cross/build/mpy-cross -march=xtensawin  raw_micropython/md/rs_h1.py
```

```bash
/home/ubuntu/micropython/mpy-cross/build/mpy-cross -march=xtensawin  raw_micropython/md/simple.py
```

```bash
/home/ubuntu/micropython/mpy-cross/build/mpy-cross -march=xtensawin  raw_micropython/main.py
```
Note:- rename main.mpy to app.mpy  and dirt main.py file with code`import app app.run()`

```bash
cp raw_micropython/main.py MPY_fridge/main.py
```

## Upload One `.mpy` File to the Board

