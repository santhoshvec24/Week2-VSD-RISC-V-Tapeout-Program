



## Cloning the VSDBabySoC Repository:
Before cloning the repository, choose a directory where you wants to store your works.
```bash
cd ~/vcd/photos
```

> [!Note]
> I prefer storing my work in a `photos` folder under `~/vcd`, but you can also clone it into 'home' directory.

Clone the required sources from the [VSDBabySoC Repository](https://github.com/manili/VSDBabySoC):

```bash
git clone https://github.com/manili/VSDBabySoC.git
cd VSDBabySoC
ls
```
`ls` used for to list out the files inside the `VSDBabySoC folder` and verify its contents.

<img width="735" height="348" alt="Screenshot from 2025-10-04 16-17-30" src="https://github.com/user-attachments/assets/1f505281-78ab-4034-b8b9-051e9e3c0391" />

---

### Analyse the contents of VSDBabySoC

After cloning the repository, it is important to understand the structure of the files and directories.
This helps in navigating the design, identifying modules, and preparing for simulation.

<img width="1182" height="714" alt="Screenshot from 2025-10-04 16-15-36" src="https://github.com/user-attachments/assets/cdf2cea2-99e4-4284-803a-42f3605009d7" />

---

### TLV to Verilog Conversion for RVMYTH

Initially, you will see only the rvmyth.tlv file inside src/module/, since the RVMYTH core is written in TL-Verilog.

To convert it into a .v file for simulation, follow the steps below:

#### TLV to Verilog Conversion Steps
```bash
# Step 1: Install python3-venv (if not already installed)
sudo apt update
sudo apt install python3-venv python3-pip

# Step 2: Create and activate a virtual environment
cd ~/vcd/photos/VSDBabySoC/
python3 -m venv sp_env
source sp_env/bin/activate

# Step 3: Install SandPiper-SaaS inside the virtual environment
pip install pyyaml click sandpiper-saas

# Step 4: Convert rvmyth.tlv to Verilog
sandpiper-saas -i ./src/module/*.tlv -o rvmyth.v --bestsv --noline -p verilog --outdir ./src/module/
```
<img width="1022" height="489" alt="Screenshot from 2025-10-04 16-34-36" src="https://github.com/user-attachments/assets/f472829b-bdf4-419b-87d6-3def6ab121e2" />

After running the above command, rvmyth.v will be generated in the src/module/ directory.

You can confirm this by listing the files:
```
cd ~/vcd/photos/VSDBabySoC
ls src/module
```
<img width="737" height="492" alt="Screenshot from 2025-10-04 16-38-18" src="https://github.com/user-attachments/assets/78cf0694-4a3d-43f3-a90d-0c9c3ebb41c4" />

#### Note
To use this environment in future sessions, always activate it first:
```bash
source sp_env/bin/activate
```
To deactivate
```bash
deactivate
```

--- 

## Simulation Steps
### Pre-Synthesis Simulation
Run the following command to perform a pre-synthesis simulation:
```bash
mkdir -p output/pre_synth_sim
iverilog -o ~/vcd/photos/VSDBabySoC/output/pre_synth_sim/pre_synth_sim.out -DPRE_SYNTH_SIM -I ~/vcd/photos/VSDBabySoC/src/include -I ~/vcd/photos/VSDBabySoC/src/module ~/vcd/photos/VSDBabySoC/src/module/testbench.v
```
then run,
```bash
cd output/pre_synth_sim
./pre_synth_sim.out
```

#### Viewing Waveform in GTKWave
After running the simulation, open the VCD file in GTKWave:
```bash
cd ~/vcd/photos/VSDBabySoC
gtkwave output/pre_synth_sim/pre_synth_sim.vcd
```
<img width="1648" height="837" alt="Screenshot from 2025-10-04 16-57-53" src="https://github.com/user-attachments/assets/17484371-b5aa-4552-b173-ff9314c83e57" />

---

### Verilog Source Files (*.v):
These files hold the RTL implementation of the BabySoC, defining its core modules and functional behavior.

`avsddac.v` – Implements the DAC (Digital-to-Analog Converter) module of the SoC.
```bash
module avsddac (
   OUT,
   D,
   VREFH,
   VREFL
);

   output      OUT;
   input [9:0] D;
   input       VREFH;
   input       VREFL;
   

   reg  real OUT;
   wire real VREFL;
   wire real VREFH;

   real NaN;
   wire EN;

   wire [10:0] Dext;	// unsigned extended

   assign Dext = {1'b0, D};
   assign EN = 1;

   initial begin
      NaN = 0.0 / 0.0;
      if (EN == 1'b0) begin
         OUT <= 0.0;
      end
      else if (VREFH == NaN) begin
         OUT <= NaN;
      end
      else if (VREFL == NaN) begin
         OUT <= NaN;
      end
      else if (EN == 1'b1) begin
         OUT <= VREFL + ($itor(Dext) / 1023.0) * (VREFH - VREFL);
      end
      else begin
         OUT <= NaN;
      end
   end

   always @(D or EN or VREFH or VREFL) begin
      if (EN == 1'b0) begin
         OUT <= 0.0;
      end
      else if (VREFH == NaN) begin
         OUT <= NaN;
      end
      else if (VREFL == NaN) begin
         OUT <= NaN;
      end
      else if (EN == 1'b1) begin
         OUT <= VREFL + ($itor(Dext) / 1023.0) * (VREFH - VREFL);
      end
      else begin
         OUT <= NaN;
      end
   end
endmodule
```
- Inputs:
        - D: A 10-bit digital input from the processor.
        - VREFH: Reference voltage for the DAC.
- Output:
        - OUT: Analog output signal.

## RTL Simulation of modules:

`avsddac.v`
```bash
gedit avsddac.v
gedit tb_avsddac.v
```

> Testbench ported from [rvmyth_avsddac_interface](https://github.com/vsdip/rvmyth_avsddac_interface/blob/main/iverilog/Pre-synthesis/avsddac_tb_test.v) repository

```bash
iverilog -o ~/vcd/photos/VSDBabySoC/src/module/avsddac.vvp ~/vcd/photos/VSDBabySoC/src/module/avsddac.v ~/vcd/photos/VSDBabySoC/src/module/tb_avsddac.v
vvp avsddac.vvp
gtkwave avsddac_tb_test.vcd
```
<img width="998" height="701" alt="Screenshot from 2025-10-04 17-37-26" src="https://github.com/user-attachments/assets/6b443cbf-ca7d-4904-b30b-bfb382c2ebf2" />

`avsdpll.v`

View the design file
```bash
module avsdpll (
   output reg  CLK,
   input  wire VCO_IN,
   input  wire ENb_CP,
   input  wire ENb_VCO,
   input  wire REF
);
   real period, lastedge, refpd;

   initial begin
      lastedge = 0.0;
      period = 25.0; // 25ns period = 40MHz
      CLK <= 0;
   end

  // Toggle clock at rate determined by period
   always @(CLK or ENb_VCO) begin
      if (ENb_VCO == 1'b1) begin
         #(period / 2.0);
         CLK <= (CLK === 1'b0);
      end
      else if (ENb_VCO == 1'b0) begin
         CLK <= 1'b0;
      end 
      else begin
         CLK <= 1'bx;
      end
   end
   
   // Update period on every reference rising edge
   always @(posedge REF) begin
      if (lastedge > 0.0) begin
         refpd = $realtime - lastedge;
         // Adjust period towards 1/8 the reference period
         //period = (0.99 * period) + (0.01 * (refpd / 8.0));
         period =  (refpd / 8.0) ;
      end
      lastedge = $realtime;
   end
endmodule
```
> Testbench ported from [rvmyth_avsdpll_interface](https://github.com/vsdip/rvmyth_avsdpll_interface/blob/main/verilog/pll_tb.v) repository

```bash
iverilog -o ~/Documents/Verilog/Labs/avsdpll.vvp ~/Documents/Verilog/Labs/VSDBabySoC/src/module/avsdpll.v ~/Documents/Verilog/Labs/tb_avsdpll.v
vvp avsddac.vvp
gtkwave avsddac_tb_test.vcd
```
<img width="998" height="704" alt="Screenshot from 2025-10-04 18-25-40" src="https://github.com/user-attachments/assets/0b47fb59-bff1-429f-94d2-873b74a923d4" />

---

## Post-synthesis Simulation of VSDBabySoc

### Synthesis :

Synthesis requires the header files essential for the rvmyth module,
these are
- sp_verilog.vh – includes core Verilog macros and parameter definitions
- sandpiper.vh – defines integration-specific settings used by SandPiper
- sandpiper_gen.vh – contains tool-generated parameters and configuration values

These files need to be present in the working directory of yosys in order to ensure error free synthesis. This is done using the following commands,

cd ~/Documents/Verilog/Labs/VSDBabySoC
cp -r src/include/sp_verilog.vh .
cp -r src/include/sandpiper.vh .
cp -r src/include/sandpiper_gen.vh .

Now inside the `../VSDBabySoC` folder, run yosys,
```bash
yosys
```
In yosys, run 
```bash
read_verilog src/module/vsdbabysoc.v 
read_verilog -I ~/vcd/photos/VSDBabySoC/src/include/ ~/vcd/photos/VSDBabySoC/src/module/rvmyth.v
read_verilog -I ~/vcd/photos/VSDBabySoC/src/include/ ~/vcd/photos/VSDBabySoC/src/module/clk_gate.v
```
This is performed to read the verilog files.
then, the library files
```bash
read_liberty -lib ~/vcd/photos/VSDBabySoC/src/lib/avsdpll.lib 
read_liberty -lib ~/vcd/photos/VSDBabySoC/src/lib/avsddac.lib 
read_liberty -lib ~/vcd/photos/VSDBabySoC/src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

Synthesize `vsdbabysoc`, specifying it as the top module,
```bash
synth -top vsdbabysoc
```

Convert D Flip-Flops into equivalent Standard Cell instances by,
```bash
dfflibmap -liberty ~/vcd/photos/VSDBabySoC/src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```
Perform Optimization and Technology mapping using the following commands,
```bash
opt
abc -liberty ~/vcd/photos/VSDBabySoC/src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib -script +strash;scorr;ifraig;retime;{D};strash;dch,-f;map,-M,1,{D}
```
Then, conduct final optimisations and clean-up through,
```bash
flatten
setundef -zero
clean -purge
rename -enumerate
```

- `flatten`: Remove hierarchy, make a flat netlist
- `setundef -zero`: Replace undefined signals with 0
- `clean -purge`: Delete unused/duplicate logic
- `rename -enumerate`: Systematically rename nets and cells

To check the statistics of the synthesised design run,
```bash
stat
```
Statistics:

<pre>


</pre>

Then finally write the netlist using the command, 
```bash
write_verilog -noattr ~/vcd/photos/vsdbabysoc_synth.v
```

