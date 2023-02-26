# IP Integrator Setup

## Table of Content

1. [Overview](#overview)
2. [Linux Install](#hlxinst_lin)
3. [Windows Install](#hlxinst_win)
4. [Vivado Overview](#vivado)
5. [Vivado Flows Overview](#projover)
6. [Summary](#summary)



<a name="overview"></a>
# Overview

이 문서에서는 개발자 키트를 복제하고 hdk 설정을 소싱했다고 가정합니다.  그러나 아래 Windows 지침에서는 개발자 키트 및 소스 스크립트를 복제하는 방법과 hdk 설정을 위한 소스 스크립트를 다룹니다.  Vivado GUI 또는 IP 통합기를 사용하기 전에 [customer examples](../README.md)를 빌드/실행하여 F1 FPGA 개발에 익숙해지는 것을 적극 권장합니다.

예제 AFI를 빌드하고 F1에서 실행하는 데 익숙해지면 [IP 통합기 튜토리얼 및 예제](./IPI_GUI_Examples.md) 문서에서 예제 설계, 새로운 설계 및 추가 튜토리얼을 시작하는 데 도움이 될 것입니다.


<a name="hlxinst_lin"></a>
# Linux Install

텍스트 편집기에서 다음 파일을 엽니다. ~/.Xilinx/Vivado/init.tcl 또는 ~/.Xilinx/Vivado/Vivado_init.tcl.

이 파일 중 하나가 없는 경우 디렉터리를 ~/.Xilinx/Vivado로 변경하고 다음 명령을 사용하여 파일을 생성합니다.

`touch Vivado_init.tcl`

다음 명령을 사용하여 $HDK\_SHELL\_DIR의 절대 경로를 가져옵니다.

`echo $HDK_SHELL_DIR`

**참고: $HDK\_SHELL\_DIR이 비어 있거나 <path>/$HDK\_SHELL\_DIR/<shell dir>이 나열되지 않는 경우 [hdk_setup](../README.md)을 소싱해야 할 수 있습니다.**

init.tcl 또는 Vivado\_init.tcl에서 $HDK\_SHELL\_DIR 경로에 따라 다음 줄을 추가합니다.

`source $::env(HDK_SHELL_DIR)/hlx/hlx_setup.tcl`

<a name="hlxhdk_switch"></a>
### Switching between HDK and HLx flows
* ~/.Xilinx/Vivado/init.tcl 또는 ~/.Xilinx/Vivado/Vivado_init.tcl 스크립트는 Vivado가 시작될 때 소싱됩니다. Linux 설치 설정을 진행하면 IP 통합기 기능이 매번 자동으로 로드됩니다. 
* HDK 플로우로 전환하려면 init.tcl 또는 Vivado\_init.tcl 파일에서 `source $::env(HDK_SHELL_DIR)/hlx/hlx_setup.tcl` 줄을 제거하십시오.

<a name="hlxinst_win"></a>
# Windows Install

Download, install, and configure the license for Vivado SDx 2017.4, 2018.2, 2018.3 or 2019.1 for Windows.  More information is provided at:

[On-Premises Licensing Help](../../docs/on_premise_licensing_help.md)

Clone the `https://github.com/aws/aws-fpga` repository either through Github Desktop or Download ZIP and extract to a new folder location on the Windows machine.  This is the install location.

Launch Vivado and determine the path where vivado\_init.tcl or init.tcl is sourced which is found as an INFO message at the top of the Tcl Console.

Open vivado\_init.tcl or init.tcl in a text editor and add the following lines at the top of the file.  Note aws-fpga could have a slightly different name like aws-fpga-master.

`set AWSINSTALL "C:/<replace with install location>/aws-fpga"`

`source $AWSINSTALL/hdk/common/shell_v04261818/hlx/hlx_setup.tcl`

Copy these lines into the TCL console to ensure paths are correct.

An error message will occur either in the TCL console or in a tab about DDR4 models.  Source the following command which only needs to be run once after cloning the github repository.  Note aws-fpga could have a slightly different name like aws-fpga-master.

`source C:/<replace with install location>/aws-fpga/hdk/common/verif/scripts/hdk_initsh.tcl`

Before installing the DDR4 models, another critical warning appeared dealing with the SH\_CL\_BB\_routed.dcp.  Download the DCP into the following install location from a web browser.  Note aws-fpga could have a slightly different name like aws-fpga-master.

`https://s3.amazonaws.com/aws-fpga-hdk-resources/hdk/shell_v04261818/build/checkpoints/from_aws/SH_CL_BB_routed.dcp`

`C:/<replace with install location>/aws-fpga/hdk/common/shell_v04261818/build/checkpoints/from_aws/`

Download the following executable from a web browser to the following HLx directory.  Note aws-fpga could have a slightly different name like aws-fpga-master.

`http://www.labtestproject.com/files/sha256sum/sha256sum.exe`

`C:/<replace with install location>/aws-fpga/hdk/common/shell_v04261818/hlx/build/scripts`

Close Vivado and launch Vivado, the HLx environment is now setup and will always be sourced and IP integrator features will be automatically loaded.

Amazon CLI for Windows can be used for access to S3 to upload .tar and ingestion flow.


<a name="vivado"></a>
# Vivado Overview

이 섹션은 Vivado GUI에 대한 기본 개요입니다.  GUI 환경에서는 모든 경험 수준의 개발자가 설계 요구 사항을 충족하는 프로젝트 옵션과 전략을 신속하게 설정할 수 있으며, 대화형 보고서 및 설계 보기를 통해 시기 또는 영역과 관련된 문제를 신속하게 해결할 수 있습니다.

IP 인티그레이터는 Vivado HLx 디자인 스위트의 디자인 입력 툴입니다.  이 툴을 사용하면 개발자가 블록 수준에서 IP를 연결하고 "보이는 그대로" RTL(Register Transfer Language) 파일을 VHDL 또는 Verilog 형식으로 생성할 수 있습니다.  IP 통합기 플로우는 표준 RTL 플로우를 개선하고 개발자에게 다음과 같은 디자이너 지원 기능에 대한 액세스를 제공합니다:


- 인터페이스 기반 연결을 통한 IP 연결 간소화

- IP의 구성에 따라 인터커넥트, DMA 또는 기타 지원 블록과 같은 헬퍼 IP를 추가하는 블록 자동화

- 블록 간에 인터페이스, 클록 및 리셋을 라우팅하는 연결 자동화

- 적절한 인터페이스 연결 및 클록 도메인 크로싱을 보장하는 DRC(설계 규칙 검사)

- 개발자가 트랜잭션 수준에서 디버깅할 수 있는 고급 하드웨어 디버그 기능


자세한 정보 및 방법론 설계 가이드라인은 다음 문서를 참조하세요:

- <a href="https://www.xilinx.com/support/documentation/sw_manuals/xilinx2017_2/ug892-vivado-design-flows-overview.pdf">ug892-vivado-design-flows-overview.pdf</a>
- <a href="https://www.xilinx.com/support/documentation/sw_manuals/xilinx2017_2/ug994-vivado-ip-subsystems.pdf">ug994-vivado-ip-subsystems.pdf</a>
- <a href="https://www.xilinx.com/support/documentation/sw_manuals/xilinx2017_2/ug949-vivado-design-methodology.pdf">ug949-vivado-design-methodology.pdf</a>


GUI를 열려면 `vivado` 명령을 실행하세요.  비바도가 로드되면 새 프로젝트 만들기를 선택하고 빈 캔버스가 표시될 때까지 메뉴를 진행하여 빈 프로젝트를 만들 수 있습니다.  아래 섹션에서는 탭/메뉴에 대해 설명하고 아래 스크린샷을 참조하세요.  빈 프로젝트를 사용하여 탭과 메뉴를 자유롭게 사용해 보세요.

![Diagram](./images/hlx/vivado_gui.jpg)

## Sources Tab

노란색 상자에는 디자인 소스가 들어 있습니다.

### Sources:Hierarchy Tab

소스는 세 가지 범주로 나뉩니다.

1. 디자인 소스 폴더는 합성/구현 소스를 위한 폴더입니다.
2. 컨스트레인트 폴더는 타이밍 컨스트레인트(XDC)를 위한 폴더입니다.
3. 시뮬레이션 소스 폴더는 시뮬레이션 전용 소스입니다.

특정 파일을 클릭하면 속성 탭(소스 아래)에 정보가 표시됩니다.  속성 탭에서 개발자는 설계 흐름에서 파일이 사용되는 방식을 변경할 수 있습니다.

RTL/IP 소스의 경우 합성/구현/시뮬레이션 또는 합성/구현 및/또는 시뮬레이션 전용으로 표시할 수 있습니다.  XDC는 합성/구현 또는 합성 전용 또는 구현 전용으로 표시할 수 있습니다.

FPGA 개발자 AMI에는 테스트 예제가 포함되어 있습니다.  설명된 대로 소스 파일 속성을 자세히 살펴보려면 더하기(+)를 클릭하여 /home/centos/src/test/counter/Sources/hdl/counter.v를 추가하고 설계 소스를 추가합니다.

### Sources:IP Sources

When IP has been created in your project, the "IP Sources" tab will be visible.  This tab will contain imported  IP sources and expanding the IP/Instantiation Template, developers can add the IP into the RTL.  Synthesis options on the IP should be global only.

## Flow Navigator

The Flow Navigator is in the green box and can be used to launch predefined design flow steps, such as synthesis and implementation.

### PROJECT MANAGER

PROJECT MANAGER section allows to Add Sources like RTL/IP/XDC sources, Language Templates for common RTL constructs/XDCs/DEBUG, and IP Catalog to add IPs to the project.  This portion of the flow targets the RTL flow.

When invoking IP Catalog, the developers can search for a particular IP or look through the different categories of IP and it’s the responsibility of the developer to add and connect the IP into the developer's RTL.


### IP INTEGRATOR

This section allows the developer to open and modify the Block Design and the Generate Block Design after the design is validated.  The framework of the Block Design with the AWS IP and board are already created with the HLx flow so Create Block Design isn’t necessary.

Double clicking on any IP in the BD brings up the Re-customize IP Dialog Box where IP settings can be reviewed or modified.  When connecting designs, Connection Automation is available to automatically connect interfaces.

### SIMULATION

This section allows the developer to change simulation settings by right clicking on SIMULATION and invoking simulations by clicking Run Simulation->Run Behavioral Simulation.

### RTL ANALYSIS

By clicking on Open Elaborate Design, the RTL files in the design are analyzed where the developer can check RTL structures and syntax before the synthesis stage.

### SYNTHESIS

By right clicking on SYNTHESIS, the developer is able to view Synthesis Settings and Launch Synthesis.  After synthesis stage is complete, clicking on Open Synthesized Design will open the post synthesis checkpoint for analysis.  This stage is necessary for developing timing constraints for the CL.

### IMPLEMENTATION

By right clicking on IMPLEMENTATION, the developer is able to view Implementation Settings and Launch Implementation.  After implementation stage is complete, clicking on Open Implementation Design will open the post implementation checkpoint for analysis of the SH/CL.

## TCL Commands

The orange box is where TCL commands are entered.  The TCL Console Tab above the orange box reports the output of the TCL command.

## Design Runs Tab

The blue box is where the Design Runs are located with similar functionality as the Flow Navigator/SYNTHESIS and Flow Navigator/IMPLEMENTATION sections. The examples and tutorials mention how to use synth\_1 and impl\_1 to build the design.


<a name="projover"></a>
# Vivado Flows Overview

The Vivado HLx environment supports IP Integrator, RTL, HLS flows in Vivado and this section will discuss these
flows from a top level. For more details about each flow take a look at [IPI_GUI_Flows](./IPI_GUI_Flows.md)


## IP + RTL flow

Developers can add in IPs, existing AWS RTL from examples, new AWS template CL files from the AWS RTL flow and custom generated Verilog/System Verilog/VHDL files as source files.

For using IPs, at this time global mode is only supported and Out of context (OOC) is not supported at this time.

![Diagram](./images/hlx/rtl_dram_dma_sources.jpg)

Refer to the section [IPI_GUI_Flow_rtl_project_with_example_design](./IPI_GUI_Flows.md#rtl-project-with-example-design)
for more details about RTL flow using example RTL design.
#### Eg:
An example can be found at [IPI_RTL_example_flow](../../hdk/docs/IPI_GUI_Examples.md#rtlexistut_world).


Refer to the section [IPI_GUI_Flow_rtl_project_with_example_design](./IPI_GUI_Flows.md#rtlnew)
for more details about RTL flow using RTL design from scratch.
#### Eg:
An example can be found at [IPI_RTL_scratch_flow](./IPI_GUI_Examples.md#rtlscrtut)


## IP Integration flow

Developers can add in Vivado IP into the block diagram to create/stitch a full design easily. RTL module referencing flow using developer RTL can be used to add custom IP to the block diagram. Both RTL and IP can be added as IP in this flow.

![Diagram](./images/hlx/ipi_mod_ref.jpg)

Refer to the section [IPI_GUI_flow](./IPI_GUI_Flows.md#ipiprojex) for more details about IPI GUI flow.
#### Eg:
An example can be found at [IPI_Integrator_flow](../../hdk/docs/IPI_GUI_Examples.md#rtlex).

## IP Integrator for HLS IP

HLS is a C programming method in order to create RTL/IP for the FPGA Developers. Developers can add developed/generated HLS IPs in either RTL/IP Integrator flows using an IP Repository. In this flow only IPs are used to create examples.

![Diagram](./images/hlx/ipi_hls.jpg)

Eg:

A link to the example will be added here in the future when we have HLS C example ready.

## General Environment

### Design Constraints in Project

Timing analysis and setting timing constraints/floorplans is discussed in the timing closure documentation.

The following top level clocks from the Shell (MMCM in the Shell to the CL) are generated dynamically based upon clock recipe’s used with the AWS flow.
The developer can’t modify these constraints as they are dynamically created before synthesis.

cl\_clocks\_aws.xdc – Top level clock constraints for the CL.

The following .xdc file is only available with the RTL flow provided by the AWS env for synthesis.  This file should be disabled in the vivado project
if DDR4 memories are not in the CL design(critical warnings could show up).

cl\_synth\_aws.xdc - Timing constraints between sh_ddr module and DDR4 IP.

The following .xdc files are available to the developer.

cl\_synth\_user.xdc – Timing constraints in the CL(I.E creating new clock structures with clock generator/using clocks in different Shell MMCM).

cl\_pnr\_user.xdc – Timing constraints between the CL/SH.  Floorplanning is done in this xdc if necessary.



### Synthesis/Implementation

Timing analysis and setting appropriate synthesis settings/implementation directives are discussed in the timing closure documentation.

By default, synthesis is using Default directive and -max\_uram\_cascade\_height is set to 1. The developer can set max\_uram\_cascade\_height in the More Options section of Design Run Settings Tab by right clicking on Synth\_1->Open Run.

By default, all implementation steps are using the Explore directive.

If needing to change implementation settings, only change the tool directives in the Design Run Settings tab by right clicking on Impl_1->Change Run Settings… .
Change only directives for opt_design ,place_design, phys_opt_design, and route_design.  Do not change Strategy options as this overrides certain options in the HLx environment at this time.

Refer to [IPI Tutorials and Examples](./IPI_GUI_Examples.md) to get started.

### HLx button (optional)

The HLx button is a custom command that can be added to Vivado toolbar for ease of use. Here is an example: [Vivado IP Integrator video](https://www.xilinx.com/video/hardware/using-vivado-ip-integrator-and-amazon-f1.html)

To add it to Vivado toolbar, please follow the steps once VIvado GUI is opened:
- Go to Tools -> Custom Commands -> Customize Commands...

![HLx_step_1](./images/hlx/HLx_button_step_1.jpg)
- Click on the "+" icon
- Enter a name (e.g. AWS_Make_IPI) and press Enter
- Configure the window as follow

![HLx_step_2](./images/hlx/HLx_button_step_2.jpg)
- Finally, press OK

You can now use the HLx button.

<a name="summary"></a>
# Summary

Now that you are familar with building an [customer examples](../README.md) on F1 (AFI) and running on F1 using the CLI/TCL method; this guide has helped you setup Vivado for IP Integrator, please move to the [IP Integrator Tutorials and Examples](./IPI_GUI_Examples.md).  This documentation will help you get started on example designs, new designs, and additional tutorials.



