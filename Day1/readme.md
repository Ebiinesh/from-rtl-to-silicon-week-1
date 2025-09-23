# 📘 Week 1 – Day 1: Fundamentals of RTL Design & Simulation

Welcome to **Day 1** of the RISC-V SoC Tapeout journey!  
This session sets the foundation for **RTL design**, **simulation**, and the **open-source toolchain**. Before jumping into fabrication-oriented flows, it’s crucial to first understand how digital circuits are modeled, tested, and verified at the RTL level.

---

## 🧩 1. Design, Testbench & Simulator

Digital design is not just about writing Verilog — it’s about creating a system that behaves exactly as intended. Three key concepts govern this process:

### 🔹 Design
- Functional description of hardware written in **HDL (Hardware Description Language)**.
- Example: A 2-to-1 multiplexer described in Verilog.
- Think of it as the **blueprint** of the digital circuit.

### 🔹 Testbench
- A **virtual laboratory** that provides controlled inputs to the design and observes outputs.
- Verifies whether the design meets its functional specification.
- No hardware is needed — everything happens in the simulation domain.

### 🔹 Simulator
- Software that executes both the design and the testbench, mimicking hardware behavior.
- Produces output logs and waveforms to validate correctness.
- Example tool: **Icarus Verilog (`iverilog`)**, open-source and widely used.

> ⚡ **Key Insight**: Without a testbench and simulator, an HDL file is just text. Simulation is what gives “life” to your design before fabrication.

---

## 🔄 2. Simulation Workflow with Icarus Verilog

The **simulation pipeline** can be visualized as:








### 📝 Explanation:
1. **Design File (.v)** – Contains RTL code of the circuit (e.g., mux, counter, adder).
2. **Testbench File (.v)** – Instantiates the design, applies stimuli, and records responses.
3. **Compilation** – `iverilog` compiles design + testbench into a simulation executable.
4. **Execution** – Running the executable simulates circuit behavior.
5. **Waveform Viewing** – The `.vcd` file captures signal transitions; tools like **GTKWave** display them graphically.

![Icarus Verilog Simulation Flow](Day1/images/iverilog.png)
---

## 📂 3. The Role of the `.vcd` File

- `.vcd` = **Value Change Dump**
- A textual log of every signal transition during simulation.
- Useful for answering:
  - When did `sel` switch from `0` to `1`?
  - What was `y` when `i0=1` and `i1=0`?
- **GTKWave** visualizes these transitions as waveforms for easier debugging.

---

## ✅ Day 1 Takeaways

- **Design** describes hardware functionality.
- **Testbench** provides the testing environment.
- **Simulator (iverilog)** validates correctness.
- **`.vcd` files** are critical artifacts for waveform visualization and debugging.

---

> Next: Lab session – simulate a multiplexer in `iverilog`, generate a `.vcd`, and view waveforms in GTKWave.
