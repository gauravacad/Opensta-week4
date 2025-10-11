# Part 3 – Generate Timing Graphs with OpenSTA
## Objective
- The purpose of this README is to document the OpenSTA timing analysis workflow, including installation, running example analyses, and comparing timing slacks for different designs.
- This task focuses on understanding timing behavior, slack calculation, and critical path evaluation using OpenSTA.

## Installation OpenSTA with dependencies 

### CUDD Installation
``` bash
$ git clone https://github.com/ivmai/cudd.git
$ cd cudd
$ aclocal -I . ; autoheader; autoconf; automake --add-missing -c
$ ./configure --enable-obj
$ make
$ make check
...
============================================================================
Testsuite summary for cudd 3.0.0
============================================================================
# TOTAL: 30
# PASS:  30
# SKIP:  0
# XFAIL: 0
# FAIL:  0
# XPASS: 0
# ERROR: 0
============================================================================

# lastly
sudo make install 
```
<img width="829" height="720" alt="image" src="https://github.com/user-attachments/assets/b9ac5aeb-f4c9-470b-923c-6a9575f68008" />

### Install Swig

``` bash
sudo apt install swig
```

<img width="814" height="494" alt="image" src="https://github.com/user-attachments/assets/4841f05c-a47d-47cf-ae16-c7b207393396" />


### Installation of OpenSTA
```
$ sudo apt-get update
$ sudo apt-get install build-essential tcl-dev tk-dev cmake git libeigen3-dev autoconf m4 perl automake 

$ git clone https://github.com/The-OpenROAD-Project/OpenSTA.git
$ cd OpenSTA
$ mkdir build
$ cd build
$ cmake .. -DUSE_CUDD=ON -DCUDD_DIR=$HOME/cudd
$ make
$ sudo make install

## RUN the command on any path typing `sta`
$ sta
```
<img width="819" height="369" alt="image" src="https://github.com/user-attachments/assets/6c8bd1b2-0d9b-4266-be2d-0f7edfcc5072" />


---
## Example Timing Analysis

### Step 1 : Verilog Module
- This module is given by OpenSTA as `example1.v` 

```verilog
module top (in1, in2, clk1, clk2, clk3, out);
  input in1, in2, clk1, clk2, clk3;
  output out;
  wire r1q, r2q, u1z, u2z;

  DFF_X1 r1 (.D(in1), .CK(clk1), .Q(r1q));
  DFF_X1 r2 (.D(in2), .CK(clk2), .Q(r2q));
  BUF_X1 u1 (.A(r2q), .Z(u1z));
  AND2_X1 u2 (.A1(r1q), .A2(u1z), .ZN(u2z));
  DFF_X1 r3 (.D(u2z), .CK(clk3), .Q(out));
endmodule 
```
---
### Step 2: Synthesis

```bash
cd /OpenSTA/examples/
yosys
read_liberty -lib nangate45_slow.lib.gz
read_verilog example1.v
synth -top top
show
```
**Screenshot** : Synthesizing terminal output

<img width="820" height="525" alt="yosys" src="https://github.com/user-attachments/assets/c19ad4bc-80a4-410c-b35c-4eb5d9bf8d27" />


---

**Screenshot** : Stats of cells used 

<img width="820" height="525" alt="yosys" src="https://github.com/user-attachments/assets/c977c500-7acb-4236-bd4c-91bf0fa5d77f" />

---
**Screenshot** : Synthesizing Output

<img width="611" height="503" alt="show" src="https://github.com/user-attachments/assets/5b9ada5c-ddd3-40f8-855f-b5a96f40fed4" />

---

### Step 3 : Timing analysis using OpenSTA

```bash
 sta
```
- After launching the OpenSTA interactive shell (denoted by the `%` prompt), you can run the following commands to carry out a basic static timing analysis:

```bash
# Load the standard cell timing library (Liberty format)
read_liberty ./OpenSTA/examples/nangate45_slow.lib.gz

# Load the gate-level Verilog netlist
read_verilog ./OpenSTA/examples/example1.v

# Link the top-level module with the loaded timing library
link_design top

# Define a 10 ns clock named 'clk' for inputs clk1, clk2, and clk3
create_clock -name clk -period 10 {clk1 clk2 clk3}

# Set input delay of 0 ns for signals in1 and in2 relative to clock 'clk'
set_input_delay -clock clk 0 {in1 in2}

# Generate a timing check report for the design (By default it do setup/max checks)
report_checks

# Generate a timing check report for min check 
report_checks -path_delay min
```
**Screenshot**: Terminal Output of STA Commands

<img width="819" height="558" alt="report" src="https://github.com/user-attachments/assets/88c32ece-0918-4874-8f30-3f07cf77edab" />
---


**Screenshot**:  Max path timing report

<img width="820" height="718" alt="timing1" src="https://github.com/user-attachments/assets/c96df76a-d93f-41a9-909e-7c877d21f529" />
---

**Screenshot** : Min path timing report
<img width="816" height="655" alt="timing2" src="https://github.com/user-attachments/assets/be179e6f-b5cc-422a-84a3-2ee3accbd526" />

---
## Analysis of Timing :

| **Parameter** | **Description**      | **Value (ns)** |
|:--------------|:---------------------|:--------------:|
| **r2/Q**      | Clock-to-Q delay      | 0.23 |
| **u1**        | Buffer delay          | 0.08 |
| **u2**        | AND2 delay            | 0.10 |
| **Clock Period** | Clock period                  | 10.00 |
| **Library Setup Time** | Setup time           | -0.16 |

---

## Timing Calculations
---
## Set Up Time Analysis

**Data Arrival Time (tarrival):**
```bash 
tarrival = tclk_q + tbuf + tand
tarrival = 0.23 + 0.08 + 0.10 = 0.41 ns
```

**Data Required Time (trequired):**
```bash
trequired = Tclk - tsetup
trequired = 10 - 0.16 = 9.84 ns
```
**Slack Calculation**
```bash
Slacksetup = trequired - tarrival
Slacksetup = 9.84 - 0.41 = 9.43 ns (Positive → MET)
```

###  Observation

- The **setup slack is positive (9.43 ns)** → **timing is met**.  
- There is a **large timing margin**, meaning the circuit can tolerate extra delay.  
- The **negative setup time (-0.16 ns)** effectively increases the available time, aiding timing closure.

---

##  Hold Time Analysis

For the **hold check**, we consider the **shortest data path**.

**Data Arrival Time (tarrival_min):** 
```bash 
tarrival_min = tclk_q + tcomb_min
tarrival_min = 0.00 ns
```

**Data Required Time (trequired_hold):**  
```bash
trequired_hold = thold
trequired_hold = 0.01 ns
```
**Slack Calculation**
```bash
Slackhold = tarrival_min − trequired_hold
Slackhold = 0.00 − 0.01
Slackhold = −0.01 ns (VIOLATED)
```
### Observation
- The **hold slack is negative (−0.01 ns)** → **Hold timing is violated**.  
- This indicates that data is arriving **too early** at the capture flip-flop before it becomes stable for the next clock edge.  
- The path therefore **fails the hold requirement**, causing a potential data corruption risk.



**Screenshot** : The timing report were analysed and Verified.

<img width="750" height="252" alt="image" src="https://github.com/user-attachments/assets/a7deaf73-f131-42a2-91b9-17fc80a71ce8" />

---

## SPEF-Based Timing Analysis

**Definition:**  
**SPEF** is a standardized textual file format that provides detailed parasitic information (resistance, capacitance, and sometimes inductance) of a chip’s routing interconnects, which is used for **accurate post-layout timing analysis (STA)**.

### Key Points:
- Captures **RC parasitics** of nets after placement and routing.  
- Used by **Static Timing Analysis (STA) tools** to compute **delay, slew, and crosstalk effects**.  
- Ensures **timing closure** by reflecting realistic interconnect delays.  
- Comes in two main types:  
  - **Unit SPEF:** resistances in ohms, capacitances in farads.  
  - **Scaled SPEF:** values scaled by a factor for readability.  

**Summary:**  
SPEF bridges the gap between layout parasitics and accurate timing analysis.

---
### Steps to do SPEF Based Timing Analysis
```bash
# Change to the directory containing OpenSTA examples
cd OpenSTA/examples

# Invoke the OpenSTA tool
sta

# Load the standard cell timing library (Liberty format)
read_liberty ./nangate45_slow.lib.gz

# Load the gate-level Verilog netlist for analysis
read_verilog ./example1.v

# Link the top-level module in the Verilog netlist with the loaded timing library
link_design top

# Load the parasitic SPEF file for accurate delay calculation
read_spef ./example1.dspef

# Define a 10 ns clock named 'clk' for signals clk1, clk2, and clk3
create_clock -name clk -period 10 {clk1 clk2 clk3}

# Set input delay of 0 ns for signals in1 and in2 relative to the clock 'clk'
set_input_delay -clock clk 0 {in1 in2}

# Generate timing report for max check
report_checks

# Generate timing report for min check
report_checks -path_delay min
```
**Screenshot** : Terminal output of the tcl script with SPEF Based Max path Check 
<img width="825" height="760" alt="reportspef" src="https://github.com/user-attachments/assets/f7aff652-bed3-4117-a357-770366764e23" />


---
**Screenshot**: SPEF Based Min path Check still voilated

<img width="820" height="637" alt="reportspef2" src="https://github.com/user-attachments/assets/3ada4f75-3fff-46e7-baf1-60e50fac7aa9" />

---

### Observation

- Including **SPEF parasitics** significantly increases the **delays** along the data path.  
- **Data arrival time** at `r3/D` increases from **0.41 ns → 7.92 ns**.  
- **Slack decreases** from **9.43 ns → 1.52 ns**, though it still meets the setup requirement.  
- The increase in delay is mainly due to **interconnect capacitance and resistance** captured in the SPEF file.  
- This highlights the importance of considering **post-layout parasitics** for accurate timing analysis.
---

## Note:
- The `Timing analysis of VSDBabySoc` and `Multi-Corner PVT` analysis are done in **VSDBabySoC_Timing_Analysis.md**

---


# Timing Analysis of VSDBabySoc
---
## Steps to do Timing analysis of VSDBabySoC

```bash
cd Desktop/SoC/VSDBabySoC
sta

# Load Liberty Libraries (standard cell + IPs)
read_liberty  ./src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_liberty  ./src/lib/avsdpll.lib
read_liberty ./src/lib/avsddac.lib

# Read Synthesized Netlist
read_verilog ./src/module/vsdbabysoc.synth.v

# Link the Top-Level Design
link_design vsdbabysoc

# Apply SDC Constraints
read_sdc ./src/sdc/vsdbabysoc_synthesis.sdc
 
#SDC Constraints
set_units -time ns
create_clock [get_pins {pll/CLK}] -name clk -period 11

# Generate Timing Report (by default max path)
report_checks

# Generate Timing Report for min path
report_checks -path_delay min
```
**Screenshot**: VSDBabySoC Timing report for max path

<img width="813" height="623" alt="1" src="https://github.com/user-attachments/assets/370dd610-9e13-4d14-a117-3a8277df0184" />
---

**Screenshot**: VSDBabySoC Timing report for min path

<img width="825" height="705" alt="2" src="https://github.com/user-attachments/assets/e00991ad-a9fe-440a-8bad-dd3d4ea64f08" />

---

 # Multi-PVT Corner Timing Analysis of VSDBabySoC using OpenSTA
---
## Multi-PVT Corners in STA

**Definition:**  
In **Static Timing Analysis (STA)**, **Multi-PVT Corners** refer to evaluating the design under multiple combinations of **Process, Voltage, and Temperature (PVT)** conditions. These corners help ensure that the design **meets timing constraints across all expected operating conditions**.

### Components of PVT:

1. **Process (P):**  
   - Variations in manufacturing, e.g., **slow (SS), typical (TT), fast (FF)** transistors.
   - Accounts for **chip-to-chip variability**.

2. **Voltage (V):**  
   - Variations in supply voltage, e.g., **nominal, high, low**.
   - Ensures timing is robust against **power supply fluctuations**.

3. **Temperature (T):**  
   - Variations in operating temperature, e.g., **-40°C, 25°C, 125°C**.
   - Models the effect of **thermal conditions** on transistor speed.

### Purpose of Multi-PVT Corners:

- STA at a **single corner** is insufficient for real-world conditions.  
- Multi-PVT analysis ensures the **design meets timing and functional requirements** under all expected scenarios:  
  - **Fast Process + High Voltage + Low Temperature** → fastest circuits, check **hold violations**.  
  - **Slow Process + Low Voltage + High Temperature** → slowest circuits, check **setup violations**.  

  **Example Scenarios:**

- **Fast Corner:** FF transistors at **-40 °C, 1.95 V** → circuits are faster; checks **hold violations** (data may arrive too early).  
- **Slow Corner:** SS transistors at **100 °C, 1.40 V** → circuits are slower; checks **setup violations** (data may arrive too late). 
---
## Timing Libraries for Multi-PVT Analysis

The timing libraries required for this analysis can be downloaded from the **SkyWater PDK**:

- **SkyWater PDK – sky130_fd_sc_hd Timing Libraries**  
  - These libraries provide **process-, voltage-, and temperature-specific timing models** needed for accurate STA.  
  - You can download them from the official [SkyWater PDK repository](https://github.com/google/skywater-pdk) or the timing library package link provided by the PDK.
---
 
## Using the Multi-PVT TCL Script for STA

We are going to use the `multi_pvt_corners.tcl` script to **automate Static Timing Analysis (STA) across multiple PVT corners**. This allows us to verify that the design meets timing constraints under all **process, voltage, and temperature conditions** without manually switching libraries or rerunning analyses.

### Steps Performed by the Script:

1. **Load PVT-specific timing libraries**  
   - Example: `sky130_fd_sc_hd__ss_100C_1v40.lib` for a **slow-slow, high-temperature, low-voltage** corner.  

2. **Link the synthesized netlist**  
   - Ensures the same RTL design is used for all corners.  

3. **Apply SDC constraints**  
   - Clocks, input/output delays, and timing exceptions are applied consistently for each corner.  

4. **Run timing checks**  
   - Includes **setup, hold, worst negative slack (WNS), total negative slack (TNS)** for each corner.  

5. **Save detailed reports**  
   - Generates a **separate report for each PVT corner** under `./sta_outputs/` for analysis.  

### Benefits:

- Provides **comprehensive timing validation** across all operating conditions.  
- Automates repetitive STA tasks, saving **time and effort**.  
- Identifies **worst-case paths** for setup and hold, ensuring reliable chip operation.  

---

## Script to run Static Timing Analysis for all corners

```bash
#---------------------------------------------
#  Multi-corner STA Automation Script (OpenSTA)
#---------------------------------------------

# Define list of timing libraries (corners)
set list_of_lib_files {
    sky130_fd_sc_hd__ff_n40C_1v95.lib
    sky130_fd_sc_hd__ff_100C_1v65.lib
    sky130_fd_sc_hd__ff_100C_1v95.lib
    sky130_fd_sc_hd__ff_n40C_1v56.lib
    sky130_fd_sc_hd__ff_n40C_1v65.lib
    sky130_fd_sc_hd__ff_n40C_1v76.lib
    sky130_fd_sc_hd__ss_100C_1v40.lib
    sky130_fd_sc_hd__ss_100C_1v60.lib
    sky130_fd_sc_hd__ss_n40C_1v28.lib
    sky130_fd_sc_hd__ss_n40C_1v35.lib
    sky130_fd_sc_hd__ss_n40C_1v40.lib
    sky130_fd_sc_hd__ss_n40C_1v44.lib
    sky130_fd_sc_hd__ss_n40C_1v76.lib
    sky130_fd_sc_hd__ss_n40C_1v60.lib
    sky130_fd_sc_hd__tt_025C_1v80.lib
    sky130_fd_sc_hd__tt_100C_1v80.lib
}

#---------------------------------------------
#  Load base cell libraries and design files
#---------------------------------------------
read_liberty ./src/lib/avsdpll.lib
read_liberty ./src/lib/avsddac.lib

#---------------------------------------------
#  Create output folder
#---------------------------------------------
file mkdir sta_outputs




