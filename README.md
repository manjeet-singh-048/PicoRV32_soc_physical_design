# VSD - Physical Design Workshop using Openlane

## Introduction to Openlane
**OpenLANE** is an open-source digital integrated circuit (IC) design flow that enables the design of custom digital circuits using open-source tools and technologies. It is a complete RTL-to-GDSII (register-transfer level to graphic design system II) design flow that can be used for designing digital circuits ranging from simple to complex. 

### OpenLane from Inside
OpenLANE includes several open-source tools and scripts that automate the digital IC design process, including synthesis, placement and routing, static timing analysis, and power analysis. The open-source tools used in OpenLANE include the **OpenROAD project, Yosys for synthesis, ABC for logic optimization**.

### **Openlane detailed ASIC flow**
![image](https://user-images.githubusercontent.com/125300415/224268894-44bac0f7-2962-4915-bb65-10a25d2ea8e6.png)


## ASIC Design using OpenLANE
Installation, documentation and architecture of OpenLane can be found in this [OpenLane Github](https://github.com/The-OpenROAD-Project/OpenLane) link
### Design Preparation
The First step is to understand the directory structure inside <openlane_working_directory>/openlane/designs/
This area has various type of designs already setup. We will do our analysis on a Picorv32a design which is a processor core with RISC V Instruction set architecture(ISA).
In this Workshop, We will be using the Picorv32a design for hands-on experience of the various steps of Physical Design.

### 1. Invoking the OpenLane and Design preparation
  - Use docker command to invoke openlane
  - ``` ./flow.tcl -interactive     ``` #Interactive mode allows us to run the various stages sequentially and also allows to set variables as & when required 
  - ``` package require openlane 0.9``` #Loading openlane packages  
  - ``` prep -design <design_name>  ``` #Initiates Design preparation and also reads the config.tcl for tool configuration.
     ![prep design](https://user-images.githubusercontent.com/125300415/224514941-9a1c8ebc-000a-4f07-9ce5-cc44baaa8a22.png)

     -We will be using Picorv32a design. All the related config files can be found in openlane/designs/<design_name>/
     -Outputs of prep design stage
     ![image](https://user-images.githubusercontent.com/125300415/224515117-4bff7cc4-d71f-456b-96f5-806b05f816fd.png)


  - All the subsequent steps done like synthesis, floorplanning, Powerplanning etc will be get stored in a dir under **/designs/picorv32a/runs/_tag_/results/**


### 2. Synthesis
  - ```run_synthesis``` #Command used for running synthesis
  - This step sythesizes a gate level netlist from the verilog input file, does the technology mapping and also does static timing analysis on the synthesized netlist using OpenSTA.
    - Synthesized Gate netlist is dumped under **/picorv32a/runs/<tag/results/synthesis/picorv32a.synthesis.v**
    - Reports directory has the reports of the synthesis step plus the timing reports
  ![image](https://user-images.githubusercontent.com/125300415/224515393-4c1006b5-110d-4edf-82e2-b8eb3a5c67ee.png)


### 3. Floorplanning
  - For jumpstarting from a previous step. We follow the following steps
     - start an interactive mode for openlane.
     - ```prep -design picorv32a -tag <synthesis run tag name>```

  - ```run_floorplan``` #During floorplan we define rules for floorplanning through config.tcl. The precedence order for various level config files are PDK.tcl >>/designs/picorv32a/config.tcl>>/openlane/configurations/floorplan.tcl(sys_defaults).
     - Floorplanning also adds endcap cells and does tap cell insertion.
     - ```run_floorplan``` calls the following atomic steps under the hood. ```init_floorplan, place_io, global_placement_or, detailed_placement, tap_decap_or```
     - play with env variables like **FP_IO_MODE** and check the dumped def
  - Outputs
     - runs/<tag>/results/floorplan/picorv32a.def
     - runs/<tag>/reports/floorplan/   -reports the core & die area 
     ![floorplan results   png](https://user-images.githubusercontent.com/125300415/224515593-e1389a80-0af7-4ca6-b3d5-46ff628de9a8.jpg)
     ![image](https://user-images.githubusercontent.com/125300415/224516131-dfda7a48-023c-437b-b4c6-29747a1752f7.png)


  - Invoke Magic to see the DEF dumped in the floorplan stage
     - ```magic -T <Tech file of PDK> lef read <path to merged lef> def read <path to DEF file>```
     - tech_file - sky130.tech, def - from floorplan outputs
     ![image](https://user-images.githubusercontent.com/125300415/224516013-74afca8d-b4b8-43de-988c-14ee75ddfa31.png)

### 4. Placement

  - ```run_placement``` #Congestion aware placement happens in two steps internally
     - **Global Placement** - uses HPWL(half parameter wire length) method to ensure reduced wire length.
     - **Detailed Placement** - performs placement of standard cells in rows, taking care of abuttment etc
  - Outputs
     - runs/<tag>/results/placement/picorv32a.placement.def
     ![image](https://user-images.githubusercontent.com/125300415/224518338-b33082a0-6eff-4b66-bd04-78619ed1a6f2.png)


  - Checking DEF after placement in Magic
     - ```magic -T <Tech file of PDK> lef read <path to merged lef> def read <path to DEF file post placement>```
     - ![image](https://user-images.githubusercontent.com/125300415/224515525-288444cc-4c84-49ba-958f-1e20bb67c79a.png)



### 5. Clock Tree Synthesis (CTS)
  - ```run_cts``` #Synthesizes clock tree, adds buffers and dumps a new verilog and def file. Buffer placement is congestion aware Various Clock Buffers are taken from the buffer cells defined through 
  CTS_CLK_BUFFER_LIST
     - runs/<tag>/results/synthesis/picorv32.sythesis.cts.v

### 6. Routing
  - ```run_routing``` #Performs (a) global/fast routing : creates routing guides (b) detailed routing : Algorithm to pick best route from routing guides and routes metal geometries in preferred direction.



## PART 2. Standard Cell Design Flow
Designing a Standard cell is done in 3 parts:
  - 1. Inputs - PDKs, DRC & LVS rules, Spice models, library and user-defined specs (Height,Width etc)
  - 2. Design steps - Circuit Design, Layout Design & Characterization using GUNA software. Characterization is done for Noise, Power & Timing.
  - 3. Outputs - CDL file (circuit desciption language file), GDS II, extracted spice netlist(.cir), timing, noise, power.libs functions

### Characterization flow
The GUNA software is used for characterizing a standard cell. Cells of different strength & functionality are characterized for different PVT corners using following steps:

  1. Read the models and tech file
  2. Read extracted spice netlist 
  3. Read Subckt
  4. Attach power sources
  5. Apply input or stimulus
  6. Apply suitable load Cap
  7. Pass necesary simulation commands

### 16 Mask CMOS process steps
  1. Selecting a substrate - generally P-type.
  2. Creating active region for transistors.
  3. N-well & P-well formation.
  4. Formation of 'Gate'terminal
  5. LDD(Lightly Doped Drain) formation
  6. Source & Drain formation
  7. Steps to form contacts and local interconnects
  8. Higher level metal formation.

### Designing a Custom CMOS Inverter cell
Creation of single height standard cell and plug this custom cell into a more complex design and perform it's PnR in the openlane flow. The standard cell chosen is a basic CMOS inverter and it will be plugged into the picorv32a core. More details in this [Github link](https://github.com/nickson-jose/vsdstdcelldesign)

#### Steps to extract spice netlist from layout (in Magic)
  - ```extract all```
  - ```ext2spice cthresh 0 rthresh 0``` #Extracting parasitic R & C
  - ```ext2spice``` # Generates a spice netlist
  
#### Doing Spice simulations on prepared cell
  - Clone the vsdstdcelldesign repo into <work_dir>/openlane/
  - Open the .mag Inverter Layout file in Magic
  ![image](https://user-images.githubusercontent.com/125300415/224517262-b4384d19-3b0c-47c6-99dd-100a0c98fa71.png)

  - Extracted Spice netlist from Layout
    ![image](https://user-images.githubusercontent.com/125300415/224516734-05398e07-0728-4138-88fa-ce5e6f24a134.png)
    ![image](https://user-images.githubusercontent.com/125300415/224517727-c5e06213-0ada-4b4f-ab7d-44b82e3ba086.png)



#### SPICE DECK creation & spice simulations.
We will create a SPICE deck for the inverter model. SPICE DECK will have
   - ```Model description, Netlist description, Component definitions and Values, Simulation commands & model include statements```
   ![Sky130_inv.spice](https://user-images.githubusercontent.com/125300415/224516776-95c499b2-189e-4626-9817-052754697b7b.png)
   - Spice Simulation ```ngspice sky130_inv.spice```
   - for plotting the waveform use ``` plot y vs time a ``` (y -> output, a-> Input)
    ![waveform](https://user-images.githubusercontent.com/125300415/224517481-2535e04b-229e-47c0-888f-05541ac75079.png)


### Plugging the VSD Inverter cell into openlane flow
  1. Copy the LEF & .lib files of the custom inverter into ```designs/picorv32a/src``` directory
  2. Edit the ```designs/picorv32a/config.tcl``` and set the env variables like LIB_SYNTH_TYPICAL, LIB_FASTEST, LIB_SLOWEST etc & run synthesis again.
  
#### Synthesis
Before delay-area optimization following is the delay stats for the design.
![image](https://user-images.githubusercontent.com/125300415/224517828-5a6ca6c4-3329-40e2-ab39-0479e12dc0ae.png)

Optimize between delay & area using the following env variables( details in ```openlane/configuration/README.md```)
```% set ::env(SYNTH_STRATEGY) "DELAY 1"```
```% set ::env(SYNTH_BUFFERING) 1```
```% set ::env(SYNTH_SIZING) 1```


#### Floorplanning
```run_floorplan``` fails after plugging in inverter LEF
![image](https://user-images.githubusercontent.com/125300415/224517978-e80f4923-e515-4386-9947-1c75bccf627a.png)

We will use atomic steps to achieve the same

```
Floor plan stage:
   init_floorplan
   place_io
   global_placement_or
   tap_decap_or
   
Placement:
   detailed_placement

PD network generation :
   gen_pdn
Routing :
   run_routing 
```
  - Checking the Inv cell in Magic  
  ![image](https://user-images.githubusercontent.com/125300415/224518067-010c7be7-9389-486f-a410-28da05aa16b7.png)

### Optimizing synthesis to reduce setup Violations
  1. sta.conf file
  ![image](https://user-images.githubusercontent.com/125300415/224518290-82c01f9f-650d-416f-8027-f51af1e619e2.png)

  2. my_base.sdc file



   

