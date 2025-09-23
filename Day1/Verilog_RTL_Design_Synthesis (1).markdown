# 1. Introduction to Verilog RTL Design and Synthesis

Welcome to the first day of our workshop! We'll dive into the essentials of RTL design using Verilog, explore simulation with Icarus Verilog (iverilog), and get hands-on with tools like GTKWave and Yosys. This session includes practical labs to solidify your understanding.

## 1.1 Overview of Icarus Verilog, RTL Design, and Test Bench

* **RTL Design**: This is the core Verilog code (or set of modules) that implements the functionality needed to match your project's specifications. Think of Register Transfer Level (RTL) as a blueprint for digital circuits—it's the go-to abstraction in modern electronic design flows, captured via HDLs like Verilog or VHDL.

  *Example*: A simple 2-input AND gate in Verilog might look like this:
  ```
  module and_gate (
      input a,
      input b,
      output y
  );
      assign y = a & b;
  endmodule
  ```
  This RTL code describes the gate's behavior without delving into physical transistors.

* **Test Bench**: Your verification powerhouse! It's a setup to feed input stimuli (test vectors) into the design and check if the outputs align with expectations. By simulating various scenarios, you ensure the design works as intended.

  *Example*: For the AND gate above, a basic test bench could stimulate inputs like (a=0, b=0) expecting y=0, or (a=1, b=1) expecting y=1.

## 1.2 Running Simulations with Icarus Verilog

* **Simulation**: This is where your HDL code comes to life! After compiling and elaborating, the design executes in a virtual environment to test functional correctness. It's crucial for spotting bugs early in Verilog or VHDL designs.

* **Simulator**: Meet Icarus Verilog (iverilog)—our tool of choice for running simulations. It takes your RTL implementation and verifies it against specs through controlled testing.

* **How Simulators Operate**: Simulators are event-driven: they watch for input changes and only recompute outputs when needed. Unchanged inputs? No unnecessary work! Results are logged to a file for analysis.

  *Example*: If an input flips from 0 to 1 in a flip-flop design, the simulator evaluates the clock edge and updates the output accordingly, dumping the change to a .vcd file.

## 1.3 Configuring Design and Test Bench

* Your Verilog RTL design features primary inputs (one or more) and outputs—think of them as the circuit's entry and exit points.
* Apply stimuli to inputs and monitor outputs: This involves a stimulus generator (inputs) and observer (outputs).
* Craft a test bench to drive the design: Instantiate the module inside it and simulate real-world inputs.
* Key fact: Test benches have no primary I/O—they're purely for simulation.

Below image illustrates the test bench configuration:  
![](DAY_1/Test_bench_setup.png)

*Example Setup*: For a counter design with inputs (clk, reset) and output (count), the test bench might toggle the clock 10 times and assert reset midway to verify counting behavior.

## 1.4 Icarus Verilog Simulation Workflow

* **Simulator Inputs**:  
  iverilog needs two key pieces:  
  1. **RTL Design**: Verilog code describing the hardware behavior.  
  2. **Test Bench**: Instantiates the design and applies test vectors for functional checks.

  *Example*: Design file: `counter.v`; Test bench: `tb_counter.v` with clock generation and reset pulses.

* **Simulator Output**:  
  A Value Change Dump (.vcd) file captures signal changes, viewable in GTKWave for waveform analysis.

Below image depicts the complete iverilog simulation workflow:  
![](DAY_1/iverilog_sim_flow.png)

## 1.5 Lab Environment Setup

Let's get your workspace ready! Follow these steps for a smooth setup.

* Log into your lab instance and create a `VLSI` directory in your home:

```
$ cd /home/user/
$ mkdir VLSI
```

![](DAY_1/LAB_setup1.1.png)

* Clone the workshop repository:

```
$ cd VLSI
$ git clone https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop.git
```

![](DAY_1/LAB_setup1.2.png)

* Explore the repository structure:

```
$ cd sky130RTLDesignAndSynthesisWorkshop
$ ls -ltr
$ cd my_lib
$ cd lib                       # Sky130 standard cell library
$ cd ..
$ cd verilog_model            # Verilog models for standard cells
```

![](DAY_1/LAB_setup1.3.png)

* Check out the Verilog files for labs:

```
$ cd sky130RTLDesignAndSynthesisWorkshop
$ ls -ltr
$ cd verilog_files
```

![](DAY_1/LAB_setup1.4.png)

*Tip*: Use `ls -ltr` to list files with details—great for spotting recent changes!

## 1.6 Simulating a Multiplexer with Icarus Verilog

Time for action! Simulate a 2:1 Multiplexer (MUX) to see the workflow in practice.

The process:
* iverilog compiles design + test bench into `a.out`.
* Run `a.out` to produce a .vcd file.
* View waveforms in GTKWave.

Detailed Steps:
1. Compile with iverilog:

```
$ cd sky130RTLDesignAndSynthesisWorkshop
$ ls -ltr
$ cd verilog_files
$ iverilog good_mux.v tb_good_mux.v
```

  *Example Outcome*: Creates `a.out` executable.

2. Execute the simulation:

```
$ ./a.out
```

  *Example*: For a MUX with inputs A=1, B=0, Sel=0, it should output 1 (selecting A).

3. Visualize in GTKWave:

```
$ gtkwave tb_good_mux.vcd
```

4. Add signals to the waveform viewer by dragging them over.  
![](DAY_1/good_mux_waveform.png)

*Pro Tip*: Zoom in on waveforms to inspect transitions closely!

## 1.7 Synthesizing Designs with Yosys

* **Synthesis**: The magic step where abstract RTL becomes a concrete circuit! Using HDLs like Verilog, tools generate a Gate-Level Netlist—a map of gates and wires. Yosys is our synthesis hero here.

In a nutshell:
* RTL → Gates + Connections.
* Output: Netlist file for further design stages.

*Example*: Synthesizing the AND gate RTL might map to a single AND cell from the library.

### 1.7.1 Yosys Synthesis Workflow

Inputs: RTL design + Liberty file (.lib). Output: Gate-level netlist.

Below image illustrates the Yosys synthesis workflow:  
![](DAY_1/Yosys_setup.png)

Step-by-Step Synthesis for `good_mux.v`:
1. Launch Yosys:

```
$ cd verilog_files
$ yosys
```

![](DAY_1/yosys_invoke.png)

2. Load the library:

```
$ read_liberty -lib ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

  **Explanation**: `-lib` imports cell definitions as blackboxes.

3. Load your design:

```
$ read_verilog good_mux.v
```

![](DAY_1/Yosys_setup_2.png)

4. Perform synthesis:

```
$ synth -top good_mux
```

  *Example*: For a MUX, this elaborates the logic into basic gates.

![](DAY_1/Yosys_setup_3.png)

5. Technology mapping:

```
$ abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

  *Example*: Maps abstract gates to Sky130 library cells like `sky130_fd_sc_hd__mux2_1`.

![](DAY_1/Yosys_setup_4.png)

6. Visualize the design:

```
$ show
```

  *Tip*: This generates a diagram—perfect for debugging logic!

![](DAY_1/Yosys_setup_5.png)

7. Export the netlist:

```
$ write_verilog -noattr good_mux_netlist.v
```

![](DAY_1/Yosys_setup_6.png)

### 1.7.2 Verifying the Synthesized Netlist

The netlist uses Sky130 cells—simulate it to confirm fidelity to the original RTL.

* Reuse the same test bench, swap RTL file with netlist in iverilog.
* Inputs/Outputs match, so verification is straightforward.

*Example*: If RTL MUX passes tests (e.g., Sel=1 outputs B), the netlist should too—mismatches indicate synthesis issues.