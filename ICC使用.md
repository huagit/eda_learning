[toc]

# Preface

- ICC学习资料

> 链接：https://pan.baidu.com/s/1ZMORqI1NRdQcpKPf6oderQ 
> 提取码：hy1a

- 里面包括:ICC Lab压缩包（在linux中解压）、ICC Lab Guide、ICC Student Guide、两篇我的博客笔记

<img src="https://i.loli.net/2020/09/17/UmOwsYFSRNhrpe8.png" style="zoom:50%;" />

- 本文根据IC Compiler 1 Lab Guide.pdf而写，一共分为两个部分，==Q&A==是记录ICC使用中遇到的一些问题，==ICC 1 Lab Guide==主要是熟练一下整个流程并记录各条命令。实际上，这些命令在每个Lab下的.solution/run.tcl中都有记录。

- 由于pdf中没有目录，不利于新手理解整个流程，所以本博客的目的就是把整个流程弄成目录的形式，帮助新手理解。如下图，整个设计流程一目了然，还可以很方便的进行跳转。

  <img src="https://i.loli.net/2020/09/17/Cy5pNK2O34txRXY.png" style="zoom: 67%;" />

- 本博客中还会加入一些==名词解释==，帮助大家更好地理解各个专有名字的意思以及它们的作用。

- 后面就基本使用英文来写了吧，锻炼一下自己的英语水平，顺带记一下专有名词，英语很重要哦。

# ICC 1 Lab Guide

## Note: 

- This is just a simple flow record, refer to the ‘IC Compiler 1 Lab Guide.pdf’ for the meaning of each step.
- A complete command script is available to help you in each lab directory if you encounter problems or get stuck: `.solutions/run.tcl`.
- If you dont understand some steps of a specific flow, just record and skip them and then continue to study. When you finish all flows of ICC design, ==go over the manual again== and this will help you understand it better.
- Before a lab learning start, you’d better read the learning objectives seriously, which will help you understand the lab better and drive you study with a specific purpose.

## 1.Data Setup & Basic Flow(P27)

### Learning Objectives

- Walk through the “data setup” process of creating and maintaining a Milkyway database to hold your design data.
- Run a complete basic flow, from loading a floorplan through routing.

### Create a Milkyway library

```tcl
create_mw_lib -technology $tech_file -mw_reference_library "$mw_path/sc $mw_path/io $mw_path\
  /ram16x128" -bus_naming_style {[%d]} -open $my_mw_lib
  
# Open Library
open_mw_lib 
```

### Load the Netlist, TLU+, Constraints and Controls(data setup)

```tcl
#############################################################
# Netlist
# Open Library & Import Designs
import_designs $verilog_file -format verilog -top $top_design

#############################################################
# TLU+
# Set TLU+ files
set_tlu_plus_files -max_tluplus $tlup_max -min_tluplus $tlup_min -tech2itf_map $tlup_map

# Check the physical and logical libraries for consistency
# Recommended check
# set_check_library_options -all
# Default check
check_library

# Check that TLU+ files are attached and that they pass three sanity checks
check_tlu_plus_files

# Verify that the specified link libraries have been loaded
list_libs

# Define the "logical" connections between power/ground pins and nets
source $derive_pg_file
check_mv_design -power_nets

#############################################################
# Constraints
# Apply the top level design constraints
read_sdc $sdc_file

# Check if any key timing constraints (for example clocks, input/output constraints) are missing
check_timing

# Check to see what "timing exception" constraints are applied to your design
report_timing_requirements

# Check to see if timing analysis was disabled along any paths
report_disable_timing

# Check to see if the design has been configured for a specific "mode" or "case", for example "functional" versus "test" mode
report_case_analysis

# Verify that the clocks are appropriately modeled
report_clock 
report_clock -skew

#############################################################
# Control
# Apply some timing and optimization controls which are specified in ./scripts/opt_ctrl.tcl
source $ctrl_file

# Run a "zero-interconnect"(zic) timing report
source scripts/zic_timing.tcl 

# Remove the ideal network definition so that it will be buffered during physical design
remove_ideal_network [get_ports scan_en]

# Save the cell and notice the new binary files under risc_chip.mw/CEL
save_mw_cel -as RISC_CHIP_data_setup
```

### Basic Flow: Design Planning

```tcl
# Read in the provided DEF file
read_def $def_file

# Ensure that standard cells will not be placed under the power and ground metal routes(this constraint is not part of DEF)
set_pnet_options -complete {METAL3 METAL4}

# Save the design cell and notice the new binary files under risc_chip.mw/CEL
save_mw_cel -as RISC_CHIP_floorplanned
```

### Basic Flow: Placement

```tcl
# Place and optimize the design for timing, and generate a timing report
place_opt
redirect -tee place_opt.timing {report_timing}

# Analyze congestion
report_congestion -grc_based -by_layer -routing_stage global

# Save the design cell
save_mw_cel -as RISC_CHIP_placed
```

### Basic Flow: CTS

```tcl
# Remove the "clock uncertainty" and enable hold-time fixing
# Using default settings to generate the clock tree
remove_clock_uncertainty [all_clocks]
set_fix_hold [all_clocks]
clock_opt 
redirect -tee clock_opt.timing {report_timing}

# Save the design cell
save_mw_cel -as RISC_CHIP_cts

exit
```

###  Basic Flow: Routing

```tcl
# Invoke IC Compiler's GUI
icc_shell -gui

# Open Libraries
open_mw_lib $my_mw_lib
open_mw_cel RISC_CHIP_cts

# Re-apply the timing and optimization controls, which were applied during data setup
source $ctrl_file

# Route the design. This will take care of all the signal nets(the clock nets were already detail-routed by clock_opt)
route_opt

# Generate a timing report
view report_timing -nosplit
v rt

# By default timing reports show maximum delay or setup timing
v rt -delay min

# Generate physical design statistics
report_design -physical

# Save the design
save_mw_cel -as RISC_CHIP_routed

# Quit the IC Complier shell
exit or quit
```

## 2.Design Planning(P43)

### Load the Design

```tcl
# Invoke IC Compiler and start the GUI
cd lab2_dp
icc_shell -gui

# Open the orca_setup cell from the orca_lib.mv design library
open_mw_cel -library orca_lib.mw orca_setup

# Apply timing and optimization controls which are specified in ./scripts/opt_ctrl.tcl
source script/opt_ctrl.tcl

# Switch to the Design Planning task
gui_set_current_task -name {Design Planning}
```

### Initialize the Floorplan

```tcl
# Create the corner and P/G cells and define all pad cell positions using a provided script
source -echo scripts/pad_cell_cons.tcl

# Initialize the floorplan, including core utilization and core to left/right/bottom/top spacing 
initialize_floorplan -core_utilization 0.8 -left_io2core 30.0 -bottom_io2core 30.0 -right_io2core 30.0 -top_io2core 30.0

# Insert the pad filler
insert_pad_filler -cell "pfeed10000 pfeed05000 pfeed02000 pfeed01000 pfeed00500 pfeed00200 pfeed00100 pfeed00050 pfeed00010 pfeed00005"
# You could also type this
source scripts/insert_pad_filler.tcl

# Make the "logical" connection(no physical routing) between the power/ground signals and all power/ground pins of the I/O pads, macros and standard cells
source -echo scripts/connect_pg.tcl

# Build the PAD area power supply ring
create_pad_rings

# Save the design as "floorplan_init"
save_mw_cel -as floorplan_init
```

### Preplace the Macros Connected to I/O Pads

```tcl
# Manually place the macros in the core area such that their connections to the I/O pads are as short as possible

# To ensure that the three macros are placed as expected, you can source the following script
source -echo scripts/preplace_macros.tcl
```

### Perform Virtual Flat Placement

```
# Verify that the current VF placement strategy options have default settings
report_fp_placement_strategy

# Apply a sliver size of 10 to prevent standard cells from being placed in narrow channels(<10um) between macros
set_fp_placement_strategy -sliver_size 10

# Execute a timing-driven VF placement with "no hierarchy gravity"(to ensure that the "logical hierarchy" does not affect placement of this non-hierarchical or flat layout)
create_fp_placement -timing_driven -no_hierarchy_gravity
```





## 3.Placement(P63)





## 4.Clock Tree Synthesis(P75)





## 5.Routing(P91)





## 6.Chip Finishing(P103)









# Q & A

## Q：Module … is not defined

<img src="https://i.loli.net/2020/09/12/3zGocJqmwf4B15W.png"/>

## A:

在按照IC Compiler 1 Lab Guide手册学习ICC的时候，遇到了上图所示的问题，解决方法为：[关于ICC-200809-SP5 "module is not defined" 原因以及解决办法](http://bbs.eetop.cn/forum.php?mod=viewthread&tid=209153&fromuid=1769107)
将IC_Compiler_2010.12-SP2/ref/mw_lib中的io、ram16x128、sc中的CEL、FRAM、LM中的文件中的`_数字`全部命名为`:数字`即可，使用命令：` rename _ : *`（这个命令不一定全部适用，需要自己改改，参考：[linux中mv和rename的区别](https://blog.csdn.net/longyulu/article/details/8547246?utm_medium=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase)）。建议直接把压缩包传到虚拟机（工作站）中，然后解压就不会出现这种问题了。

- 命名前

<img src="https://i.loli.net/2020/09/12/T7fVegXU6qrk2ij.png"/>

- 命名后

<img src="https://i.loli.net/2020/09/12/jtwXmIxiOCTQgcB.png"/>

将三个文件夹中的文件名都修改之后，重新执行，便可正常运行。

## Q: What are the differences among CEL, FRAM and LM?

## A:

Please refer to: [ICC 中的 FRAM 什么意思？](http://bbs.eetop.cn/forum.php?mod=viewthread&tid=347447&fromuid=1769107)
![](http://bbs.eetop.cn/data/attachment/forum/month_1208/120821210353efabb12293c964.jpg)

