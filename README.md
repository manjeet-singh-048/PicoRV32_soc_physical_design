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
In this Workshop, We will be using the Picorv32a design for hands-on experience of various steps of Physical Design.

1) Invoking the OpenLane and Design preparation
docker command invokes openlane
''' ./flow.tcl -interactive
''' prep -design picorv32a

## Synthesis

## Floorplanning
After starting an interactive mode for openlane, we need to do 
prep -design picorv32a -tag <synthesis run tag name>
run_floorplanning
