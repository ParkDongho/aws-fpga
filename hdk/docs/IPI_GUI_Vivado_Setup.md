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

프로젝트에서 IP가 생성되면 "IP 소스" 탭이 표시됩니다.  이 탭에는 임포트한 IP 소스가 포함되며, 개발자는 IP/인스턴스화 템플릿을 확장하여 해당 IP를 RTL에 추가할 수 있습니다.  IP의 합성 옵션은 글로벌 옵션만 사용해야 합니다.

## Flow Navigator

흐름 탐색기는 녹색 상자에 있으며 합성 및 구현과 같이 미리 정의된 디자인 흐름 단계를 시작하는 데 사용할 수 있습니다.

### PROJECT MANAGER

프로젝트 관리자 섹션에서는 RTL/IP/XDC 소스와 같은 소스 추가, 일반적인 RTL 구성/XDC/디버그용 언어 템플릿, 프로젝트에 IP를 추가할 수 있는 IP 카탈로그 등을 사용할 수 있습니다.  흐름의 이 부분은 RTL 흐름을 대상으로 합니다.

IP 카탈로그를 호출할 때 개발자는 특정 IP를 검색하거나 다양한 범주의 IP를 살펴볼 수 있으며, 해당 IP를 개발자의 RTL에 추가하고 연결하는 것은 개발자의 책임입니다.


### IP INTEGRATOR

이 섹션에서는 개발자가 블록 디자인을 열고 수정할 수 있으며, 디자인이 검증되면 블록 디자인 생성도 가능합니다.  AWS IP 및 보드가 포함된 블록 디자인의 프레임워크는 HLx 플로우로 이미 생성되어 있으므로 블록 디자인 생성은 필요하지 않습니다.

BD에서 IP를 더블 클릭하면 IP 설정을 검토하거나 수정할 수 있는 IP 재사용 대화 상자가 나타납니다.  설계를 연결할 때 연결 자동화를 사용하면 인터페이스를 자동으로 연결할 수 있습니다.

### SIMULATION

이 섹션에서 개발자는 시뮬레이션을 마우스 오른쪽 버튼으로 클릭하고 시뮬레이션 실행->행동 시뮬레이션 실행을 클릭하여 시뮬레이션을 호출하여 시뮬레이션 설정을 변경할 수 있습니다.

### RTL ANALYSIS

Open Elaborate Design을 클릭하면 설계의 RTL 파일이 분석되어 개발자가 합성 단계 전에 RTL 구조와 구문을 확인할 수 있습니다.

### SYNTHESIS

개발자는 합성을 마우스 오른쪽 버튼으로 클릭하면 합성 설정 및 합성 시작을 볼 수 있습니다.  합성 단계가 완료된 후 합성된 디자인 열기를 클릭하면 분석을 위한 합성 후 체크포인트가 열립니다.  이 단계는 CL의 타이밍 제약 조건을 개발하는 데 필요합니다.

### IMPLEMENTATION

구현을 마우스 오른쪽 버튼으로 클릭하면 개발자는 구현 설정 및 구현 시작을 볼 수 있습니다.  구현 단계가 완료된 후 구현 디자인 열기를 클릭하면 SH/CL 분석을 위한 구현 후 체크포인트가 열립니다.

## TCL Commands

주황색 상자는 TCL 명령을 입력하는 곳입니다.  주황색 상자 위의 TCL 콘솔 탭은 TCL 명령의 출력을 보고합니다.

## Design Runs Tab

파란색 상자는 흐름 탐색기/합성 및 흐름 탐색기/구현 섹션과 유사한 기능을 가진 디자인 실행이 있는 곳입니다. 예제 및 튜토리얼에서는 synth\_1 및 impl\_1을 사용하여 디자인을 빌드하는 방법에 대해 설명합니다.


<a name="projover"></a>
# Vivado Flows Overview

Vivado HLx 환경은 Vivado에서 IP 통합기, RTL, HLS 플로우를 지원하며 이 섹션에서는 이러한 플로우를
흐름에 대해 설명합니다. 각 플로우에 대한 자세한 내용은 [IPI_GUI_Flows](./IPI_GUI_Flows.md)를 참조하세요.


## IP + RTL flow

개발자는 IP, 예제에서 제공하는 기존 AWS RTL, AWS RTL 플로우에서 제공하는 새로운 AWS 템플릿 CL 파일, 사용자 지정 생성된 Verilog/System Verilog/VHDL 파일을 소스 파일로 추가할 수 있습니다.

IP 사용의 경우 현재 글로벌 모드만 지원되며, OOC(Out of Context)는 현재 지원되지 않습니다.

![Diagram](./images/hlx/rtl_dram_dma_sources.jpg)

예제 디자인을 사용한 RTL 흐름에 대한 자세한 내용은 [IPI_GUI_Flow_rtl_project_with_example_design](./IPI_GUI_Flows.md#rtl-project-with-example-design) 섹션을 참조하세요.
를 참조하여 예제 RTL 디자인을 사용한 RTL 흐름에 대한 자세한 내용을 확인하세요.
#### Eg:
예제는 [IPI_RTL_example_flow](../../hdk/docs/IPI_GUI_Examples.md#rtlexistut_world)에서 찾을 수 있습니다.


자세한 내용은 [IPI_GUI_Flow_rtl_project_with_example_design](./IPI_GUI_Flows.md#rtlnew) 섹션을 참조하세요.
섹션을 참조하여 RTL 디자인을 처음부터 사용하는 RTL 흐름에 대해 자세히 알아보세요.
#### Eg:
예제는 [IPI_RTL_scratch_flow](./IPI_GUI_Exples.md#rtlscrtut)에서 확인할 수 있습니다.


## IP Integration flow

개발자는 블록 다이어그램에 Vivado IP를 추가하여 전체 설계를 쉽게 생성/접합할 수 있습니다. 개발자 RTL을 사용하는 RTL 모듈 레퍼런스 플로우를 사용하여 블록 다이어그램에 커스텀 IP를 추가할 수 있습니다. 이 플로우에서는 RTL과 IP를 모두 IP로 추가할 수 있습니다.

![Diagram](./images/hlx/ipi_mod_ref.jpg)

IPI GUI 플로우에 대한 자세한 내용은 [IPI_GUI_flow](./IPI_GUI_Flows.md#ipiprojex) 섹션을 참조하세요.
#### Eg:
예제는 [IPI_Integrator_flow](../../hdk/docs/IPI_GUI_Examples.md#rtlex)에서 확인할 수 있습니다.

## IP Integrator for HLS IP

HLS는 FPGA 개발자를 위한 RTL/IP를 생성하기 위한 C 프로그래밍 방법입니다. 개발자는 IP 리포지토리를 사용하여 개발/생성된 HLS IP를 RTL/IP 통합기 플로우에 추가할 수 있습니다. 이 플로우에서는 예제 생성에 IP만 사용됩니다.

![Diagram](./images/hlx/ipi_hls.jpg)

Eg:

향후 HLS C 예제가 준비되면 여기에 예제 링크가 추가될 예정입니다.

## General Environment

### Design Constraints in Project

타이밍 분석 및 타이밍 제약 조건/플로어플랜 설정은 타이밍 클로저 문서에서 설명합니다.

셸의 다음 최상위 클럭(셸에서 CL까지)은 AWS 흐름에 사용된 클럭 레시피에 따라 동적으로 생성됩니다.
이러한 제약 조건은 합성 전에 동적으로 생성되므로 개발자가 수정할 수 없습니다.

cl\_clocks\_aws.xdc - CL에 대한 최상위 클럭 제약 조건입니다.

다음 .xdc 파일은 합성을 위해 AWS 환경이 제공하는 RTL 흐름에서만 사용할 수 있습니다.  이 파일은 비바도 프로젝트에서 비활성화해야 합니다.
에서 비활성화해야 합니다(중요 경고가 표시될 수 있음).

cl\_synth\_aws.xdc - sh_ddr 모듈과 DDR4 IP 간의 타이밍 제약 조건.

개발자는 다음 .xdc 파일을 사용할 수 있습니다.

cl\_synth\_user.xdc - CL의 타이밍 제약 조건(예: 클럭 제너레이터로 새 클럭 구조 생성/다른 셸 MMCM에서 클럭 사용).

cl\_pnr\_user.xdc - CL/SH 간의 타이밍 제약 조건.  필요한 경우 이 xdc에서 플로어 플래닝이 수행됩니다.



### Synthesis/Implementation

타이밍 분석과 적절한 합성 설정/구현 지시어 설정은 타이밍 클로저 문서에 설명되어 있습니다.

기본적으로 합성은 Default 지시문을 사용하며 -max\_uram\_cascade\_height는 1로 설정되어 있습니다. 개발자는 Synth\_1->Open Run을 마우스 오른쪽 버튼으로 클릭하여 디자인 실행 설정 탭의 추가 옵션 섹션에서 max\_uram\_cascade\_height를 설정할 수 있습니다.

기본적으로 모든 구현 단계는 탐색 지시문을 사용합니다.

구현 설정을 변경해야 하는 경우 Impl_1->실행 설정 변경...을 마우스 오른쪽 버튼으로 클릭하여 디자인 실행 설정 탭에서 도구 지시어만 변경합니다.
opt_design, place_design, phys_opt_design, route_design에 대한 지시어만 변경합니다.  전략 옵션은 현재 HLx 환경의 특정 옵션을 재정의하므로 변경하지 마십시오.

시작하려면 [IPI 튜토리얼 및 예제](./IPI_GUI_Examples.md)를 참조하세요.

### HLx button (optional)

HLx 버튼은 사용하기 쉽도록 Vivado 도구 모음에 추가할 수 있는 사용자 지정 명령입니다. 다음은 예제입니다: [비바도 IP 통합기 동영상](https://www.xilinx.com/video/hardware/using-vivado-ip-integrator-and-amazon-f1.html)

비바도 툴바에 추가하려면 비바도 GUI가 열리면 다음 단계를 따르세요:
- Go to Tools -> Custom Commands -> Customize Commands...

![HLx_step_1](./images/hlx/HLx_button_step_1.jpg)
- "+" 아이콘을 클릭합니다.
- 이름(예: AWS_Make_IPI)을 입력하고 Enter 키를 누릅니다.
- 다음과 같이 창을 구성합니다.

![HLx_step_2](./images/hlx/HLx_button_step_2.jpg)
- 마지막으로 OK를 누릅니다.

이제 HLx 버튼을 사용할 수 있습니다.

<a name="summary"></a>
# Summary

이제 F1(AFI)에서 [고객 예제](../README.md)를 빌드하고 CLI/TCL 방법을 사용하여 F1에서 실행하는 데 익숙해졌으므로 이 가이드가 IP 통합기용 Vivado 설정에 도움이 되었으므로 [IP 통합기 튜토리얼 및 예제](./IPI_GUI_Exples.md)로 이동하십시오.  이 문서는 예제 디자인, 새로운 디자인 및 추가 튜토리얼을 시작하는 데 도움이 될 것입니다.



