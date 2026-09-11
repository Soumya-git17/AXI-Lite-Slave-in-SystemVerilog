# AXI4-Lite Slave in SystemVerilog

A modular **AXI4-Lite slave IP** implemented in SystemVerilog, featuring independent read and write channels, address decoding, a configurable register bank, byte-write strobes, and error handling.

## Features

* AXI4-Lite slave interface
* 32-bit address and 32-bit data width
* 16 programmable 32-bit registers
* Independent AXI4-Lite read and write channels
* Supports both:
  * AW-first transactions
  * W-first transactions
  * AW/W arriving in the same cycle
* Byte-wise write support using `WSTRB`
* Address decoding for register accesses
* Invalid-address detection
* AXI `DECERR` response for invalid accesses
* Separate read and write control logic
* SystemVerilog interface and package
* Self-checking SystemVerilog testbench
* Simulation using Verilator
* Waveform generation for transaction-level debugging
* RTL linting using Verilator

## AXI4-Lite Channels

The implementation handles the five independent AXI4-Lite channels:

### Write Address Channel

```text
AWVALID / AWREADY
AWADDR
```

The slave captures a write address when the address handshake occurs.

### Write Data Channel

```text
WVALID / WREADY
WDATA
WSTRB
```

The implementation allows the write address and write data to arrive independently.

### Write Response Channel

```text
BVALID / BREADY
BRESP
```

The slave generates an `OKAY` response for valid accesses and `DECERR` for invalid addresses.

### Read Address Channel

```text
ARVALID / ARREADY
ARADDR
```

The read address is accepted and decoded before generating the corresponding read response.

### Read Data Channel

```text
RVALID / RREADY
RDATA
RRESP
```

The requested register value is returned together with the AXI response.

## Write Transaction Handling

AXI4-Lite does not require `AWVALID` and `WVALID` to arrive in the same cycle.

The write-channel controller therefore handles:

```text
Case 1: AW arrives first

AWVALID ────────┐
                └── Address captured
                       ↓
                 Wait for WDATA


Case 2: W arrives first

WVALID ─────────┐
                └── Data captured
                       ↓
                 Wait for AWADDR


Case 3: AW and W arrive together

AWVALID ────────┐
WVALID ─────────┼── Both captured
                │
                ▼
             Write register
```

This was an important design consideration because treating the address and data channels as if they were a single handshake would not correctly model AXI4-Lite.

## Register Map

The design uses 16 registers with 32-bit data width.

| Address | Register |
| ------: | -------- |
|  `0x00` | REG0     |
|  `0x04` | REG1     |
|  `0x08` | REG2     |
|  `0x0C` | REG3     |
|   `...` | `...`    |
|  `0x3C` | REG15    |

Only word-aligned register addresses are considered valid.

An invalid address generates an AXI `DECERR` response.

## WSTRB Support

The write logic supports byte-level updates using `WSTRB`.

For a 32-bit data bus:

```text
WSTRB[3] → DATA[31:24]
WSTRB[2] → DATA[23:16]
WSTRB[1] → DATA[15:8]
WSTRB[0] → DATA[7:0]
```

For example:

```text
WSTRB = 4'b0011
```

updates the lower 16 bits while preserving the upper 16 bits.

Similarly:

```text
WSTRB = 4'b1100
```

updates the upper 16 bits.

## Project Structure

```text
AXI-Lite-Slave-in-SystemVerilog/
│
├── rtl/
│   ├── axi_lite_pkg.sv
│   ├── axi_lite_if.sv
│   ├── axi_lite_addr_decoder.sv
│   ├── axi_lite_write_channel.sv
│   ├── axi_lite_read_channel.sv
│   ├── axi_lite_reg_bank.sv
│   └── axi_lite_slave_top.sv
│
├── tb/
│   └── tb_axi_lite_slave.sv
│
├── waves/
│   └── axi_lite.vcd
│
├── Makefile
│
└── README.md
```

## Verification

The design was verified using a self-checking SystemVerilog testbench.

The testbench exercises the major AXI4-Lite behaviors, including:

* Write transactions
* Read transactions
* AW-first operation
* W-first operation
* Simultaneous AW/W handshakes
* Valid register accesses
* Invalid-address accesses
* `WSTRB` byte enables
* AXI response generation
* Reset behavior

Simulation was performed using **Verilator**, with VCD waveform generation for signal-level debugging.

### Example Simulation Flow

```bash
make
```

The generated waveform can be inspected using GTKWave.

```bash
gtkwave waves/axi_lite.vcd
```

## Linting

The RTL was also checked using Verilator linting.

The design was iteratively corrected for issues such as SystemVerilog elaboration, signal connectivity, and RTL coding problems.

## Planned Verification Extensions

The next stage of the project is to strengthen the verification environment with **SystemVerilog Assertions (SVA)** and **functional coverage**.

Planned SVA checks include:

* VALID stability until handshake
* Address/data stability while waiting for READY
* Correct response-channel behavior
* Read/write handshake properties
* Reset-related protocol properties
* Invalid-address response checking
* AXI4-Lite protocol sequencing

Functional coverage will subsequently be used to measure whether important transaction scenarios have been exercised.

## Tools

* SystemVerilog
* Verilator
* GTKWave
* Make
* Git / GitHub

## Learning Objectives

This project was developed to gain practical experience with:

* RTL design
* SystemVerilog
* AMBA AXI4-Lite protocol
* Finite-state machines
* Memory-mapped register interfaces
* Byte-enable logic
* Modular RTL architecture
* Self-checking verification
* RTL linting
* Waveform-based debugging
* Assertion-based verification
