 # **Note:** Opensta- Installation instructions:

### Step 1: Clone the Repository

```bash
git clone https://github.com/parallaxsw/OpenSTA.git
cd OpenSTA
```



#### Step 2: Build the Docker Image
```bash
docker build --file Dockerfile.ubuntu22.04 --tag opensta .
```
This builds a Docker image named opensta using the provided Ubuntu 22.04 Dockerfile. All dependencies are installed during this step.



#### Step 3: Run the OpenSTA Container
To run a docker container using the OpenSTA image, use the -v option to docker to mount direcories with data to use and -it to run interactively.
```bash
docker run -i -v $HOME:/data opensta
```
#### Breakdown:

Docker run: Runs a new container.
-i: Interactive mode with a pseudo-TTY (so you can interact with the shell).
-v $HOME:/data: Mounts your home directory into the container at /data.
opensta: The name of the Docker image you're trying to run (OpenSTA in this case).



You now have OpenSTA installed and running inside a Docker container. After successful installation, you will see the % prompt—this indicates that the OpenSTA interactive shell is ready for use.

### Timing Analysis Using Inline Commands

Once inside the OpenSTA shell (% prompt), you can perform a basic static timing analysis using the following inline commands:
```shell
# Instructs OpenSTA to read and load the Liberty file "nangate45_slow.lib.gz".
read_liberty /OpenSTA/examples/nangate45_slow.lib.gz

# Intructs OpenSTA to read and load the Verilog file (gate level verilog netlist) "example1.v"
read_verilog /OpenSTA/examples/example1.v

# Using "top," which stands for the main module, links the Verilog code with the Liberty timing cells.
link_design top

# Create a 10ns clock named 'clk' for clk1, clk2, and clk3 inputs 
create_clock -name clk -period 10 {clk1 clk2 clk3}

# Set 0ns input delay for inputs in1 and in2 relative to clock 'clk'
set_input_delay -clock clk 0 {in1 in2}

# Report of the timing checks for the design 
report_checks 
```
  
Note: We used report_checks here because only the slow liberty file (nangate45_slow.lib.gz) is loaded.
This represents a setup (max delay) corner, so the analysis focuses on setup timing by default.

### 🤔Why Does report_checks Show Only Max (Setup) Paths?
# Practical 
<details>
 <summary>Project Structure</summary>
 <details>
 # Set 0ns input delay for inputs in1 and in2 relative to clock 'clk'
set_input_delay -clock clk 0 {in1 in2}

# Report of the timing checks for the design 
report_checks 
```
  
Note: We used report_checks here because only the slow liberty file (nangate45_slow.lib.gz) is loaded.
This represents a setup (max delay) corner, so the analysis focuses on setup timing by default.

