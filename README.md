# 16-Bit Computer

### Description

A 16-bit computer designed and tested in Logisim *(a digitial logic simulator)* for my custom ISA. It implements an ALU, control unit, memory, and simple memory-mapped I/O in the form of a screen.
* Hardware abstractions are used to wrap the hardware-level logic gates for components like clocks, registers, multiplexers, and RAM into single, independent IC's that can be re-used or copy-pasted across the entire simulated machine as needed.

    Specific logic families are also abstracted away so that the focus is on the digital logic rather than the actual hardware, although the digital logic circuit could be used as a blueprint for actually construction a physical version of this computer.

See `LogisimProjects/16BitComputer.circ` for the Logisim file.

---

### Development Setup
Follow Logisim's installation guide at [github.com/logisim-evolution](https://github.com/logisim-evolution/logisim-evolution). It provides the full toolkit for making, simulating, and debugging the digital logic.
* Install [Fritzing](https://fritzing.org/download/) if working on the physical, hardware-level side-projects.

---

### Custom Instruction Set Architecture Overview

**Machine Specifications**

* 8 Registers
    * 6 General Purpose Registers labeled `R0` - `R6` in assembly
    * R7 *(`RSP`)* is the Stack Pointer, and is not directly accessible
    * R8 *(`RIP`)* is the Instruction Pointer Register
* 16-bit addresses (Big-Endian)

**Instruction Set**

* 16-bit values and memory addresses  
* OpCode byte is split into two nibbles (category \- instruction)
* Additional Notes:  
    * False: `0x0000` | True: `0xFFFF`
    * `JMPT` jumps as long as the value in `RCondition` is NOT `0`


| *OpCode* | *Name* | *Format* | *Description* |
| :---: | :---: | :---: | :---: |
| `0x00` | NOP | `NOP PAD PAD PAD` | No Operation |

| *OpCode* | *Name* | *Format* | *Description* |
| :---: | :---: | :---: | :---: |
| `0x10` | ADD | `ADD RDest RAddend RAddend` | Adds the values of two registers and stores the sum in a destination register |
| `0x11` | SUB | `SUB RDest RMinuend RSubtrahend` | Subtracts the values of two registers and stores the difference in a destination register |
| `0x12` | MUL | `MUL RDest RFactor RFactor`  | Multiplies the values of two registers and stores the product in a destination register |
| `0x13` | DIV | `DIV RDest RDividend RDivisor`  | Divides the values of two registers and stores the quotient in a destination register |
| `0x14` | MOD | `MOD RDest RDividend RDivisor`  | Divides the values of two registers and stores the remainder in a destination register |

| *OpCode* | *Name* | *Format* | *Description* |
| :---: | :---: | :---: | :---: |
| `0x20` | NOT | `NOT RDest RSource PAD`  | Performs a bitwise NOT operation on the contents of RSource and stores the result in `RDest` |
| `0x21` | AND | `AND RDest R_LHS R_RHS` | Performs a bitwise AND operation on the contents of `R_LHS` and `R_RHS` registers and stores the result in `RDest` |
| `0x22` | OR | `OR RDest R_LHS R_RHS` | Performs a bitwise OR operation on the contents of `R_LHS` and `R_RHS` registers and stores the result in `RDest` |
| `0x23` | XOR | `XOR RDest R_LHS R_RHS` | Performs a bitwise XOR operation on the contents of `R_LHS` and `R_RHS` registers and stores the result in `RDest` |
| `0x24` | EQ | `EQ RDest R_LHS R_RHS` | Performs a bitwise equality check on the contents of `R_LHS` and `R_RHS` registers, storing `0xFFFF` in `RDest` if equal and `0x0000` if not equal. |
| `0x25` | NEQ | `NEQ RDest R_LHS R_RHS` | Performs a bitwise inequality check on the contents of `R_LHS` and `R_RHS` registers, storing `0x0000` in `RDest` if equal and `0xFFFF` if not equal. |
| `0x26` | GTE | `GTE RDest R_LHS R_RHS` | Performs a bitwise comparison on the contents of `R_LHS` and `R_RHS` registers, storing `0xFFFF` in `RDest` if `R_LHS` ≥ `R_RHS` and `0x0000` otherwise. |
| `0x27` | LTE | `LTE RDest R_LHS R_RHS` | Performs a bitwise comparison on the contents of `R_LHS` and `R_RHS` registers, storing `0xFFFF` in `RDest` if `R_LHS` ≤ `R_RHS` and `0x0000` otherwise. |
| `0x28` | GT | `GT RDest R_LHS R_RHS` | Performs a bitwise comparison on the contents of `R_LHS` and `R_RHS` registers, storing `0xFFFF` in `RDest` if `R_LHS` \> `R_RHS` and `0x0000` otherwise. |
| `0x29` | LT | `LT RDest R_LHS R_RHS` | Performs a bitwise comparison on the contents of `R_LHS` and `R_RHS` registers, storing `0xFFFF` in `RDest` if `R_LHS` \< `R_RHS` and `0x0000` otherwise. |

| *OpCode* | *Name* | *Format* | *Description* |
| :---: | :---: | :---: | :---: |
| `0x30` | JMP | `JMP PAD AddrHigh AddrLow` OR `JMP #labelName` | Jumps to a memory location denoted by a 16-bit value (`AddrHigh AddrLow`).  Labels are supported in ASM, replacing the address |
| `0x31` | JMPi | `JMPi PAD RLocation PAD` | Indirectly jumps to a memory location denoted by a 16-bit value stored in `RLocation`. |
| `0x32` | JMPT | `JMPT RCondition AddrHigh AddrLow` OR `JMPT RCondition #labelName` | Jumps to a memory location denoted by a 16-bit value (`AddrHigh AddrLow`) if the value of a given register is not 0\.  Labels are supported in ASM, replacing the address |
| `0x33` | JMPTi | `JMPTi RCondition RLocation PAD` | Indirectly jumps to a memory location denoted by a 16-bit value (`AddrHigh AddrLow`) if the value of a given register is not 0\. |

| *OpCode* | *Name* | *Format* | *Description* |
| :---: | :---: | :---: | :---: |
| `0x40` | SET | `SET RDest ValueHigh ValueLow` | Sets the contents of `RDest` to the literal value represented by `ValueHigh ValueLow` |
| `0x41` | COPY | `COPY RDest RSource PAD` | Copies the contents of `RSource` to `RDest` |
| `0x42` | LOAD | `LOAD RDest AddrHigh AddrLow` | Loads the value from memory at `AddrHigh AddrLow` into `RDest` |
| `0x43` | LOADi | `LOADi RDest RSource PAD` | Indirectly loads the value from memory at the address stored in `RSource` into `RDest` |
| `0x44` | STR | `STR RSource AddrHigh AddrLow` | Stores the contents of `RSource` into memory at `AddrHigh AddrLow` |
| `0x45` | STRi | `STRi RSource RDest PAD` | Indirectly stores the contents of `RSource` into the memory location specified by the contents of `RDest` |
| `0x46` | PUSH | `PUSH RSource PAD PAD` | Pushes the contents of `RSource` onto the stack |
| `0x47` | POP | `POP RDest PAD PAD` | Pops the value on top of the stack into `RDest` |

| *OpCode* | *Name* | *Format* | *Description* |
| :---: | :---: | :---: | :---: |
| `0x50` | CLR | `CLR PAD PAD PAD` | Clears the screen |
| `0x51` | DRAW | `DRAW XLocation YLocation Color` | Sets the color of a pixel at the specified X,Y coordinates to a given color (XTerm256 format). |
| `0x52` | DRAWi | `DRAWi RXLocation RYLocation RColor` | Sets the color of a pixel at the specified X,Y coordinates to a given color (XTerm256 format). |
