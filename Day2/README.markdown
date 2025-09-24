# 1. Day 2 - Timing Libraries, Hierarchical vs. Flat Synthesis, and Efficient Flip-Flop Coding

Welcome to Day 2 of our workshop on digital design using the SKY130 process! This section introduces key concepts of timing libraries, the differences between hierarchical and flat synthesis, and best practices for coding flip-flops to optimize design performance. The content is structured to provide a clear understanding for beginners while offering practical insights for advanced users.

## Table of Contents
1. [Introduction to Timing Libraries](#1.1-introduction-to-timing-libraries)
   - [Library Naming Conventions](#1.1.1-library-naming-conventions)
   - [Understanding Liberty Files (.lib)](#1.1.2-understanding-liberty-files)
2. [Hierarchical vs. Flat Synthesis](#1.2-hierarchical-vs-flat-synthesis)
   - [Hierarchical Synthesis](#1.2.1-hierarchical-synthesis)
   - [Flat Synthesis](#1.2.2-flat-synthesis)
   - [Submodule-Level Synthesis](#1.2.3-submodule-level-synthesis)
3. [Efficient Flip-Flop Coding Styles](#1.3-efficient-flip-flop-coding-styles)
   - [Why Flip-Flops Are Essential](#1.3.1-why-flip-flops-are-essential)
   - [Flip-Flop Coding Styles](#1.3.2-flip-flop-coding-styles)

---

## 1.1 Introduction to Timing Libraries

Timing libraries are critical in digital design, providing detailed information about the behavior of standard cells in a specific technology node. This section explores the naming conventions and structure of these libraries, focusing on the SKY130 process.

### 1.1.1 Library Naming Conventions

The SKY130 process, developed by SkyWater Technology, offers seven standard cell libraries tailored for different design goals, such as high density or high performance, with three cell height variations. In this workshop, we use the `sky130_fd_sc_hd__tt_025C_1v80.lib` library, optimized for **high density**. This library balances gate density and low power consumption but sacrifices some drive strength compared to other libraries.

The naming of SKY130 libraries follows a consistent structure:
```
<Process>_<Source>_<Type>[_<Variant>]
```

Let’s break down the components of `sky130_fd_sc_hd__tt_025C_1v80.lib`:
- **sky130**: Refers to the SKY130 process technology.
- **fd**: Indicates the library is provided by SkyWater Foundry.
- **sc**: Denotes digital standard cells.
- **hd**: Specifies high-density cells.
- **tt**: Represents the typical process corner.
- **025C**: Indicates the operating temperature (25°C).
- **1v80**: Specifies the operating voltage (1.8V).

This naming helps designers quickly identify the library’s purpose and characteristics.

### 1.1.2 Understanding Liberty Files

Liberty files (`.lib`) are an IEEE-standardized format used to describe the characteristics of standard cells in a technology node. These ASCII files, typically with a `.lib` extension, contain critical data such as:
- Timing characteristics (e.g., propagation delay).
- Power consumption (dynamic and leakage).
- Input/output relationships.
- Noise parameters.

For example, in `sky130_fd_sc_hd__tt_025C_1v80.lib`, you’ll find definitions for various standard cells, such as logic gates and flip-flops. Let’s examine a specific cell, `sky130_fd_sc_hd__a21110_1`, which implements a logic function with five inputs: an AND of two inputs ORed with three others. The file also details leakage power for all 32 possible input combinations.

![](/DAY_2/Lib_details.png)

Additionally, the library includes different versions of the same logic gate, such as the AND gate, with varying sizes and characteristics:
![](/DAY_2/and_versions.png)

Comparing different AND gate variants (`and2_0`, `and2_2`, `and2_4`):
- **Area**: `and2_0` < `and2_2` < `and2_4`
- **Power**: `and2_0` < `and2_2` < `and2_4`
- **Delay**: `and2_0` > `and2_2` > `and2_4`

**Key Insight**:
- **Larger cells** (e.g., `and2_4`) use wider transistors, consuming more area and power but offering faster performance (less delay).
- **Smaller cells** (e.g., `and2_0`) use narrower transistors, reducing area and power but increasing delay.

This trade-off is crucial when selecting cells for your design.

---

## 1.2 Hierarchical vs. Flat Synthesis

Synthesis transforms RTL code into a gate-level netlist using cells from a library. This section compares hierarchical and flat synthesis approaches and discusses submodule-level synthesis for large designs.

### 1.2.1 Hierarchical Synthesis

Consider an RTL design with a top module instantiating two submodules, as shown below:
![](/DAY_2/mutiple_modules.png)

In hierarchical synthesis, the tool preserves the module hierarchy in the netlist. Instead of replacing submodules with their gate-level implementations, the synthesis tool retains references to the submodules (`u1` and `u2`). The resulting netlist reflects this structure:
![](/DAY_2/multiple_hier_net.png)

Here’s an example of how hierarchical synthesis is performed using Yosys:
```
$ read_liberty -lib ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
$ read_verilog multiple_modules.v
$ synth -top multiple_modules
$ abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
$ write_verilog -noattr multiple_modules_hier.v
```

The output netlist (`multiple_modules_hier.v`) maintains the hierarchy, referencing submodules instead of individual gates.

### 1.2.2 Flat Synthesis

Flat synthesis, on the other hand, collapses the hierarchy, replacing submodules with their gate-level implementations. The Yosys `flatten` command achieves this by substituting submodules with their corresponding standard cells from the library.

Here’s the process:
```
$ read_liberty -lib ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
$ read_verilog multiple_modules.v
$ synth -top multiple_modules
$ abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
$ write_verilog -noattr multiple_modules_hier.v
$ flatten
$ write_verilog -noattr multiple_modules_flat.v
```

The flattened netlist (`multiple_modules_flat.v`) replaces submodules with gates, as shown below:
![](/DAY_2/Flatten_net.png)

The difference between hierarchical and flat netlists is illustrated here:
![](/DAY_2/Hier_vs_Flat.png)

**Key Difference**:
- **Hierarchical Synthesis**: Preserves module boundaries, making the design easier to debug and modify.
- **Flat Synthesis**: Simplifies the netlist by removing hierarchy, which may improve optimization but makes the design harder to trace.

### 1.2.3 Submodule-Level Synthesis

For designs with multiple instances of the same submodule, synthesizing at the submodule level can save time. Instead of synthesizing each instance, the tool synthesizes one instance and reuses it. This approach also reduces the synthesis tool’s workload for large designs by breaking them into smaller, manageable pieces.

For example, to synthesize only `sub_module1` in `multiple_modules.v`:
```
$ read_liberty -lib ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
$ read_verilog multiple_modules.v
$ synth -top sub_module1
$ abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
$ write_verilog -noattr multiple_modules_sub.v
```

The resulting netlist includes only `sub_module1`, ignoring `sub_module2`:
![](/DAY_2/sub_net.png)

This focused synthesis is useful for optimizing specific parts of a design independently.

---

## 1.3 Efficient Flip-Flop Coding Styles

Flip-flops are essential for managing glitches and ensuring stable outputs in digital circuits. This section explains why flip-flops are needed and explores different coding styles for optimal performance.

### 1.3.1 Why Flip-Flops Are Essential

In combinational circuits, each gate has a propagation delay, causing output transitions to lag behind input changes. If delays vary across gates, unwanted transitions (glitches) can occur. For example, consider this circuit, where the output should always be 1 due to Boolean logic:
![](/DAY_2/Glitch&wave.png)

However, differing gate delays can cause glitches, as shown in the timing diagram. More complex combinational paths increase the likelihood of glitches.

**Solution**: Flip-flops act as storage elements, capturing inputs only on clock edges and isolating outputs between edges. By placing flip-flops between combinational logic stages, glitches are prevented from propagating, ensuring stable inputs and outputs.

**Important Note**: Always define the initial state of flip-flops to avoid undefined (X) states, which can cause unpredictable behavior in downstream logic. Control pins like **SET** (sets output to 1) and **RESET** (sets output to 0) can be synchronous or asynchronous to the clock.

### 1.3.2 Flip-Flop Coding Styles

Below are common flip-flop coding styles, each with specific use cases. When synthesizing flip-flops, use the `dfflibmap` command to map them to the library, as flip-flops may reside in a separate library:
```
dfflibmap -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

1. **Asynchronous Reset D-Flip-Flop**  
   This flip-flop resets its output to 0 asynchronously when the reset signal is asserted.  
   ![](/DAY_2/dff_asynch_res.png)  
   Synthesis command:
   ```
   $ read_liberty -lib ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
   $ read_verilog dff_asyncre.v
   $ synth -top dff_asyncres
   $ dfflibmap -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
   $ abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
   $ show
   ```

2. **Asynchronous Set D-Flip-Flop**  
   Sets the output to 1 asynchronously when the set signal is active.  
   ![](/DAY_2/dff_asynch_set.png)

3. **Synchronous Reset D-Flip-Flop**  
   Resets the output to 0 synchronously on the clock edge when the reset signal is asserted.  
   ![](/DAY_2/dff_synch_res.png)

4. **Asynchronous and Synchronous Reset D-Flip-Flop**  
   Combines both reset types, allowing asynchronous reset for immediate action and synchronous reset for clock-aligned updates.  
   ![](/DAY_2/dff_asynch_sync_res.png)

**Important Note**: Combining asynchronous and synchronous resets does not cause race conditions. However, combining set and reset signals in the same flip-flop can lead to race conditions and should be avoided.

**Example Verilog Code for Asynchronous Reset D-Flip-Flop**:
```verilog
module dff_asyncres (
    input clk,
    input reset,
    input d,
    output reg q
);
    always @(posedge clk or posedge reset) begin
        if (reset)
            q <= 0;
        else
            q <= d;
    end
endmodule
```

This code ensures the flip-flop resets to 0 when `reset` is high, independent of the clock, and otherwise captures the input `d` on the rising clock edge.

---

This guide provides a comprehensive yet accessible introduction to timing libraries, synthesis strategies, and flip-flop coding. Use the provided commands and examples to experiment with your designs in the SKY130 process!