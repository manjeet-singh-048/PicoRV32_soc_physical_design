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

## Synthesis

## Floorplanning
After starting an interactive mode for openlane, we need to do 
prep -design picorv32a -tag <synthesis run tag name>
run_floorplanning
