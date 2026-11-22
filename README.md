# 8-Bit Bidirectional Barrel Shifter with Circular Rotation

An 8-bit transistor-level barrel shifter designed in **Cadence Virtuoso** using **GPDK 90 nm CMOS**. The circuit uses a **single right-shift core** to perform logical shifts and circular rotations in both directions. It supports shift amounts from **0 to 7 bits** and uses **47 transmission-gate 2:1 multiplexers**.

## How one core handles both directions

For a right shift, the input passes directly through three stages that can shift by **1, 2, and 4 bits**. Each stage either shifts its input or passes it through unchanged. For example, a shift amount of `101` enables the 1-bit and 4-bit stages, producing a 5-bit shift.

For a left shift, the circuit **reverses the input bit order, performs a right shift, and reverses the output bit order**. This allows the same three stages to serve both directions.

| Step | Example: logical left shift of `10110001` by 2 |
|---|---|
| Reverse the input bits | `10001101` |
| Shift right by 2 using the shared core | `00100011` |
| Reverse the output bits | `11000100` |

The `ROTATE` control selects what enters the vacated bit positions. A logical shift fills them with zeros. A circular rotation routes the bits leaving one end of the word back into the other end. The same bit-reversal logic also turns a right rotation into a left rotation.

## Operating modes

`DIR` selects the direction, `ROTATE` selects shift or rotation, and `SHIFT[2:0]` sets the amount. The examples below use input `10110001` and `SHIFT = 001`.

| `DIR` | `ROTATE` | Operation | Example output |
|---|---|---|---|
| `0` | `0` | Logical left shift | `01100010` |
| `0` | `1` | Rotate left | `01100011` |
| `1` | `0` | Logical right shift | `01011000` |
| `1` | `1` | Rotate right | `11011000` |

## Transistor-level implementation

The basic building block is a **transmission-gate 2:1 multiplexer**. Each transmission gate uses an NMOS and a PMOS in parallel with complementary gate controls. The same mux cell is reused throughout the shifter.

| Circuit block | 2:1 muxes | Function |
|---|---:|---|
| Input bit reversal | 8 | Selects the original or reversed input word |
| Three shift stages | 24 | Eight muxes per stage implement shifts of 1, 2, and 4 bits |
| Rotation wraparound | 7 | Selects zero-fill or wraparound for 1, 2, and 4 boundary bits |
| Output bit reversal | 8 | Restores bit order for leftward operations |
| **Total** | **47** | **Complete 8-bit shifter** |

## Simulation and power results

Transient simulation in Cadence Virtuoso covered **32 control combinations**: four operating modes at eight shift amounts each. The output waveform shows how `OUT[7:0]` changes with the direction, rotation control, and shift amount.

| Parameter | Result |
|---|---|
| Process | GPDK 90 nm CMOS |
| Supply voltage | 1.0 V |
| Average supply power over the applied test sequence | **3.17 nW** |

Average supply power was calculated as `P_avg = VDD × I_supply,avg` over the simulated test sequence. The power-dissipation plot shows how power varies during that sequence, while the average-power result gives the value over the measurement interval.

## Schematics and simulation outputs

| Repository artifact | What it shows |
|---|---|
| **2:1 MUX schematic** | The transistor-level transmission gates and complementary select signals used in the basic cell |
| **Complete barrel-shifter schematic** | The shared right-shift core, input/output reversal banks, and wraparound connections |
| **Functional waveform** | The input word, control signals, and corresponding output for shift and rotation operations |
| **Power-dissipation plot** | Supply power as a function of time during transient simulation |
| **Average-power result** | The simulated average supply power of **3.17 nW** at **1.0 V** |
