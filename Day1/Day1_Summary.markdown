## Day 1 Summary

| **Topic**              | **Key Points**                                                                 |
|------------------------|--------------------------------------------------------------------------------|
| **Overview**           | Introduces Verilog RTL design, test bench concepts, and simulation workflow using Icarus Verilog. |
| **RTL Design**         | Defines digital system behavior at the register-transfer level; written in Verilog modules. |
| **Test Bench**         | Applies input stimuli to verify design functionality; observes outputs without primary IO. |
| **Simulation Workflow**| Simulator executes RTL + test bench; tracks input changes; generates .vcd for GTKWave visualization. |
| **Lab Setup**          | Creates working directories, clones GitHub repository, and explores Verilog files for hands-on exercises. |
| **Multiplexer Simulation** | Compile RTL + test bench with iverilog; run executable; visualize signals in GTKWave. |
| **Yosys Synthesis**    | Transforms RTL into gate-level netlist; includes reading library, synthesizing top module, technology mapping, visualizing, and writing netlist. |
| **Netlist Verification**| Simulate synthesized netlist with existing test bench to confirm functional equivalence with RTL. |