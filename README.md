# Week 3 - RISC-V SoC Tapeout Journey 
## Gate-Level Simulation & Static Timing Analysis using OpenSTA
---

## Objective

- The goal of **Week 3** is to **understand and perform Gate-Level Simulation (GLS)** after synthesis, validate the **functional correctness of the synthesized netlist**, and gain a practical understanding of **Static Timing Analysis (STA)** concepts using **OpenSTA**.  
- This week bridges the gap between **functional verification** and **timing validation**, ensuring that the synthesized design meets both **logical** and **timing constraints**.

---

## Week 3 Structure

### **Part 1 – Post-Synthesis GLS**

- Performed **Gate-Level Simulation** using the synthesized netlist and standard cell library of VSDBabySoC.
- Verified **functional equivalence** between RTL and post-synthesis simulation.
- Understood the significance of **SDF (Standard Delay Format)** for accurate timing-based simulations.
- Observed how **timing delays and gate-level optimizations** influence the design’s output behavior.



---

### **Part 2 – Fundamentals of STA (Static Timing Analysis)**

- Learned the STA From `Static Timing Analysis-I` course by Kunal Ghosh sir 
- Covered **Core Concepts** like :
  - **Setup time**, **hold time**, and **slack**
  - **Clock latency**, **skew**, **jitter**, **timing paths**, **OCV and Its Timing Analysis**
- Understood how **STA differs from dynamic simulation** and why it’s essential for verifying **timing closure**.


---

###  **Part 3 – Generate Timing Graphs with OpenSTA**

- Installed and configured **OpenSTA** with necessary dependencies.
- Performed a basic `timing analysis` to check for **setup and hold violations**.
- Also performed timing analysis for `VSDBabySoC` and `Multi Corner PVT analysis`
- Used **TCL scripts** to read the synthesized netlist, constraints, and liberty files.
- Generated **timing graphs** and evaluated **slack**, **critical paths**, and **timing margins** across multiple PVT corners.
- Interpreted **WNS (Worst Negative Slack)** and **TNS (Total Negative Slack)** to assess timing quality.



---

##  Author

**Gaurav Purohit**

---

## Conclusion

- Throughout Week 3, I learned to connect the dots between **synthesis**, **simulation**, and **timing verification**.  
- Performing Gate-Level Simulation gave me hands-on exposure to real timing delays, while OpenSTA introduced me to **static timing validation**, a crucial skill in ASIC and FPGA design flows.  

By the end of this week, I could:
- Confidently perform **post-synthesis GLS** to verify netlist correctness.  
- Understand **setup/hold time, slack, skew, and clocking effects**.  
- Use **OpenSTA** to analyze timing and identify **critical paths**.  

This week strengthened my foundation in **timing analysis** and gave me practical insights into **timing closure and design performance optimization**.

---

**Completed as part of Week 3 learning and experimentation on Gate-Level Simulation and Static Timing Analysis.**
