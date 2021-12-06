# Preface

本文中英文结合（学习一些专有名词），主要介绍ICC II软件进行后端设计的主要流程，在阅读之前需要对数字IC设计流程有一定的了解。

逻辑综合相关知识请查看：[Synopsys逻辑综合及DesignCompiler的使用](https://blog.csdn.net/qq_42759162/article/details/105541240)（想了解逻辑综合的可以看看这个，但内容较多）

数字IC设计整体流程请查看：[关于数字IC后端设计的一些基础概念与常识](https://blog.csdn.net/mjwwzs/article/details/77413454?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.channel_param)（基础不太扎实的建议先看一遍这个）

有关ICC中用到的一些名词解释请查看：[干货满满--数字后端设计及ICC教程整理](https://mp.weixin.qq.com/s/kz1J8HGEgzWxoe-csWijjg)（建议关注公众号，不是打广告，我也是写这篇blog的时候发现的，真的非常全面）（建议先看我这个博客了解大体流程，然后再看他这个，加深了解；或者看我的博客时还有一些专有名词不太理解啥意思，就可以去他那里找一找解释，效果更佳）

本文类似于笔记形式，整合了网上一些相关资料，参考资料都在最后Reference中列出，尊重知识产权。

# Content

[toc]

# Blocks and Design Libraries

<img src="https://i.loli.net/2020/09/07/IzLxwlK6GCHfb4X.png" style="zoom:67%;" />

- block是所有design数据的载体，给网表创建一个block
- 针对block的一些命令都是*_block来命名的，比如open_block, save_block
- block 包括 design data 
- design library 包括 block、technology data

```
read_verilog
open_block   save_block
create_lib   open_lib   save_lib
```

> 门级网表文件（.v文件）
>
> 该文件可以用逻辑综合工具（如Design Compiler, DC）来产生，某些部分可以人为手工修改/编写，在导入ICC中之前，首先需要检查网表的质量，以尽早排除可能造成后端设计困难的问题，比如浮动输入信号、多驱动、未采用寄存器输入输出、输入到寄存器、寄存器到寄存器、寄存器到输出、扇入扇出等。这些问题如果及时发现，并在前端进行改善会比较容易，且非常有利于后端设计的顺利进行。

# Objects

<img src="https://i.loli.net/2020/09/07/g7S9h2nmGsYfQOZ.png"/>

- object classes：design, port, cell, pin, net…

```
get_*
help_attributes
```

## Attributes of Objects

每个objects都有属性，属性有值

<img src="https://i.loli.net/2020/09/07/qEmsZoeG8XkQn3M.png"/>

```
report_attributes -application [get_selection]
```

## Application Options(App Options)

<img src="https://i.loli.net/2020/09/07/6KH2oIWO9wP1EzV.png"/>

- *place_opt.flow.do_spg*控制是否考虑SPG放置

- *naming convention*（惯例、习俗、常规）：命名约定

- *==SPG==: Synopsys Physical Guidence*，在综合的时候采用后端物理floor plan的信息去综合，突出一些问题信息，用来给后端流程使用

  ```
  place_opt.flow.do_spg
  category.sub_category.option_name
  ```

## Finding/Applying Application Options

<img src="https://i.loli.net/2020/09/07/beQocur5WG9LqzK.png"/>

```
report_app_options    get_app_options
report_app_options time.*  (报道Timing相关的app_options)
report_app_options -non_default  (一般都是默认值，该命令可知道哪些进行了修改)
set_app_options -name time.remove_clock_reconvergence_pessimism -value true  (设定app_options)
```

# IC Compiler II GUI

<img src="https://i.loli.net/2020/09/07/tkVybNvmEx4dowA.png"/>

## Built-In Script Editor

<img src="https://i.loli.net/2020/09/07/l2kqH8F5go7BjhT.png"/>

# Design Setup

<img src="https://i.loli.net/2020/09/07/3YrTImcbhJ4zLal.png"/>

- 输入IC Compiler 的有：门级网表、库文件、时序约束。
- 输出IC Compiler 的是layout（常用格式是GDSII）

![](https://img-blog.csdn.net/20170819155145938?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWp3d3pz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## Overview

<img src="https://i.loli.net/2020/09/07/yAw5xODmvKQVjkh.png"/>

- Design Library 包括Block、Cell Libraries（ndm形式）、Technology Library（ndm形式）
- Block包括Gate-Level Netlist（门级网表）以及一些Design约束的一些文件

## NDM

<img src="https://i.loli.net/2020/09/07/fWH6eBX41Qzbs3c.png"/>

- Timing View可以理解为db，包含timing的信息及功耗的信息
- design view相当于做design，比如block，design本身中有绕线、place的结果
- frame view和lef是一样的
- *lef*文件是==布局布线==根据使用的cell几何信息库的文件格式，布局布线工具将根据LEF文件的信息决定怎样布局，怎么走线，怎样生成通孔。

> LEF：
>
> (Library exchangeformat)，叫库交换格式，它描述了库单元的物理属性，包括端口位置、层定义和通孔定义。它抽象了单元的底层几何细节，提供了足够的信息，以便允许布线器在不对内部单元约束来进行修订的基础上进行单元连接。它包含了工艺的技术信息，如布线的层数、最小的线宽、线与线之间的最小距离以及每个被选用cell，BLOCK，PAD的大小和pin的实际位置。cell，PAD的这些信息由厂家提供的LEF文件给出，自己定制的BLOCK的LEF文件描述经ABSTRACT后生成，只要把这两个LEF文件整合起来就可以了。

- 其他文件格式说明请查看：[后端设计中常用文件格式说明](https://mp.weixin.qq.com/s?__biz=MzIyMjc3MDU5Mw==&mid=2247483762&idx=2&sn=ea295913e924a8bc9c5429c32cf19043&chksm=e82925bcdf5eacaa5971077ecc74eef4fa5a3041eb80dcb3ffd787ceb624181b610a21cfb69a&scene=21#wechat_redirect)

## block library

<img src="https://i.loli.net/2020/09/07/zacp4Z3B6CVljYk.png"/>

- block library其实也可以理解为ndm
- design view: 默认的视图
- frame view：==有限==的物理视图，其中仅包含执行将块作为实例放置并路由到实例端口所需的信息：块轮廓，引脚，过孔区域和布线阻塞
- abstract view比较重要，是一个简化版的design view，top实现的时候用的就是abstract view，把interface的物理信息以及clock tree上的物理信息提取出来

## IC Compiler II Library Manager

<img src="https://i.loli.net/2020/09/07/hUSw1FgCIbfduT3.png"/>

- ndm产生是用一个ICC II Library Manager
- 针对所有design用到的一些标准单元/存储器/IP/IO，需要知道Timing信息（.db文件）/物理信息（.frame/LEF/GDS）/技术文件
- 通过ICC II Library Manager产生NDM，NDM中包含了logic/physical的信息，然后在ICC II中做使用

### Library Manager(icc2_lm_shell) Flow

<img src="https://i.loli.net/2020/09/07/aS4rRgnfDxL19zF.png"/>

### Library Prep-icc2_lm_shell

<img src="https://i.loli.net/2020/09/07/IGk3ELfyBlY9D7P.png"/>

- 一个简单的脚本

```
create_workspace 需要指定一个名字以及需要的tech file
read_db  read_lef
check_workspace
commit_workspace 保存ndm
```

# APR Flow - Design & Timing Setup

- ==自动布局布线==：布局就是在[版图](https://baike.baidu.com/item/版图/791489)上给单元、宏模块等分配物理位置，使得单元、宏模块等部件互不重叠。该分配需要根据用户给出的特定约束来对代价函数进行优化。布局之后,单元和[引脚](https://baike.baidu.com/item/引脚/10879873)的确切位置己经确定,所需的互联也已经确定。为布线预留的区域称为布线区。布线必须在布线区内进行,要遵循布线规则,不能引起布线的规则违反。

- 中文名

  自动布局布线

- 外文名

  Automatic Placement and Routing, APR

<img src="https://i.loli.net/2020/09/07/od6wIJEVnjW3Ft9.png"/>

- 有了ndm就可以进行design & timing setup

```
create a design library
load the netlist and power intent(UPF)
apply the floorplan 
load the scan chain definitions(scan-DEF)
perform MCMM setup
apply timing and optimization controls
```

- 如今的集成电路（Integrated Circuit，IC）设计往往要求芯片包含多个工作模式，并且在不同工艺角（corner）下能正常工作。工艺角和工作模式的增加，无疑使时序收敛面临极大挑战。
- MCMM（Multicorner-Multimode）技术: 多工艺角多工作模式
- ==corner:==

>不同的晶片和不同的批次之间，MOSFETs参数的变化范围比较大。为减轻设计困难度，需要将器件性能限制在某个范围内，并报废超出这个范围的芯片，来严格控制预期的参数变化。工艺角即为这个性能范围。
>
>5-corner model：
>
>5-corner model有5个corners：TT，FF，SS，FS，SF。前后两个字符分别对应NMOS和PMOS。其中TT是指typical corner。Typical表示晶体管饱和电流的平均值。单一器件所测的结果是呈正态分布的。均值为TT，最小最大限制为SS和FF。饱和电流（Isat）大的器件，阈值电压小（LVT），运行速度快（F）。饱和电流（Isat）小的器件，阈值电压大（HVT），运行速度慢（S）。
>
>不同的工艺不同的device对应的sigma值不同。如果NMOS和PMOS的性能与Typical的偏差在3sigma时，也能满足设计需求，则此corner芯片为3SS或者3FF corner 芯片。

- DEF：

> DEF：(Design exchange format)，叫设计交换格式，是ASCII格式的文件，它描述的是实际的设计，对库单元及它们的位置和连接关系进行了列表，使用DEF来在不同的设计系统间传递设计，同时又可以保持设计的内容不变。DEF与只传递几何信息的GDSII不一样。它可以将设计的逻辑信息和物理信息传递给布局布线工具。逻辑信息包括逻辑连接关系（由网表表示）、grouping信息以及物理约束。物理信息包括布局规划、布局位置及方向、绕线几何数据。

## Create a “Container”: The Design Library

<img src="https://i.loli.net/2020/09/07/3eRdGMU48pmIKaW.png"/>

- design library 的create：
- 指定technology以及一些cell library（cell即ndm）
- default的library是存在memory里面

```
lappend search_path /x/y/libs
create_libs ORCA.dlib \
  -use_technology_lib abc14_9m_tech.ndm \
  -ref_libs {
    abc14_9m_tech.ndm
    abc14_hvt_std.ndm   abc14_svt_std.ndm   abc14_lvt_std.ndm
    abc14_srams.ndm     abc14_ip.ndm
  }
```

## Read the Netlist and Create a Design

<img src="https://i.loli.net/2020/09/07/jnoabk2AU1GTwlS.png"/>

- 读网表生成block

```
lappend search_path ./netlist
read_verilog -top ORCA ORCA.v
link_block
```

## Multiple Modes and Multiple Corners

<img src="https://i.loli.net/2020/09/07/aFEYP39dGAXiRuC.png"/>

- 现在的芯片会在很多的模式以及不同的corner运作
- *multiple modes* 

>standby mode、test mode、low power mode、high performance mode、normal functional mode

- *multiple corners*

> Hi-T Slow、Lo-T Slow、Lo-T Fast、Hi-T Fast、Max Leakage

- more information please check:[数字IC设计中的多模多角MCMM（Mulit-Corner Mult-Mode）](https://mp.weixin.qq.com/s?__biz=MzIyMjc3MDU5Mw==&mid=2247484913&idx=1&sn=c5474b8808bf0758bd3f6f55fdb4c2f0&chksm=e829213fdf5ea82924fb6557a13559cb2d6eee976597b06f560805bf115aa7aa75e8817aed42&scene=21#wechat_redirect)

### Concurrent MCMM Optimization

- Concurrent: 并行性、同时存取、并发、并行

<img src="https://i.loli.net/2020/09/07/AgylqkjBUpWKIzb.png"/>

- 因为应用场景的不同，这些mode和corner组合在一起形成一个scenario，即情景，场景

> FUNC_SLOW Scenario = FUNC Mode + SLOW Corner

- IC可以在不同的场景之下同时进行优化，改进场景中的每个冲突，同时尝试不在另一个场景中导致/增加冲突
- violation: 违法，违反

### Mode, Corner and Scenario Setup

<img src="https://i.loli.net/2020/09/07/hZD53V1JBxgcLHd.png"/>

- 先define mode 和 corner，然后mode和corner组合在一起就是scenarios

```
create_mode M1
create_mode M2

create_corner C1;  // Common corner to both modes

create_scenario -mode M1 -corner C1 -name M1_C1
create_scenario -mode M2 -corner C1 -name M2_C1
```

### Loading Constraints

<img src="https://i.loli.net/2020/09/07/GgHjsz2QTVxZirf.png"/>

- 得到scenario之后，需要把一些约束文件读进来
- populate: 填充，迁移、移居，居住于、生活于

```
current_scenario M1_C1                  current_scenario M2_C1
read_sdc C1_corner.sdc                  read_sdc M2_mode.sdc
read_sdc M1_mode.sdc                    read_sdc M2_C1_scenario.sdc
read_sdc M1_C1_scenario.sdc

read_sdc global_constraints.sdc
```

## Defining PVT Directly - Recommended

<img src="https://i.loli.net/2020/09/07/8MyPaz1pU5hb493.png"/>

- 针对每一个scenario去指定相应的process/voltage/temperature
- 建议使用这种直接指定的方式赋值

> set_process_number 0.99
>
> set_voltage 0.75 -object_list VDD
>
> set_voltage 0.95 -object_list VDDH
>
> set_temperature 125

## Specify TLUplus Parasitic RC Models

<img src="https://i.loli.net/2020/09/07/nAcNIeivfxquSdH.png"/>

- 把寄生参数提取的模型弄进来
- 针对每个corner去把对应corner的TLUplus文件读进来
- TLUplus可以是之前tech库里的ndm读进来，也可以后续design library中指定

```
#if the TLUplus models have not been loaded into a technology library, they can
#be loaded into the design library:
#read_parasitic_tech -tlup $TLUPLUS_MAX_FILE -name maxLTU
#read_parasitic_tech -tlup $TLUPLUS_MIN_FILE -name minTLU

set_parasitic_parameters -corner c_slow -library $(techlib) -early_spec maxTLU -late_spec maxTLU
set_parasitic_parameters -corner c_fast -library $(techlib) -early_spec minTLU -late_spec minTLU
```

> **tluplus文件**
>
> 寄生RC查找表（存储RC系数的二进制表格式），ICC使用网络几何形状以及该文件来计算互联电阻电容。TLUPlus模型通过包括宽度，空间，密度和温度对电阻系数的影响，可以实现精确的RC提取结果。 
>
> 若tluplus文件没有时，可由Foundry给的.itf转成tluplus。其中.itf文件全称是Interconnect Technology Format
>
> itf文件由foundry提供的提供，用于生成tluplus文件，在ICC流程中使用。用Synopsys公司的Star-RCXT，在shell下用此命令就行：
>
> 　　grdgenxo -itf2TLUPlus -i <ITF file> -o <TLU+ file>

## Controlling Scenario Analysis / Optimization

<img src="https://i.loli.net/2020/09/07/lOiwIT2t3rVNdKQ.png"/>

- simultaneously: 同时

```
create_scenario 
set_scenario_status
```

# APR Flow - Floorplan Definition

<img src="https://i.loli.net/2020/09/07/lmXFek4nRrfx5bt.png"/>

## Floorplanning Overview

<img src="https://i.loli.net/2020/09/07/K9aVu6xJtXG7Avi.png"/>

- Defining
  - 定义了大小和形状
  - 电压域的形状和位置
  - Macro的位置
  - I/O pin的位置
  - 为了避免拥堵，可能使用一些标准单元的blockages

- 产生电源网络PNS
- 写出floor

## Create the Initial Floorplan

<img src="https://i.loli.net/2020/09/07/NRoJq7ABx5U2LGy.png"/>

```
initialize_floorplan -shape U 
```

## Place Macros and Standard Cells

<img src="https://i.loli.net/2020/09/07/CL9YK85WUc3eN2H.png"/>

- coarse: 粗糙的
- 把macro和标准单元都摆进去
- 在运行place opt之前，请修复所有宏的位置

```
create_placement -floorplan [-congestion] 
set_fixed_objects  [get_flat_cells -filter "is_hard_macro"]在place_opt之前要把hard_macro fix住
```

## Place I/O Pins

<img src="https://i.loli.net/2020/09/07/sH2wlDfYGn1ygbK.png"/>

- 摆完了macro之后需要摆I/O Pins
- 通常摆macro时会把他摆在block的边上，但有时候会挡住macro
- 摆pin时需要注意，有时候block之间会有一些交互，故可能会有对pin block摆放的约束；或者针对特殊的pin，比如信号pin、clock、差分信号pin等的摆放有些特殊要求，如果有要求，在摆pin之前需要把它读进来，用place_pins -self把所有的pin摆放到正确的位置上。

```
set_block_pin_constraints -self -allowed_layers "M3 M4" -sides "1 2 3" | -exclude_sides "4 5 6"
set_individual_pin_constraints -ports ...
place_pins -self
```

## Power Planning Challenges

<img src="https://i.loli.net/2020/09/07/8IZPHd1uXcwnUxV.png"/>

- 构建电源网络

## Pattern_Based Power Network Synthesis

<img src="https://i.loli.net/2020/09/07/qEYj2BwfSvNAcrQ.png"/>

- topology: 拓扑学
- no fixed coordinates: 无固定坐标
- coordinate: n：坐标，套装；v：搭配，协调
- 首先define一个区域给它
- define power网格的结构：层次、线宽、线与线之间的距离
- 把pattern放到PG region中，同时可也以apply到voltage area 或者是bounds，需要指定打哪些net，比如VDD、VSS等；这样做的好处是，全程都不会有固定坐标的方式，当floorplan后续有一些变化时，就很灵活，不需要手动改这些数值
- compile power network

## Write out the Floorplan for ICC II/DC-G

<img src="https://i.loli.net/2020/09/07/mtYkUKhoTayf5Es.png"/>

- 把floorplan信息写下来，如果需要重新跑一下ICC，可以引用之前存下来的信息
- 如果送到前端综合的话，是不需要包含标准单元的placement的，DC在综合的时候会重新考虑macro的摆放

```
write_floorplan -output ORCA_TOP.fp
write_floorplan -format icc -output ORCA_TOP.fp.dc -net_types {power ground} -include_physical_status {fixed locked}
```

# APR Flow - Placement & Optimization

<img src="https://i.loli.net/2020/09/07/8lOXbJiK6Yzrhj7.png"/>

## Key Steps of the Placement Phase

<img src="https://i.loli.net/2020/09/07/GLlxTCQNvBWmibu.png"/>

## Design and Flow Requirement Setup Steps

<img src="https://i.loli.net/2020/09/07/GQAHWzSDV1j38IL.png"/>

- 指定scenario去优化
- 选择是否打开SPG Flow，SPG就是之前DC-G综合之后产生一个初始位置信息，ICC II可以直接使用这个信息做下面的优化
- remove掉不需要的ideal networks
- 指定特定类型的library cell去使用
- 选择是否优化leakage或者dynamic或者leakage+dynamic的total优化
- SPG：synopsys physical guidence

<img src="https://i.loli.net/2020/09/09/QUSeBvpLkaZxiqH.png"/>

## Congestion-Focused Setup Steps

<img src="https://i.loli.net/2020/09/07/4SaoGyl5IJdDYT2.png"/>

- density: 密度
- QoR（quality of results）主要分为三部分：Congestion/Timing/Power
- Congestion
  - 跑完place_opt后先分析congestion和cell/pin density
  - 如果有拥堵问题的话，可以使用以下方法去解决congestion
- Timing
- Power/Area

## The Five Stages of place_opt

<img src="https://i.loli.net/2020/09/07/ySnr6VUufPBTgvM.png"/>

- ==five stages==: initial_place/initial_drc/initial_opto/final_place/final_opto

## Placement and Logic Optimization: place_opt

<img src="https://i.loli.net/2020/09/07/PmZlqs1gUHYwtzn.png"/>

- place_opt默认会直接跑完五个阶段
- 可以使用-from/-to控制阶段，可以提高效率
- 可以通过app options控制：congestion/timing等

## Recommended place_opt Exploration Flow

<img src="https://i.loli.net/2020/09/07/IX2iflZxh7DuMvs.png"/>

## Example Script

<img src="https://i.loli.net/2020/09/07/SpWjwsztVLIhok3.png"/>

```
open_lib design.dlib
open_block floorplan

#Place setup
remove_ideal_network -all
set_lib_cell_purpose -include none [get_lib_cells "*/*BUF_X64* */*REG_ulvt*"]
set_app_options -list [opt.power.mode none | leakage | dynamic | total]

#Apply Place configuration steps, as needed
set_scenario_set_scenario_status * -active false
set_scenario_status <list_of_placement_scenarios> -active true
set_scenario_status *corner_FAST -setup false
set_scenario_status mode_TEST* -leakage_power false

#Enable SPG, if applicable:
set_app_options -list {place_opt.flow.do_spg true}

place_opt
```

# APR Flow - CTS & Optimization

<img src="https://i.loli.net/2020/09/07/ZR7Ue9p5w2Ti1co.png"/>

## The definition of CTS

>    在大规模集成电路中，大部分时序元件的数据传输是由时钟同步控制的时钟频率决定了数据处理和传输的速度，时钟频率是电路性能的最主要的标志。在集成电路进入深亚微米阶段，决定时钟频率的主要因素有两个，一是组合逻辑部分的==最长电路延时==，二是同步元件内的==时钟偏斜(clock skew)==,随着晶体管尺寸的减小，组合逻辑电路的开关速度不断提高，时钟偏斜成为影响电路性能的制约因素。时钟树综合的主要目的是减小时钟偏斜。
>
>    以一个时钟域为例，一个时钟源点(source )最终要扇出到很多寄存器的时钟端(sink)，从时钟源扇出很大，负载很大，时钟源是无法驱动后面如此之多的负载的。这样就需要一个时钟树结构，通过一级一级的buffer去驱动最终的叶子结点(寄存器)。

## Clock Tree Synthesis Goal and Flows

<img src="https://i.loli.net/2020/09/07/LizMatVdgZYHNfB.png"/>

- CTS的目的：
  - 建立时钟树缓冲结构
  - 给时钟网络布线
  - 优化数据路径逻辑以建立和保持时序以及DRC
- 支持两种CTS流
  - 典型的CTS流：首先做CTS，然后数据路径优化
  - 并行时钟、数据流：CTS和数据路径优化并行执行
    - 建议用于时序关键型设计

## Comparing Classic CTS versus CCD

<img src="https://i.loli.net/2020/09/07/xpYZdqg1ny28r6W.png"/>

- Classic CTS不看data path，只看clk入点到每个reg点之间的tree是不是平的，减少skew偏差
- 优化时data path是一个untouched状态
- CCD优化时可能会动到clock tree，而Classic CTS不会

## Clock Tree Balancing Setup

<img src="https://i.loli.net/2020/09/07/ZH2TKGpwzy9NsiD.png"/>

```
# 设定clock tree balance
set_clock_balance_points

# 指定期望的延时以及相互之间的偏差
set_clock_tree_options

# 控制CTS选择哪种cell
set_lib_cell_purpose -include cts $cts_cells
set_dont_touch $cts_cells false

create_clock_balance_group

derive_clock_balance_constraints -slack_less_than -0.3
```

## Non-Default Rules

<img src="https://i.loli.net/2020/09/07/R8BG5VPEiZO9CgH.png"/>

## Defining/Applying NDR

<img src="https://i.loli.net/2020/09/07/Y7k89JRuW4lh5ej.png"/>

```
create_routing_rule 2xs_2xW_CLK_RULE -width {M1 0.11 M2 0.11 M3 0.14 M4 0.14 M5 0.14}\
  -spacings {M1 0.4 M2 0.4 M3 0.48 M4 0.48 M5 1.1}
  -cuts {
  	{VIA3 {Vrect 1}}\
  	...             \
  	{VIA5 {Vrect 1}}\
  }
set_clock_routing_rules -rule 2xs_2xW_CLK_RULE -min_routing_layer M4 -max_routing_layer M5
```

## Defining CTS-Specific DRC Values

<img src="https://i.loli.net/2020/09/07/VvpqQkslSweOuYt.png"/>

```
set_max_transition <value> -clock_path [all_clocks]     (default: 0.5ns)
set_max_capacitance <value> -clock_path [all_clocks]    (default: 0.6pF)

set_max_transition 0.2 -clock_path -scenarios "S1 S4" [get_clocks SYS_CLK]
```

## CTS Execution

<img src="https://i.loli.net/2020/09/07/VAZXumwbrYvNzO1.png"/>

```
clock_opt

#CCD enabled
clock_opt.flow.enable_ccd

#four stages of clock_opt
bulid_clock -> route_clock -> final_optp -> global_route_opt(optional)
```

## Analyzing CTS Results: Clock QoR Report

<img src="https://i.loli.net/2020/09/07/eL2tsZQTdlYhEj3.png"/>

```
report_clock_qor [-type area | balance_groups | drc_violators | latency | local_skew \
	| power | robustness | structure | summary] \
	[-histogram_type latency | transition | level |...]
	[-modes ...] [-corners ...]...
```

## Analyzing CTS Results: Clock Timing Report

<img src="https://i.loli.net/2020/09/07/1fTlDhLcq7CikmI.png"/>

```
report_clock_timing -type summary | transition | latency | ...
	-modes {m1 m2}
	-corners {c1 c2}...
```

## Example Script

<img src="https://i.loli.net/2020/09/07/ohicMDj1kXHO7F4.png"/>

```
open_lib design.dlib
open_block place

#CTS setup
source clock_tree_balance.tcl
source clock_routing_tules.tcl
source clock_constraints.tcl

#Apply CTS configuration steps, as needed
set_scenario_status -active true [all_scenarios]
set_scenario_status {s2 s4} -hold true
set_app_options -name clock_opt.hold.effort -value high
set_app_options -name cts.compile.enable_global_route -value true
set_app_options -name opt.common.allow_physical_feedthrough -value true

#Enable CCD, if applicable:
set_app_options -name clock_opt.flow.enbale_ccd -value true

clock_opt
```

# APR Flow - Routing & Optimization

<img src="https://i.loli.net/2020/09/07/4nXcv2yO69B5Zuz.png"/>

## Routing Phase Goal

<img src="https://i.loli.net/2020/09/07/oWb4VzAJKxm2tI7.png"/>

- routing阶段的目的是：
  - 以最小的物理DRC违规路由所有信号网
  - 优化定时、DRC和电源的数据路径逻辑
  - 可选择执行后路CTO或CCD

## Recommended Routing Flow 

<img src="https://i.loli.net/2020/09/07/g3ai97BwXr5q2mV.png"/>

```
route_auto

#Routing performs
Global Routing -> Track Assignment -> Detail Routing

route_opt
```

- 并行优化
  - 时序和最大转换/最大电容（默认）
  - 时钟树、电源（可选择）
  - 可选地使用PrimeTime延迟计算和StarRC提取

## Check Zroute DRC Violations

<img src="https://i.loli.net/2020/09/07/1q2u63hntQHWrOe.png"/>

```
check_routes
```

## Example Script

<img src="https://i.loli.net/2020/09/07/wKNiFEOjqbsGrUx.png"/>

```
open_lib design.dlib
open_block cts

#route setup
source antenna_rules.tcl
set_app_options -list {
	route.global.timing_driven true
	route.track.timing_driven  true
	route.detail.timing_driven true
}
set_app_options -name time.si_enable_analysis -value true

#Apply route configuration steps, as needed
set_scenario_status -active true [all_scenarios]
set_scenario_status {s2 s4} -hold true

#Enable CCD, if applicable:
set_app_options -name route_opt.flow.enable_ccd -value true

route_auto
route_opt
```

# APR Flow - Signoff

<img src="https://i.loli.net/2020/09/07/bBTNt78d4ySJWm6.png"/>

## Signoff

<img src="https://i.loli.net/2020/09/07/5siPCv71zOgyH3L.png"/>

- 主要是做一些Timing ECO或者一些Functional ECO, 就是进行DRC和LVS物理验证。
- ASIC设计流程中 验证测试完成后的确认叫sign-off（字面意思就是负责人签字）包括前端signoff和后端signoff 后端signoff之后就是tape-out

> 1、什么是signoff？
> signoff，签发。
> 后端所说的signoff，是指将设计数据交给芯片制造厂商生产之前，对设计数据进行复检，确认设计数据达到交付标准，这些检查和确认统称为signoff。
>
> 2、signoff的主要方向
> timing signoff 静态时序验证
> PA signoff 电源完整性分析
> PV signoff 物理验证
> RV signoff 可靠性验证
> FM/CLP signoff 形式验证和低功耗验证
>
> 3、signoff要点
> timing：setup check 建立时间检查——hold check 保持时间检查——drv check 最大传输时间检查和最大电容检查——SI check 信号一致性检查；
> PA signoff：关注芯片功耗，静态和动态IR降，电荷迁移等；
> PV signoff：关注芯片是否满足工艺设计规则，物理设计与逻辑网表的一致性；
> RV signoff：关注ESD，latchup，ERC等检查；
> FM signoff：关注最终输出的逻辑网表与最初输入的逻辑网表之间的一致性；
> CLP signoff：关注在低功耗设计中引入的特殊单元，电源域划分及组成单元的正确性；
>
> 4、通常设计人员所说的第一次signoff指的是代码的冻结freeze，freeze code后，后续所有的代码修改均需提交patch进行审核。

## Prime Time Signoff-driven Physically Aware ECO Flow

<img src="https://i.loli.net/2020/09/07/zxmJ8QygsWrGhpq.png"/>

- Timing ECO

> PrimeTime inputs:
>
> - Netlist(Verilog)
> - Timing constraints(SDC)
> - Power Intent(UDF)
> - Layout(NDM or DEF+Tcl)
> - RC Parasitics with coordinates(SPEF, GPD)
> - Standard cell spacing rules(Encrypted Tcl)
> - Logic dbs
> - Tech Info(CLIB or LEF)
> - Physical libraries(CLIB or LEF)
>
> PrimeTime ouput:
>
> - ASCII ECO file with coordinates

> 时序约束文件（.sdc文件）
>
> 该文件可以由DC工具导出，并人工进行修改，以使其满足设计要求，约束要合理，不能过约束，否则后端软件可能无法达到要求。

## Function ECO Flow

<img src="https://i.loli.net/2020/09/07/63JbRpucVrIPtB5.png"/>

- 在design设计可能会出现bug，后仿时也会出现一些问题，我们要做一些修正，不会从综合开始重新开始，经常使用ECO来解决
- **先比较，把改动的东西写出来；然后apply change，把eco cell摆放进去；最后做一个eco route**

```
#Perform ECO comparison
eco_netlist -by_verilog_file ECO_netlist.v -write_changes ECO_changes.tcl

#Apply ECO changes and place
source ECO_changes.tcl
connect_pg_net
place_eco_cells -cel_changed_cells

#ECO routing and post-route optimization
route_eco -max_detail_wires true -utilize_dangling_wires true -open_net_driven true \ 
  -reroute modified_nets_first_then_others
route_opt
```

## Standard Cell Fillers and Metal Fill

<img src="https://i.loli.net/2020/09/07/XzveOCJaYli8mfy.png"/>

- 标准单元填充以及金属填充

> - Boundary cell insertion
> - place_opt
> - clock_opt
> - Initial Route(route_auto)
> - Post-route Opt(route_opt)
> - Filler cell Insertion
> - Add Metal Fill

## Filler Insertion & Removal

<img src="https://i.loli.net/2020/09/07/D4E1jQpwh65oquJ.png"/>

- Filler cell insertion

```
#Insert filler cells. First cells with metal, then without
#Should be sorted from largest to smallest

set FILLER_CELL_METAL "saed32/FILL128 saed32/FILL64 ... saed32/FILL2 saed32/FILL1"
create_stdcell_fillers -lib_cells $FILLER_CELL_METAL 
connect_pg_net
remove_stdcell_fillers_with_violation

create_stdcell_fillers -lib_cells $FILLER_CELL_NO_METAL
connect_pg_net
```

- Filler cell removal

```
remove_cells [get_cells -hierarchical -filter design_type==filler]
```

## In-Design Signoff DRC Checking and Fixing

<img src="https://i.loli.net/2020/09/07/t9261eUnSVFimXp.png"/>

- eliminate: 消除
- 不需要流式输出设计来运行DRC

## Signoff DRC using IC Validator

<img src="https://i.loli.net/2020/09/07/fphLveSaER5Xl6J.png"/>

## Metal Fill Insertion

<img src="https://i.loli.net/2020/09/07/Ud2XQsaETFPck7O.png"/>

# Customer Support

<img src="https://i.loli.net/2020/09/07/XNy5iAEU7VGnOFx.png"/> 

## Solvnet

<img src="https://i.loli.net/2020/09/07/OSCibaFBzr9TGLZ.png"/>

## Regerence Flow

<img src="https://i.loli.net/2020/09/07/YyzJjqdNZnQDuWv.png"/>



# ICC II介绍

> IC Compiler，简称ICC，是Synopsys新一代布局布线系统（Astro是前一代布局布线系统），通过将物理综合扩展到整个布局和布线过程以及Sign off驱动的设计收敛，来保证卓越的质量并缩短设计时间。上一代解决方案由于布局、时钟树和布线独立运行，有其局限性。IC Compiler的扩展物理综合(XPS)技术突破了这一局限，将物理综合扩展到了整个布局和布线过程。IC Compiler采用基于TCL的统一架构，实现了创新并利用了Synopsys的若干最为优秀的核心技术。作为一套完整的布局布线设计系统，它包括了实现下一代设计所必需的一切功能，如物理综合、布局、布线、时序、信号完整性(Signal Integrity, SI)优化、低功耗、可测性设计(Design For Test, DFT)和良率优化。新版ICC运行时间更快、容量更大、多角/多模优化(MCMM)更加智能、而且具有改进的可预测性，可显著提高设计人员的生产效率。同时，新版本还推出了支持45 nm、32 nm技术的物理设计。IC Compiler正成为越来越多市场领先的IC设计公司在各种应用和广泛硅技术中的理想选择。新版的重大技术创新将为加速其广泛应用起到重要作用。IC Compiler引入了用于快速运行模式的新技术，在保证原有质量的情况下使运行时间缩短了35%。新版增加了集成的、层次化的设计规划的早期介入，有助于用户高效处理一亿门级的设计。提高生产能效的另一个关键在于物理可行性流程，它能够使用户迅速生成和分析多次试验布局，以确定具体实现的最佳起始值。

## ICC命令集

请查看：[ICC 命令集](https://mp.weixin.qq.com/s?__biz=MzIyMjc3MDU5Mw==&mid=2247484094&idx=1&sn=120c943e284c8fcb13e2924df2b39071&chksm=e8292670df5eaf66d5e26d52efd31825ed50c0a02b21e405723c8687dceb5cd26c414327e4e1&scene=21#wechat_redirect)

==注：==该文只是总结了ICC的一些命令，但是没有对应的解释，各命令具体含义，请在ICC命令行man一下进行查阅。

## 多级物理层次

<img src="https://i.loli.net/2020/09/08/ANstWqJxRKLgYfa.png"/>

- 不支持多级物理层次的话就得并行，支持的话就可以包含进去

<img src="https://i.loli.net/2020/09/08/GOw5j3ualTLEZFd.png"/>

## 可扩展Timer

<img src="https://i.loli.net/2020/09/08/yJFndh4mODwczj5.png"/>

- Timer可进行时序运行
- ICCII是基于mode而不是scenario，一个mode对应的库有不同的电压/温度/process，这样就可以对不同的PVT进行插值，从而增加更多的灵活性

## 并行优化

<img src="https://i.loli.net/2020/09/08/cHNXuE7GrzPUdYj.png"/>

## preroute optimization

<img src="https://i.loli.net/2020/09/08/kEncXv6YufOgWzs.png"/>

- 首先是place，place之后有一个early clock，即做一个早期的CTS，目的是在place阶段考虑到clock对于绕线以及clock cell/clock buffer面积的影响；如果对IC进行优化的话，因为无法知道ICG的slack，因为你没有做tree，即early clock就是做一个初期的tree来预估ICG的timing，然后进行优化；故在place阶段需要把early clock进行使能，使它在做完timing之后时间都很match
- place之后的timing可以和cts之后的timing一致性做的非常好

## 主要特点

<img src="https://i.loli.net/2020/09/08/fSLUvd486M1YNrV.png"/>

## Fusion with redhawk

<img src="https://i.loli.net/2020/09/08/OIw78Z1ebygG5Wn.png"/>

## Fusion with StarRC & PT/PX

 <img src="https://i.loli.net/2020/09/08/Mli96pnTDO3BrZX.png"/>

<img src="https://i.loli.net/2020/09/08/D1XRlbH2wj58crV.png"/>

## Fusion with ICV

<img src="https://i.loli.net/2020/09/08/3BIGhLrVbWdTYjX.png"/>

## other fusion

<img src="https://i.loli.net/2020/09/08/WfoY1UTlathLgvz.png"/>

## PT StarRC直接读取ICCII Database

<img src="https://i.loli.net/2020/09/08/Uopc2HQxeDYFjmV.png"/>

# Reference

[新思在线课程：IC Compiler](https://players.brightcove.net/6024607096001/default_default/index.html?videoId=6143948276001)

[极术干货|Mount-新一代布局总线系统IC Compiler II 初识（PPT下载+视频回放）](https://aijishu.com/a/1060000000126998)

[process corner工艺角](https://blog.csdn.net/hepiaopiao_wemedia/article/details/98077146)[什么是itf文件？什么是TLUplus文件？](https://www.cnblogs.com/lantingyu/p/12106943.html)

[芯片Timing sign-off Corner理解](https://www.cnblogs.com/gujiangtaoFuture/articles/10097814.html)

[后端signoff含义](https://blog.csdn.net/weixin_45270982/article/details/106155993)

[关于数字IC后端设计的一些基础概念与常识](https://blog.csdn.net/mjwwzs/article/details/77413454?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.channel_param)

[干货满满--数字后端设计及ICC教程整理](https://mp.weixin.qq.com/s/kz1J8HGEgzWxoe-csWijjg)

[IC Compiler简介](https://mp.weixin.qq.com/s?__biz=MzIyMjc3MDU5Mw==&mid=2247483762&idx=1&sn=70359a38b63a7cc48e381902b74c0757&chksm=e82925bcdf5eacaa42fdf6410092dd834eeb463d8be970a62a4022483ba312a8907a771a8f5b&scene=21#wechat_redirect)

[ICC后端设计准备-1.数据准备](https://mp.weixin.qq.com/s?__biz=MzIyMjc3MDU5Mw==&mid=2247483782&idx=2&sn=421465ddb17130b5e874d9e81348ad41&chksm=e8292548df5eac5e15bbced502a3f253f3193b5d3086e03ce397c16e0579eb9d34fb668a3e60&scene=21#wechat_redirect)













# Appendix

[Signoff (electronic design automation)](https://en.wikipedia.org/wiki/Signoff_(electronic_design_automation))

## Signoff (electronic design automation)

In the [automated](https://en.wikipedia.org/wiki/Electronic_design_automation) design of [integrated circuits](https://en.wikipedia.org/wiki/Integrated_circuit), **signoff** (also written as **sign-off**) checks is the collective name given to a series of verification steps that the design must pass before it can be [taped out](https://en.wikipedia.org/wiki/Tape-out). This implies an iterative process involving incremental fixes across the board using one or more check types, and then retesting the design. There are two types of sign-off's: [front-end sign-off](https://en.wikipedia.org/w/index.php?title=Front-end_sign-off&action=edit&redlink=1) and [back-end sign-off](https://en.wikipedia.org/w/index.php?title=Back-end_sign-off&action=edit&redlink=1). After back-end sign-off the chip goes to fabrication. After listing out all the features in the specification, the verification engineer will write coverage for those features to identify bugs, and send back the RTL design to the designer. Bugs, or defects, can include issues like missing features (comparing the layout to the specification), errors in design (typo and functional errors), etc. When the coverage reaches a maximum% then the verification team will sign it off. By using a methodology like UVM, OVM, or VMM, the verification team develops a reusable environment. Nowadays, UVM is more popular than others.

### Check types

Signoff checks have become more complex as [VLSI](https://en.wikipedia.org/wiki/VLSI) designs approach [22nm](https://en.wikipedia.org/wiki/22nm) and below process nodes, because of the increased impact of previously ignored (or more crudely approximated) second-order effects. There are several categories of signoff checks.

- [Design rule checking](https://en.wikipedia.org/wiki/Design_rule_checking) (DRC) – Also sometimes known as geometric verification, this involves verifying if the design can be reliably [manufactured](https://en.wikipedia.org/wiki/Semiconductor_fabrication) given current photolithography limitations. In advanced process nodes, [DFM](https://en.wikipedia.org/wiki/Design_for_manufacturability_(IC)) rules are upgraded from optional (for better yield) to required.
- [Layout Versus Schematic](https://en.wikipedia.org/wiki/Layout_Versus_Schematic) (LVS) – Also known as schematic verification, this is used to verify that the [placement](https://en.wikipedia.org/wiki/Placement_(electronic_design_automation)) and [routing](https://en.wikipedia.org/wiki/Routing_(electronic_design_automation)) of the [standard cells](https://en.wikipedia.org/wiki/Standard_cell) in the design has not altered the functionality of the constructed circuit.
- [Formal verification](https://en.wikipedia.org/wiki/Formal_verification) – Here, the logical functionality of the post-[layout](https://en.wikipedia.org/wiki/Integrated_circuit_layout) netlist (including any layout-driven optimization) is verified against the pre-layout, post-[synthesis](https://en.wikipedia.org/wiki/Logic_synthesis) [netlist](https://en.wikipedia.org/wiki/Netlist).
- [Voltage drop](https://en.wikipedia.org/wiki/Power_network_design_(IC)) analysis – Also known as IR-drop analysis, this check verifies if the [power grid](https://en.wikipedia.org/wiki/Power_network_design_(IC)) is strong enough to ensure that the [voltage](https://en.wikipedia.org/wiki/IC_power_supply_pin) representing the binary **high** value never dips lower than a set margin (below which the circuit will not function correctly or reliably) due to the combined switching of millions of transistors.
- [Signal integrity](https://en.wikipedia.org/wiki/Signal_integrity) analysis – Here, noise due to crosstalk and other issues is analyzed, and its effect on circuit functionality is checked to ensure that capacitive glitches are not large enough to cross the [threshold voltage](https://en.wikipedia.org/wiki/Threshold_voltage) of gates along the data path.
- [Static timing analysis](https://en.wikipedia.org/wiki/Static_timing_analysis) (STA) – Slowly being superseded by [statistical static timing analysis](https://en.wikipedia.org/wiki/Statistical_static_timing_analysis) (SSTA), STA is used to verify if all the logic data paths in the design can work at the intended [clock frequency](https://en.wikipedia.org/wiki/Clock_frequency), especially under the effects of [on-chip variation](https://en.wikipedia.org/wiki/Process_corners). STA is run as a replacement for [SPICE](https://en.wikipedia.org/wiki/SPICE), because SPICE simulation's runtime makes it infeasible for full-chip analysis modern designs.
- [Electromigration](https://en.wikipedia.org/wiki/Electromigration) lifetime checks – To ensure a minimum lifetime of operation at the intended clock frequency without the circuit succumbing to electromigration.
- [Functional](https://en.wikipedia.org/wiki/Functional_verification) Static Sign-off checks – which use search and analysis techniques to check for design failures under all possible test cases; functional static sign-off domains include [clock domain crossing](https://en.wikipedia.org/wiki/Clock_domain_crossing), reset domain crossing and X-propagation.

### Tools

A small subset of tools are classified as "golden" or signoff-quality. Categorizing a tool as signoff-quality without vendor-bias is a matter of trial and error, since the accuracy of the tool can only be determined after the design has been fabricated. So, one of the metrics that is in use (and often touted by the tool manufacturer/vendor) is the number of successful tapeouts enabled by the tool in question. It has been argued that this metric is insufficient, ill-defined, and irrelevant for certain tools, especially tools that play only a part in the full flow.[[1\]](https://en.wikipedia.org/wiki/Signoff_(electronic_design_automation)#cite_note-1)

While vendors often embellish the ease of end-to-end (typically [RTL](https://en.wikipedia.org/wiki/Register_transfer_level) to [GDS](https://en.wikipedia.org/wiki/GDSII) for [ASICs](https://en.wikipedia.org/wiki/Application-specific_integrated_circuit), and RTL to [timing closure](https://en.wikipedia.org/wiki/Timing_closure) for [FPGAs](https://en.wikipedia.org/wiki/FPGA)) execution through their respective tool suite, most semiconductor design companies use a combination of tools from various vendors (often called "[best of breed](https://en.wikipedia.org/wiki/Best_of_breed)" tools) in order to minimize correlation errors pre- and post-silicon.[[2\]](https://en.wikipedia.org/wiki/Signoff_(electronic_design_automation)#cite_note-2) Since independent tool evaluation is expensive (single licenses for design tools from major vendors like [Synopsys](https://en.wikipedia.org/wiki/Synopsys) and [Cadence](https://en.wikipedia.org/wiki/Cadence_Design_Systems) may cost tens or hundreds of thousands of dollars) and a risky proposition (if the failed evaluation is done on a production design, resulting in a [time to market](https://en.wikipedia.org/wiki/Time_to_market) delay), it is feasible only for the largest design companies (like [Intel](https://en.wikipedia.org/wiki/Intel), [IBM](https://en.wikipedia.org/wiki/International_Business_Machines), [Freescale](https://en.wikipedia.org/wiki/Freescale_Semiconductor), and [TI](https://en.wikipedia.org/wiki/Texas_Instruments)). As a [value add](https://en.wikipedia.org/wiki/Value_add), several semiconductor foundries now provide pre-evaluated reference/recommended methodologies (sometimes referred to as "RM" flows) which includes a list of recommended tools, versions, and scripts to move data from one tool to another and automate the entire process.[[3\]](https://en.wikipedia.org/wiki/Signoff_(electronic_design_automation)#cite_note-3)

This list of vendors and tools is meant to be representative and is not exhaustive:

- DRC/LVS - [Mentor HyperLynx DRC Free/Gold](https://www.mentor.com/pcb/hyperlynx/electrical-rule-check/drc-editions), [Mentor Calibre](http://www.mentor.com/products/ic_nanometer_design/verification-signoff/physical-verification/), [Magma Quartz](http://www.magma-da.com/products-solutions/verification/quartzDRCLVS.aspx), [Synopsys Hercules](https://web.archive.org/web/20090309103912/http://synopsys.com/tools/implementation/physicalverification/pages/hercules.aspx), [Cadence Assura](http://www.cadence.com/products/mfg/apv/pages/default.aspx)
- Voltage drop analysis - [Cadence Voltus](http://www.cadence.com/products/mfg/voltus/pages/default.aspx), [Apache Redhawk](https://web.archive.org/web/20090412030004/http://www.apache-da.com/apache-da/Home/ProductsandSolutions/SoCPowerNoiseReliability.html), [Magma Quartz Rail](http://www.magma-da.com/products-solutions/lowpower/QuartzRail.aspx)
- Signal integrity analysis - [Cadence CeltIC](http://w2.cadence.com/datasheets/3073E_CeltIC_DS_Fnl.pdf) (crosstalk noise), [Cadence Tempus Timing Signoff Solution](http://www.cadence.com/products/mfg/tempus/pages/default.aspx), [Synopsys PrimeTime SI](http://www.synopsys.com/Tools/Implementation/SignOff/Pages/PrimeTime.aspx) (crosstalk delay/noise), [Extreme-DA GoldTime SI](http://www.extreme-da.com/Gold_Time_Suite.html) (crosstalk delay/noise)
- Static timing analysis - [Synopsys PrimeTime](http://www.synopsys.com/Tools/Implementation/SignOff/Pages/PrimeTime.aspx), [Magma Quartz SSTA](http://www.magma-da.com/products-solutions/verification/quartzssta.aspx), [Cadence ETS](http://www.cadence.com/products/di/ets/pages/default.aspx), [Cadence Tempus Timing Signoff Solution](http://www.cadence.com/products/mfg/tempus/pages/default.aspx), [Extreme-DA GoldTime](http://www.extreme-da.com/Gold_Time_Suite.html)

### References

1. **[^](https://en.wikipedia.org/wiki/Signoff_(electronic_design_automation)#cite_ref-1)**["Vendors should count silicon, not tapeout wins"](https://www.eetimes.com/document.asp?doc_id=1142973). *EETimes*. Retrieved 2019-04-03.
2. **[^](https://en.wikipedia.org/wiki/Signoff_(electronic_design_automation)#cite_ref-2)** DeepChip - [SNUG survey of physical verification tools](http://www.deepchip.com/items/snug07-09.html).
3. **[^](https://en.wikipedia.org/wiki/Signoff_(electronic_design_automation)#cite_ref-3)** [TSMC's sign-off flow](http://www.eetimes.com/showArticle.jhtml?articleID=216900259&printable=true)