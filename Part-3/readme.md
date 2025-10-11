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

``` bash
## Option 1 - only the timing directory

git clone --no-checkout https://github.com/efabless/skywater-pdk-libs-sky130_fd_sc_hd.git
cd skywater-pdk-libs-sky130_fd_sc_hd
git sparse-checkout init --cone
git sparse-checkout set timing
git checkout
```
 
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

#---------------------------------------------
#  Loop through each .lib file (corner)
#---------------------------------------------
set i 1
foreach lib_file $list_of_lib_files {

    puts "\n=== Running STA for corner: $lib_file ==="

    # Load corner-specific library
    read_liberty ./src/lib/$lib_file

    # Read design and constraints
    read_verilog ./src/module/vsdbabysoc.synth.v
    link_design vsdbabysoc
    current_design vsdbabysoc
    ## we will change this adding the input delay as shown below ##
    read_sdc ./src/sdc/vsdbabysoc_synthesis.sdc 

    # Perform timing checks
    check_setup -verbose

    #-----------------------------------------
    # Generate detailed reports
    #-----------------------------------------
    report_checks \
        -path_delay min_max \
        -fields {nets cap slew input_pins fanout} \
        -digits 4 \
        > ./sta_outputs/min_max_$lib_file.txt

    #-----------------------------------------
    # Save key metrics (WNS, TNS)
    #-----------------------------------------
    exec echo "$lib_file" >> ./sta_outputs/sta_worst_max_slack.txt
    report_worst_slack -max -digits 4 >> ./sta_outputs/sta_worst_max_slack.txt

    exec echo "$lib_file" >> ./sta_outputs/sta_worst_min_slack.txt
    report_worst_slack -min -digits 4 >> ./sta_outputs/sta_worst_min_slack.txt

    exec echo "$lib_file" >> ./sta_outputs/sta_tns.txt
    report_tns -digits 4 >> ./sta_outputs/sta_tns.txt

    exec echo "$lib_file" >> ./sta_outputs/sta_wns.txt
    report_wns -digits 4 >> ./sta_outputs/sta_wns.txt

    incr i
}
puts "\n All corners analysed. Reports saved in ./sta_outputs/"
```
**Screenshot:** All the corners are analysed but the Input delay is missing in the file  `read_sdc ./src/sdc/vsdbabysoc_synthesis.sdc`

<img width="824" height="644" alt="image" src="https://github.com/user-attachments/assets/1feeedf6-905d-44bf-b79a-659a4dd74617" />

---

## New updated Input Delay in SDC Constraints For VSDBabySoC

```bash
# =============================================================================
# SDC Constraints for vsdbabysoc Module (synthesised netlist)
# Generated for OpenSTA Static Timing Analysis
# Clock period: 11 ns (~90.9 MHz)
# =============================================================================

set_units -time ns

# Clock definition
create_clock -name clk -period 11 [get_pins pll/CLK]

set_clock_latency -source 2 [get_clocks clk]
set_clock_latency 1 [get_clocks clk]
set_clock_uncertainty -setup 0.5 [get_clocks clk]
set_clock_uncertainty -hold 0.5 [get_clocks clk]

# Design constraints
set_max_area 8000
set_max_fanout 5 vsdbabysoc
set_max_transition 10 vsdbabysoc

# Input constraints
set_input_delay -clock clk -max 4 [get_ports {reset VCO_IN ENb_CP ENb_VCO REF VREFH}]
set_input_delay -clock clk -min 1 [get_ports {reset VCO_IN ENb_CP ENb_VCO REF VREFH}]
set_input_transition -max 0.4 [get_ports {reset VCO_IN ENb_CP ENb_VCO REF VREFH}]
set_input_transition -min 0.1 [get_ports {reset VCO_IN ENb_CP ENb_VCO REF VREFH}]

# Output constraints
set_load -max 0.5 [get_ports OUT]
set_load -min 0.5 [get_ports OUT]
set_output_delay -clock clk -max 0.5 -clock clk [get_ports OUT]
set_output_delay -clock clk -min 0.5 -clock clk [get_ports OUT]

# Path delay
set_max_delay 10 -from [get_clocks clk] -to [get_ports OUT]
```
## How to run Multi-PVT corner analysis
```bash
# Go to OpenSTA interactive shell (denoted by %)
sta

source multi_pvt_corners.tcl
```
**Screenshot:** Showing the updated corner analysis with input delay

<img width="823" height="662" alt="image" src="https://github.com/user-attachments/assets/7a2c7fb4-458d-4bd0-9124-130167256587" />

---
**Screenshot:** Completed and saved 

<img width="820" height="699" alt="image" src="https://github.com/user-attachments/assets/3062b5da-46a4-425f-b92a-1e16c98344e4" />

---

## STA Output
- After successfull analysis of `multi pvt corners`, it produces 4 output files.
They are.
```bash
sta_tns.txt
sta_wns.txt
sta_worst_max_slack.txt
sta_worst_min_slack.txt
```
**Screenshot:** Showing the output directory Structure 

<img width="815" height="464" alt="image" src="https://github.com/user-attachments/assets/0a6892d6-9361-4d01-8c0f-ab6ead98b440" />

---
**Screenshot of sta_worst_max_slack.txt**

<img width="823" height="704" alt="image" src="https://github.com/user-attachments/assets/880c26db-66f3-4946-a00a-f10acf77e9fe" />


---
**Screenshot of sta_worst_min_slack.txt**

<img width="825" height="703" alt="image" src="https://github.com/user-attachments/assets/7c493e8b-3736-4075-a777-09ebab78e7fe" />


---
**Screenshot of sta_wns.txt**

<img width="817" height="676" alt="image" src="https://github.com/user-attachments/assets/63a2e530-4a8a-4f05-98c9-a549e2af7398" />


---
**Screenshot of sta_tns.txt**

<img width="819" height="676" alt="image" src="https://github.com/user-attachments/assets/e0e55555-e8b8-4575-a790-c426bee0cd63" />

---
# Multi-PVT Timing Summary Report

This report summarises the **setup and hold timing performance** across multiple PVT (Process, Voltage, Temperature) corners analysed using **OpenSTA** for the vsdbabysoc design.

---

## 📊 Timing Analysis Summary Table

| Library Corner     | Max/Worst Max Slack (Setup) | Min/Worst Min Slack (Hold) | WNS       | TNS         | Observation                  |
|--------------------|-----------------------------|-----------------------------|-----------|-------------|------------------------------|
| ff_n40C_1v95       | 4.0421                      | 0.1875                      | 0.0000    | 0.0000      | 🟡 <span style="color:#DAA520;">Hold timing marginal</span>       |
| ff_100C_1v65       | 2.4466                      | 0.2491                      | 0.0000    | 0.0000      | 🟡 <span style="color:#DAA520;">Hold timing marginal</span>       |
| ff_100C_1v95       | 3.8366                      | 0.1960                      | 0.0000    | 0.0000      | 🟡 <span style="color:#DAA520;">Hold timing marginal</span>       |
| ff_n40C_1v56       | 1.1270                      | 0.2915                      | 0.0000    | 0.0000      | 🟡 <span style="color:#DAA520;">Hold timing marginal</span>       |
| ff_n40C_1v65       | 2.1219                      | 0.2551                      | 0.0000    | 0.0000      | 🟡 <span style="color:#DAA520;">Hold timing marginal</span>       |
| ff_n40C_1v76       | 2.9919                      | 0.2243                      | 0.0000    | 0.0000      | 🟡 <span style="color:#DAA520;">Hold timing marginal</span>       |
| ss_100C_1v40       | -13.0402                    | 0.9053                      | -13.0402  | -7521.4248  | 🔴 <span style="color:red;">Major setup violations</span>         |
| ss_100C_1v60       | -6.2777                     | 0.6420                      | -6.2777   | -2909.8362  | 🔴 <span style="color:red;">Setup violations present</span>       |
| ss_n40C_1v28       | -52.9031                    | 1.8296                      | -52.9031  | -36775.8398 | 🔴 <span style="color:red;">Critical: Setup violations</span>     |
| ss_n40C_1v35       | -33.1984                    | 1.3475                      | -33.1984  | -23278.9902 | 🔴 <span style="color:red;">Critical: Setup violations</span>     |
| ss_n40C_1v40       | -24.6564                    | 1.1249                      | -24.6564  | -17170.5898 | 🔴 <span style="color:red;">Critical: Setup violations</span>     |
| ss_n40C_1v44       | -19.9610                    | 0.9909                      | -19.9610  | -13600.6846 | 🔴 <span style="color:red;">Major setup violations</span>         |
| ss_n40C_1v76       | -3.9606                     | 0.5038                      | -3.9606   | -1905.4320  | 🔴 <span style="color:red;">Setup violations present</span>       |
| ss_n40C_1v60       | -9.0172                     | 0.6628                      | -9.0172   | -5181.2949  | 🔴 <span style="color:red;">Setup violations present</span>       |
| tt_025C_1v80       | 1.1060                      | 0.3096                      | 0.0000    | 0.0000      | 🟢 <span style="color:green;">Timing met</span>                   |
| tt_100C_1v80       | 1.1452                      | 0.3145                      | 0.0000    | 0.0000      | 🟢 <span style="color:green;">Timing met</span>                   |

---

### Visual Legend: Symbol and Meaning 

- 🟢 Green = Timing met
- 🟡 Yellow = Marginal hold timing (positive slack, but close)
- 🔴 Red = Setup violations (negative slack)

## Description   

- | 🟢    | PASS                    | Slack ≥ 0 ns → Meets timing              |
- | 🔴    | FAILURE                 | Slack ≤ −1 ns → Fails timing             |

---

### Key Observations from our data:

1. 🟢 **FF (Fast-Fast)** and **TT (Typical-Typical)** corners meet setup & hold comfortably.  
2. 🟢 Hold slacks are positive across all corners — **no hold failures** observed.  
3. 🔴 **SS (Slow-Slow)** corners show significant setup violations due to **low voltage and high temperature**. 
4. 🟢 Classic trade-off observed:  
   - *Fast corners → Hold-critical* (short paths).  
   - *Slow corners → Setup-critical* (long paths).  
5. Worst setup violation at *ss_n40C_1v28* `the worst WNS at -52.9ns and TNS at -36,775ns`  → severe slowdown under cold/low-voltage.  
6. Indicates need for *path optimization, retiming, or clock relaxation* to close setup timing at slow corners.  
7. Hold timing (Min Slack) is generally met across all corners
---

### Heatmap 
In STA, a heatmap is used to visualize the timing slack of different paths under various conditions (PVT corners, clock domains, or corners vs cells). Each cell of the heatmap represents the slack of a particular path under a specific corner.

<img width="592" height="734" alt="image" src="https://github.com/user-attachments/assets/fb7bbc71-0568-41ca-8fa6-fc9b63984a0f" />


**Inference**
- Green → Worst slack (more negative or closer to violation)
- Light yellow → Best slack (more positive, safe)
---

1. **Worst Max Slack Across corners**

<img width="1454" height="904" alt="image" src="https://github.com/user-attachments/assets/a2ed89aa-1858-4371-9744-a484d4e3e977" />


**Observation**
- The Setup Slack values for most corners are positive, indicating timing is met for those corners.
- The worst corner (ss_n40C_1v28) has the maximum setup slack (~-51.21 ns), showing a significant timing violation under the slow-slow (SS) corner at low voltage and low temperature.

---

2. **Worst Min Slack Across Corners**

<img width="1457" height="904" alt="image" src="https://github.com/user-attachments/assets/0a13db38-8855-4a02-8896-24cc6b2ec234" />

**Observation**
- The Hold Slack values for most corners are positive, indicating no hold violations for those corners.
- The worst corner (ss_n40C_1v28) has the minimum hold slack (~1.83 ns), highlighted in dark red.

---

3. **Worst Negative Slack Across Corners**

<img width="1448" height="836" alt="image" src="https://github.com/user-attachments/assets/803051e3-e9a9-4707-9649-22931a3d2541" />



**Observation**
- Some corners have negative setup slack, indicating setup timing violations.
- The worst negative slack (WNS) occurs at ss_n40C_1v28 with -51.21 ns, highlighted in black.
---

4. **Total Negative Slack Across Corners**

<img width="1441" height="840" alt="image" src="https://github.com/user-attachments/assets/1281071e-a5e7-46b9-8805-6f894a76e87c" />


**Observation**
- Total Negative Slack aggregates all negative setup slacks.
- The highest TNS occurs at ss_n40C_1v28, indicating the largest cumulative timing violations.

---
## Inference from Multi PVT Corners slack analysis

- The analysis of hold slack, setup slack, and total negative slack across all process corners reveals that the worst-case timing occurs in the ss_n40C_1v28 corner, with significant setup violations.
- Hold timing is generally safe across corners. The total negative slack highlights cumulative violations, guiding optimization priorities. 
- Focus should be on critical paths in the slowest corners to achieve timing closure and reliable operation.


---
##  Conclusion

- Fast (FF) and Typical (TT) corners meet timing comfortably; setup and hold slacks are positive.
- Slow-Slow (SS) corners, especially ss_n40C_1v28, show severe setup violations, indicating paths are too slow under low voltage and low temperature.
- Hold timing is safe across all corners; no early data capture issues observed.
- Optimization is needed for critical paths in slow corners: retiming, path restructuring, or clock adjustment.
- Overall, the design is robust in typical and fast conditions, but slow corners require attention to ensure reliable operation across all PVT scenarios.
---
