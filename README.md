# VSD - Physical Design Workshop using Openlane

## Introduction to Openlane
**OpenLANE** is an open-source digital integrated circuit (IC) design flow that enables the design of custom digital circuits using open-source tools and technologies. It is a complete RTL-to-GDSII (register-transfer level to graphic design system II) design flow that can be used for designing digital circuits ranging from simple to complex. 

### OpenLane from Inside
OpenLANE includes several open-source tools and scripts that automate the digital IC design process, including synthesis, placement and routing, static timing analysis, and power analysis. The open-source tools used in OpenLANE include the **OpenROAD project, Yosys for synthesis, ABC for logic optimization, Magic, Netgen, Klayout etc**.

### **Openlane detailed ASIC flow**
![detailed ASIC flow](https://user-images.githubusercontent.com/125300415/224268894-44bac0f7-2962-4915-bb65-10a25d2ea8e6.png)

### Open-source eda tools in OpenLane
OpenLane flow integrated several key open source tools over the execution stages
```
    Synthesis
        yosys - Performs RTL synthesis
        abc - Performs technology mapping
        OpenSTA - Performs static timing analysis on the resulting netlist to generate timing reports
    Floorplan and PDN
        init_fp - Defines the core area for the macro as well as the rows (used for placement) and the tracks (used for routing)
        ioplacer - Places the macro input and output ports
        pdn - Generates the power distribution network
        tapcell - Inserts welltap and decap cells in the floorplan
    Placement
        RePLace - Performs global placement
        Resizer - Performs optional optimizations on the design
        OpenDP - Perfroms detailed placement to legalize the globally placed components
    CTS
        TritonCTS - Synthesizes the clock distribution network (the clock tree)
    Routing
        FastRoute - Performs global routing to generate a guide file for the detailed router
        CU-GR - Another option for performing global routing.
        TritonRoute - Performs detailed routing
        SPEF-Extractor - Performs SPEF extraction
    GDSII Generation
        Magic - Streams out the final GDSII layout file from the routed def
        Klayout - Streams out the final GDSII layout file from the routed def as a back-up
    Checks
        Magic - Performs DRC Checks & Antenna Checks
        Klayout - Performs DRC Checks
        Netgen - Performs LVS Checks
        CVC - Performs Circuit Validity Checks
```

## ASIC Design using OpenLane
Installation, documentation and architecture of OpenLane can be found in this [OpenLane Github](https://github.com/The-OpenROAD-Project/OpenLane) link
### Design Preparation
Let's understand the directory structure inside **<openlane_work_dir>/openlane/designs/** <br>
This area has various type of designs already setup. We will do our analysis on a **Picorv32a** design which is a processor core with RISC V Instruction set architecture(ISA).
In this Workshop, We will be using the Picorv32a design for hands-on experience of the various steps of Physical Design.

### 1. Invoking the OpenLane and Design preparation
  - ``` ./flow.tcl -interactive     ``` #Interactive mode allows us to run the various stages sequentially and also allows to set variables as & when required. We can view results & reports after each stage.
  - ``` package require openlane 0.9``` #Loading openlane packages  
  - ``` prep -design <design_name>  ``` #Initiates Design preparation, creates merged LEF and also reads the config.tcl for tool configuration.
     ![prep design](https://user-images.githubusercontent.com/125300415/224514941-9a1c8ebc-000a-4f07-9ce5-cc44baaa8a22.png)

     - We will be using Picorv32a design. All the related config files can be found in **openlane/designs/\<design_name\>/**
     - Outputs of prep design stage
     ![ls tag](https://user-images.githubusercontent.com/125300415/224515117-4bff7cc4-d71f-456b-96f5-806b05f816fd.png)


  - All the subsequent steps done like synthesis, floorplanning, Powerplanning etc will be get stored in a dir under **/designs/picorv32a/runs/\<tag\>/results/**


### 2. Synthesis
  - ```run_synthesis``` #Command used for running synthesis
  - This step sythesizes a gate level netlist from the verilog input file, does the technology mapping and also does static timing analysis on the synthesized netlist using OpenSTA.
    - Synthesized Gate netlist is dumped as below file **/picorv32a/runs/\<tag\>/results/synthesis/picorv32a.synthesis.v**
    - Reports directory has the reports of the synthesis step plus the timing reports
    ![synthesis reports](https://user-images.githubusercontent.com/125300415/225746952-f47f45cb-7c69-485a-8624-980ad28ad42e.png)
    - Flop Ratio can be found from synthesis report [1-yosys_4.stats.rpt]
       ![yosys 4 rpt](https://user-images.githubusercontent.com/125300415/225918255-b9452a56-4f80-42a1-877d-cbdb3d1b54c0.png)

       ![image](https://user-images.githubusercontent.com/125300415/225917677-7d09443e-a7d5-468e-9da4-748c65b71ded.png)
       - No. of DFF : 1613
       - No. of Cells : 14876
       - Flop Ratio : 1613/14876 = 10.8%


### 3. Floorplanning
  - **Jumpstarting from a previous step**. We follow the following steps
     - start an interactive mode for openlane.
     - ```prep -design picorv32a -tag <synthesis run tag name>```

  - ```run_floorplan``` #During floorplan we define rules for floorplanning through config.tcl. The precedence order for various level config files are 
**PDK.tcl \> /designs/picorv32a/config.tcl \> /openlane/configurations/floorplan.tcl**
     - Floorplanning also adds endcap cells and does tap cell insertion.
     - ```run_floorplan``` calls the following atomic steps under the hood. ```init_floorplan, place_io, global_placement_or, detailed_placement, tap_decap_or```
     - play with env variables like **FP_IO_MODE** and check the dumped def
  - Outputs
     - ```runs/\<tag\>/results/floorplan/picorv32a.def```
     ![floorplan results   png](https://user-images.githubusercontent.com/125300415/225748837-608c1fb5-b0d5-4a65-b5ec-46475a5de926.jpg)

     - ```runs/\<tag\>/reports/floorplan/ ```  #reports the core & die area 
     ![reports floorplan](https://user-images.githubusercontent.com/125300415/225749032-72d28130-c4a0-48c0-82b6-a03621344506.jpg)

     
     
     - **Invoke Magic to see the DEF dumped in the floorplan stage**
        - ```magic -T <Pdk Tech file> lef read <path to merged lef> def read <path to DEF file> ```
        - tech_file = sky130.tech, def = from floorplan outputs
     ![Magic def](https://user-images.githubusercontent.com/125300415/224516013-74afca8d-b4b8-43de-988c-14ee75ddfa31.png)

Core Utilization
  - Area occupied by cells from synthesis report [1-yosys_4.stats.rpt] : 147,712.9184 um2
  - Core area form floor plan stage report : 420,473.2672 um2
  - Core utilization = 35.13% [as specified in PDK specific config.tcl]
  ![pdk core utilization](https://user-images.githubusercontent.com/125300415/225919204-0cf43bf2-2cf6-4d6e-aa14-b59f231693c1.png)


### 4. Placement

  - ```run_placement``` #Congestion aware placement happens in two steps internally
     - **Global Placement** - uses HPWL(half parameter wire length) method to ensure reduced wire length.
     - **Detailed Placement** - performs placement of standard cells in rows, taking care of abuttment etc
  - Outputs
     - ```runs/<tag>/results/placement/picorv32a.placement.def```
     ![results - placement](https://user-images.githubusercontent.com/125300415/225749364-68ae463c-330d-4953-a0f6-a16354412fda.jpg)


  - Checking DEF after placement in Magic
     - ```magic -T <Tech file of PDK> lef read <path to merged lef> def read <path to DEF file post placement>```
     - ![magic DEF](https://user-images.githubusercontent.com/125300415/224515525-288444cc-4c84-49ba-958f-1e20bb67c79a.png)



### 5. Clock Tree Synthesis (CTS)
  - ```run_cts``` #Synthesizes clock tree, adds buffers and dumps a new verilog and def file. Buffer placement is congestion aware Various Clock Buffers are taken from the buffer cells defined through 
  CTS_CLK_BUFFER_LIST
     - runs/\<tag\>/results/synthesis/picorv32.sythesis.cts.v
     ![results - cts](https://user-images.githubusercontent.com/125300415/225749713-91d23cd8-354a-4885-bca1-a2a6c0bd24eb.jpg)



### 6. Routing
  - ```run_routing``` 
  - global/fast routing : creates routing guides (using FastRoute)
  - detailed routing : Algorithm to pick best route from routing guides and routes metal geometries in preferred direction. (Using Triton Route)
  - runs/\<tag\>/results/routing/
  ![results - routing](https://user-images.githubusercontent.com/125300415/225750160-c2172648-0b85-46d0-997f-a4bee3a8c7ad.jpg)




## PART 2. Standard Cell Design Flow
Designing a Standard cell is done in 3 parts:
  -  Inputs - PDKs, DRC & LVS rules, Spice models, library and user-defined specs (Height,Width etc)
  -  Design steps - Circuit Design, Layout Design & Characterization using GUNA software. Characterization is done for Noise, Power & Timing.
  -  Outputs - CDL file (circuit desciption language file), GDS II, extracted spice netlist(.cir), timing, noise, power.libs functions

### Characterization flow
The GUNA software is used for characterizing a standard cell. Cells of different strength & functionality are characterized for different PVT corners using following steps:

  1. Read the models and tech file
  2. Read extracted spice netlist 
  3. Read Subckt
  4. Attach power sources
  5. Apply input or stimulus
  6. Apply suitable load Cap
  7. Pass necesary simulation commands

### Designing a Custom CMOS Inverter cell
Creation of single height standard cell and plug this custom cell into a more complex design and perform it's PnR in the openlane flow. The standard cell chosen is a basic CMOS inverter and it will be plugged into the picorv32a core. More details in Nickson's [Github link](https://github.com/nickson-jose/vsdstdcelldesign)

#### Steps to extract spice netlist from layout (in Magic)
  - ```extract all```
  - ```ext2spice cthresh 0 rthresh 0``` #Extracting parasitic R & C
  - ```ext2spice``` # Generates a spice netlist
  
#### Doing Spice simulations on prepared cell
  - Clone the vsdstdcelldesign repo into <work_dir>/openlane/
   ![git clone vsdstd cell](https://user-images.githubusercontent.com/125300415/225768932-bfcc2fd2-f4f6-4b24-8932-2ed0f707635d.jpg)
   ![vsdstdcelldesign](https://user-images.githubusercontent.com/125300415/225751233-b5a168fe-2850-44a8-a1c2-24772d79b9c8.png)

  - Open the .mag Inverter Layout file in Magic
  ![image](https://user-images.githubusercontent.com/125300415/224517262-b4384d19-3b0c-47c6-99dd-100a0c98fa71.png)

  - Extracted Spice netlist from Layout
    ![image](https://user-images.githubusercontent.com/125300415/224516734-05398e07-0728-4138-88fa-ce5e6f24a134.png)
    ![image](https://user-images.githubusercontent.com/125300415/224517727-c5e06213-0ada-4b4f-ab7d-44b82e3ba086.png)



#### SPICE DECK creation & SPICE simulations.
We will create a SPICE deck for the inverter model. SPICE deck will have following things : 
   - ```Model description, Netlist description, Component definitions and Values, Simulation commands & model include statements```
   ![Sky130_inv.spice](https://user-images.githubusercontent.com/125300415/224516776-95c499b2-189e-4626-9817-052754697b7b.png)
   - ```ngspice sky130_inv.spice``` Open Inverter spice file with ngspice
   - for plotting the waveform use ``` plot y vs time a ``` (y -> output, a-> Input)
    ![waveform](https://user-images.githubusercontent.com/125300415/224517481-2535e04b-229e-47c0-888f-05541ac75079.png)

#### LEF generation of a Standard Cell
Look for the tracks.info file in the pdks directory, tracks defined here will be used for routing.
Next, we enable grid in magic using command ```grid [xspacing [yspacing [xorigin yorigin]]]``` in current scenario ```grid 0.46um 0.34um 0.23um 0.17um```
![grid track info](https://user-images.githubusercontent.com/125300415/225921569-4a9783e2-8df9-4f6c-bd4f-ee7ae299f599.png)
Width of standard cell should be an odd multiple of x-pitch and height should be an odd multiple of y-pitch. Signal ports should be at the intersection of horizonatal and vertical tracks. We can see that Width of our cell is 3 times x-pitch in above image. More details of LEF creation can be reffered in ![Nickson's Github link](https://github.com/nickson-jose/vsdstdcelldesign).


## Part 3-Plugging the VSD Inverter cell into the openlane flow
  1. Copy the LEF & .lib files of the custom inverter into ```designs/picorv32a/src``` directory
  2. Edit the ```designs/picorv32a/config.tcl``` and set the env variables like ```LIB_SYNTH_TYPICAL```, ```LIB_FASTEST```, ```LIB_SLOWEST``` etc & run synthesis again.
  
### Synthesis
Before delay-area optimization following is the delay stats for the design.
![image](https://user-images.githubusercontent.com/125300415/224517828-5a6ca6c4-3329-40e2-ab39-0479e12dc0ae.png)

Optimize between delay & area using the following env variables( more details in ```openlane/configuration/README.md```) <br>

```% set ::env(SYNTH_STRATEGY) "DELAY 1"``` <br>
```% set ::env(SYNTH_BUFFERING) 1``` <br>
```% set ::env(SYNTH_SIZING) 1``` <br>


### Floorplanning
```run_floorplan``` fails after plugging in inverter LEF
![image](https://user-images.githubusercontent.com/125300415/224517978-e80f4923-e515-4386-9947-1c75bccf627a.png)

We will do these atomic steps to achieve the same

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
  - create sta.conf file
  ![image](https://user-images.githubusercontent.com/125300415/224518290-82c01f9f-650d-416f-8027-f51af1e619e2.png)

  - create SDC file for STA analysis in ```/designs/picorv32a/src/my.sdc``` <br>
  ![image](https://user-images.githubusercontent.com/125300415/225758010-a8c2bd1b-8e51-4d28-bbb0-746d03385b13.png)
  - running with the default sdc files in design. we get very high delays. so firstly set the max fanout using ```set ::env(SYNTH_MAX_FANOUT) 4``` and rerun synthesis and we will run STA again to verify the effect.

![report tns wns](https://user-images.githubusercontent.com/125300415/225759071-b45ee6db-ff6a-49fe-8f1f-d444ba905508.jpg)

Picking up cells with more fanout and less strength and then replacing them using ```replace_cell <instance_name> <lib_cellname>```. **Target is to get wns \< 1ns**. Starting from the starting few cells in clk path, and replacing with higher drive strength cells.
```report_net -connections <net_id>``` : to check the fanout details <br>
![report_cehcks   replace cell](https://user-images.githubusercontent.com/125300415/225759623-e2e781c1-6e85-44b4-b802-1e7ae07636ea.jpg)

We will dump out the modified netlist now using below command.
```write_verilog <tag>/results/synthesis/<design>.synthesis.v``` <br>


Place and Route is an iterative process, we fix some things, optimize and do the stages again. Now, we have a netlist with wns below 1ns. we will go ahead with floorplanning.

  - ```run_floorplan``` : Since we written verilog into previous synthesis results. Improved netlist will be taken <br>
  - we will use atomic steps for floorplan as described above in floorplan_stage followed by ```detailed_placement```
  ![lef reading after placement](https://user-images.githubusercontent.com/125300415/225770095-49dc0b06-d4ab-40b7-8ab4-b57a7f059561.jpg)

  - Checking the custom Inv cell in updated DEF.
   ![DEF view expand](https://user-images.githubusercontent.com/125300415/225769047-ab7b494d-28f4-4fc2-baf8-21d106827e64.jpg) 
   ![expand on our vsd inv cell](https://user-images.githubusercontent.com/125300415/225767544-e2429f9c-7b62-4904-bc58-92876094389e.jpg)



  
  - ```run_cts``` : after CTS stage, a new verilog file with cts data in results/synthesis folder is generated and cts.def is generated in results/cts folder 
    - We can check input values in ```openlane/scripts/openroad/or_cts.tcl```. Check max_cap, max_slew, Clock_buffer list etc <br>
  ![CTS succesful](https://user-images.githubusercontent.com/125300415/225765374-fe8044d9-2572-496b-b6ea-cda0ea8274ce.jpg)


  
  - **OpenRoad** : We can also do Timing Analysis by invoking openroad while running openlane. This way we can simply use to env variables.
    - Run these steps to setup a db first <br>
```
read_lef /openLANE_flow/designs/picorv32a/runs/24-02_13-58/tmp/merged.lef
read_def /openLANE_flow/designs/picorv32a/runs/24-02_13-58/results/cts/picorv32a.cts.def
write_db pico_cts.db
read_db pico_cts.db
read_verilog /openLANE_flow/designs/picorv32a/runs/24-02_13-58/results/synthesis/picorv32a.synthesis_cts.v
read_liberty -max $::env(LIB_SLOWEST)
read_liberty -min $::env(LIB_FASTEST)
read_sdc /openLANE_flow/designs/picorv32a/src/my_base.sdc
set_propagated_clock [all_clocks]
report_checks -path_delay min_max -format full_clock_expanded -digits 4
```

  - verify the **min_path** & **max_path** timing condition is met after setting the correct library here using $::env(LIB_SYNTH_COMPLETE) since we used typical library for synthesis.
  ![image](https://user-images.githubusercontent.com/125300415/225774279-35c809f8-a5c0-4090-bed8-0d08d3c0e205.png)

**Small exercise:** To remove smallest strenth clk buffer from cts_clk_buffer_list
  - ``` echo $::env(CTS_CLK_BUFFER_LIST)``` and set using ```lreplace``` as shown.

    ![cts clk buf edit](https://user-images.githubusercontent.com/125300415/225773242-133a777c-5238-4a23-bdd4-461bd8ef32eb.png)
    ![run_cts](https://user-images.githubusercontent.com/125300415/225773484-0f52bcff-1884-4b43-b501-95184fde9984.png)
  - set the correct ```CURRENT_DEF``` before ```run_cts``` to avoid error
  - Check the clock skew as shown below
  ![image](https://user-images.githubusercontent.com/125300415/225773642-089ff06d-f19c-4946-a6e7-c13cba36bfc0.png)
  
## Day 5- Final Steps for RTL2GDS using tritonRoute and OpenSTA

### Generating Power Distribution Network(PDN)
  ![PDN](https://user-images.githubusercontent.com/125300415/236278070-e7cb444b-0100-46df-991f-6397315bfbdc.png) <br>

Power Distribution Network is created in order to provide access to the Power and GND domain for all the cells in the chip. This is achieved by creating supply rings, straps & rails. The various VDD/VSS domains are taken from Power PADS to Rings and it is tapped from these rings using metal straps & finally reaches the cells through power rails. To generate pdn we follow the below steps:

  - ```gen_pdn``` : generates PG network as per the pitch & width values for track.info file. In tmp/floorplan folder pdn.def is generated containing PG rounting + CTS def.
  ![gen_pdn](https://user-images.githubusercontent.com/125300415/225764671-ca63cae5-06cb-4c76-a037-2941cb8f6d74.png)
  ![PDN generation](https://user-images.githubusercontent.com/125300415/225774507-5fbbc565-34b4-4d6f-b617-50ab39872dc5.jpg)
  
### Design Rule Checks(DRC)
Before Routing lets look at the design rule checks. During routing we need to take care of the Design rule checks as well. Some common DRC rules are as follows:
  ![DRC rules](https://user-images.githubusercontent.com/125300415/236391691-4092843e-bdce-4043-a161-f65b35e8332b.png)
  - Minimum Wire Width
  - Optimal Wire Pitch 
  - Minimum Wire Spacing
  - Via Width
  - Via Spacing
 
### Routing 
Routing is a two steps process where firstly it creates a global or fast route & then it does detailed route. Below Image is self explanatory of these two steps.
  ![Routing](https://user-images.githubusercontent.com/125300415/236282128-7e426be8-5b0b-44a0-b001-c159fc8e53ba.png) <br>
  
  1. Fast Route - It is coarse global route in which the whole die is divided into smaller blocks and very basic routing is performed. Output of Fast Route is routing guide and is basically fed to Detail route. It only creates grid cells and tiles in which the TritonRoute will use its algorithm for having the best routing with least twists and turns.
         ![Routing guide](https://user-images.githubusercontent.com/125300415/236283417-831f6d9b-a32d-4836-85e4-dbabd35dbe7f.png)
  
  2. Detail Route- TritonRoute does the detailed routing based on the pre processed routing guides from FastRoute and satisfies the inter-guide connectivity. 

The Routing is done using  MILP (mixed integer linear programming) based panel routing scheme with Intra-Layer parallel and inter-layer sequential routing framework. This means that first M1 routing will happen and after that it will go to M2. But in same layer, routing always happens parallel. Steps for running the Routing is as below:

  - ```run_routing``` (a) global route(FastRoute) creates routing guides, (b) detail route (TritonRoute) algorithm picks best possible route and places geometries.
  
  ![Routing](https://user-images.githubusercontent.com/125300415/225765268-8f798182-2b78-471f-8dd4-1fa73a18211f.png)
    - Outputs: ```<tags>/results/routing/```
     ![image](https://user-images.githubusercontent.com/125300415/225931181-cc51f21b-8838-46e1-a6fd-06bad06f2de8.png)
    - ```ls <tags>/tmp/routing/```
    ![tmp/routing/](https://user-images.githubusercontent.com/125300415/225929110-5dfccadb-8949-44e0-abc8-0fd861b8eb15.png)

Now, hook up the **final DEF** file '''/openLANE_flow/designs/picorv32a/runs/<tag date>/results/routing/picorv32a.def''' with the tech file and the LEF to get the final layout using **magic**
     ![image](https://user-images.githubusercontent.com/125300415/236287111-4f7d8587-91aa-4542-8ee2-4e8eac9a09c9.png)

### SPEF Extraction
SPEF stands for **Standard Parasitic Exchange Format** which is used to represent the parasitic like Resistance & Capacitance of the route geometries. OpenLANE has the tool known as SPEF_EXTRACTOR, which is basically a parser which takes the LEF & DEF files as inputs and generates the SPEF file as output. This step is executed while executing '''run_routing'''
  - SPEF Generated is shown below.
   ![image](https://user-images.githubusercontent.com/125300415/225932414-9cb7730f-d816-4a99-81e9-34d119a74168.png)
 

## PART 3. 16 Mask CMOS process steps

The 16-mask CMOS process is a common manufacturing technique used in the production of modern integrated circuits (ICs) and microprocessors. This process involves a series of 16 distinct steps, each of which contributes to the creation of a functional and reliable CMOS device. These steps include wafer preparation, deposition of various layers of material, photolithography to pattern these layers, etching to remove unwanted material, and finally, implantation of dopants to create the desired electrical properties. The end result is a complex circuit that can perform a wide range of tasks with high efficiency and precision. Understanding these steps is essential for designing and optimizing CMOS devices for a variety of applications, from consumer electronics to high-performance computing. Following are the Key Steps of this process: 
  1. Selecting a substrate - generally P-type.
  2. Creating active region for transistors.
  3. N-well & P-well formation.
  4. Formation of 'Gate'terminal
  5. LDD(Lightly Doped Drain) formation
  6. Source & Drain formation
  7. Steps to form contacts and local interconnects
  8. Higher level metal formation.

![Screenshot (369)](https://user-images.githubusercontent.com/125300415/225776569-de909eb7-d388-4e04-b485-ff4c9b5f87eb.png)
  

## ACKNOWLEDGEMENT
  - ![Kunal Ghosh](https://github.com/kunalg123/) - Co-founder of VLSI System Design (VSD)
  - ![Nickson Jose](https://github.com/nickson-jose) - Physical Design & Lab expert, VLSI System Design (VSD) <br>

Thank you for the enriching learning experience through this workshop. It was a good combination of theory plus Labs and I am looking forward to apply the knowledge gained here in interesting projects.


   

