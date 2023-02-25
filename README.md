# Table of Contents

1. [Overview of AWS EC2 FPGA Development Kit](#overview-of-aws-ec2-fpga-development-kit)
    - [Developer Support](#developer-support)
    - [Development Flow](#development-flow)
    - [Development environments](#development-environments)
    - [FPGA Developer AMI](#fpga-developer-ami)
    - [FPGA Hardware Development Kit (HDK)](#hardware-development-kit-hdk)
    - [FPGA Software Development Kit (SDK)](#runtime-tools-sdk)
    - [Software Defined Development Environment](#software-defined-development-environment)
1. [Amazon EC2 F1 platform features](#amazon-ec2-f1-platform-features)
1. [Getting Started](#getting-started)
    - [Getting Familiar with AWS](#getting-familiar-with-aws)
    - [First time setup](#setting-up-development-environment-for-the-first-time)
    - [Quickstarts](#quickstarts)
    - [How To's](#how-tos)
1. [Documentation Overview](#documentation-overview)

# Overview of AWS EC2 FPGA Development Kit

AWS EC2 FPGA 개발 키트는 [Amazon EC2 F1 인스턴스](https://aws.amazon.com/ec2/instance-types/f1/)에서 하드웨어 가속 애플리케이션을 개발, 시뮬레이션, 디버그, 컴파일 및 실행할 수 있는 개발 및 런타임 도구 세트입니다.
이 github 리포지토리와 AWS에서 제공하는 [Centos](https://aws.amazon.com/marketplace/pp/B06VVYBLZZ)/[AL2](https://aws.amazon.com/marketplace/pp/B08NTMMZ7X)의 FPGA 개발자 AMI를 통해 개발 도구 비용 없이 배포됩니다.

⚠️ <b>NOTE:</b> 개발자 키트는 Linux 운영 체제에서만 지원됩니다.

## Developer Support

AWS FPGA 개발 키트에 대한 지원을 받으려면 [GitHub 이슈](https://github.com/aws/aws-fpga/issues)를 개설하는 것이 가장 좋습니다. 또한, [FPGA 개발 re:Post Tag](https://repost.aws/tags/TAc7ofO5tbQRO57aX1lBYbjA/fpga-development)를 통해 AWS 고객 커뮤니티, AWS 고객 지원 및 AWS FPGA 개발 팀에서 FPGA 관련 토론 주제를 찾을 수 있습니다.

## Development Flow
개발자는 FPGA 설계(CL - 사용자 정의 로직이라고도 함)를 생성한 후 Amazon FPGA 이미지(AFI)를 생성하고 이를 F1 인스턴스에 쉽게 배포할 수 있습니다. AFI는 재사용 및 공유가 가능하며 확장 가능하고 안전한 방식으로 배포할 수 있습니다.

![Alt text](hdk/docs/images/f1-Instance-How-it-Works-flowchart.jpg)

## Development Environments

| Development Environment | Description | Accelerator Language | Hardware Interface | Debug Options| Typical Developer                                                     |
| --------|---------|-------|---------|-------|-----------------------------------------------------------------------|
| [Vitis](Vitis/README.md)/[SDAccel](SDAccel/README.md)를 이용한 소프트웨어 정의 가속기 개발 | 개발 경험은 최적화된 컴파일러를 활용하여 새로운 가속기를 쉽게 개발하거나 기존 C/C++/openCL, Verilog/VHDL을 AWS FPGA 인스턴스로 마이그레이션할 수 있도록 지원합니다. | C/C++/OpenCL, Verilog/VHDL (RTL) | OpenCL APIs and XRT | SW/HW 에뮬레이션, 시뮬레이션, GDB, 가상 JTAG(칩스코프) | FPGA 경험이 전혀 없는 SW 또는 HW 개발자                          |
| [비바도를 사용한 하드웨어 가속기 개발](hdk/README.md) | 완전 맞춤형 하드웨어 개발 경험은 하드웨어 개발자에게 AWS FPGA 인스턴스용 AFI를 개발하는 데 필요한 도구를 제공합니다.  | Verilog/VHDL | [XDMA Driver](sdk/linux_kernel_drivers/xdma/README.md), [peek/poke](sdk/userspace/README.md) | Simulation, Virtual JTAG | 고급 FPGA 경험이 있는 HW 개발자                            |
| [IP Integrator/High Level Design(HLx) using Vivado](hdk/docs/IPI_GUI_Vivado_Setup.md) | IP 및 고수준 합성 개발 통합을 위한 그래픽 인터페이스 개발 경험 | Verilog/VHDL/C | [XDMA Driver](sdk/linux_kernel_drivers/xdma/README.md), [peek/poke](sdk/userspace/README.md) | Simulation, Virtual JTAG | 중급 FPGA 경험이 있는 HW 개발자                        |
 | [On-premise development for Alveo U200 using Vitis targetted for migration to F1](Vitis/docs/Alveo_to_AWS_F1_Migration.md) | F1으로의 마이그레이션을 목표로 온프레미스 U200 플랫폼을 사용한 바이티스 플로우 개발 |  C/C++/OpenCL, Verilog/VHDL (RTL) | OpenCL APIs and XRT | SW/HW 에뮬레이션, 시뮬레이션, GDB, JTAG(칩스코프) | FPGA 경험이 없고 온프레미스 U200 카드가 없는 SW 또는 HW 개발자 |
 | [On-premise development for Alveo U200 using F1.A.1.4 shell](hdk/docs/U200_to_F1_migration_HDK.md) | F1으로의 마이그레이션을 대상으로 하는 F1.A.1.4 셸을 사용하는 온프레미스 U200 카드용 HDK 흐름 | Verilog/VHDL | XDMA driver, peek/poke | Simulation, JTAG | 고급 FPGA 및 온프레미스 U200 카드 경험이 있는 HW 개발자   |
> on-premise 개발의 경우, SDAccel/Vitis/Vivado는 [올바른 라이선스가 있어야 하며 지원되는 도구 버전 중 하나를 사용](./docs/on_premise_licensing_help.md)해야 합니다.

## FPGA Developer AMI

[FPGA 개발자 AMI](https://aws.amazon.com/marketplace/pp/B06VVYBLZZ)는 AWS 마켓플레이스에서 소프트웨어 비용 없이 사용할 수 있으며, AWS F1에서 실행할 FPGA 설계를 개발하는 데 필요한 툴이 포함되어 있습니다. 

AWS F1 인스턴스 내부에 사용되는 FPGA의 크기가 크기 때문에, 자일링스 툴은 32GiB 메모리에서 가장 잘 작동합니다. 
z1d.xlarge/C5.4xlarge 및 z1d.2xlarge/C5.8xlarge 인스턴스 유형은 각각 30GiB+ 및 60GiB+의 메모리로 가장 빠른 실행 시간을 제공합니다. 
비용을 절감하고자 하는 개발자는 t2.2xlarge와 같은 저비용 인스턴스에서 코딩 및 시뮬레이션을 시작한 후 앞서 언급한 대형 인스턴스로 이동하여 가속 코드 합성을 실행할 수 있습니다.

AWS 마켓플레이스는 여러 버전의 FPGA 개발자 AMI를 제공합니다. 다음 섹션 표에서는 현재 지원되는 개발자 키트 버전과 AMI 버전의 매핑에 대해 설명합니다.

## Xilinx tool support

| 개발자 키트 버전          | 지원 도구 버전            | 호환 가능한 FPGA 개발자 AMI 버전                  |
|-----------------------|------------------------|---------------------------------------------|
| 1.4.23+               | 2021.2                 | v1.12.X (Xilinx Vivado/Vitis 2021.2)        |
| 1.4.21+               | 2021.1                 | v1.11.X (Xilinx Vivado/Vitis 2021.1)        |
| 1.4.18+               | 2020.2                 | v1.10.X (Xilinx Vivado/Vitis 2020.2)        |
| 1.4.16+               | 2020.1                 | v1.9.0-v1.9.X (Xilinx Vivado/Vitis 2020.1)  |
| 1.4.13+               | 2019.2                 | v1.8.0-v1.8.X (Xilinx Vivado/Vitis 2019.2)  |
| 1.4.11+               | 2019.1                 | v1.7.0-v1.7.X (Xilinx Vivado/SDx 2019.1)    |
| 1.4.8 - 1.4.15b       | 2018.3                 | v1.6.0-v1.6.X (Xilinx Vivado/SDx 2018.3)    |
| 1.4.3 - 1.4.15b       | 2018.2                 | v1.5.0-v1.5.X (Xilinx Vivado/SDx 2018.2)    |
| ⚠️ 1.3.7 - 1.4.15b    | 2017.4                 | v1.4.0-v1.4.X (Xilinx Vivado/SDx 2017.4) ⚠️ |

⚠️ 개발자 키트 릴리스 v1.4.16에서는 Xilinx 2017.4, 2018.2, 2018.3 툴셋에 대한 지원이 제거됩니다. 개발자 키트 릴리스 v1.4.16 이후에는 이전 Xilinx 툴을 지원하지 않지만, HDK 릴리스 v1.4.15b 이하에서는 계속 사용할 수 있습니다. 
자일링스 2017.4, 2018.2, 2018.3 툴셋을 사용하려면 [Github의 최신 v1.4.15b 릴리스 태그](https://github.com/aws/aws-fpga/releases/tag/v1.4.15b)를 확인하세요.

지원 중단 공지는 [지원 중단 공지](./README.md#end-of-life-announcements)에서 확인하시기 바랍니다.

소프트웨어 정의 개발의 경우 사용 중인 자일링스 툴셋에 따른 런타임 호환성 표를 참조하시기 바랍니다:
[SDAccel](SDAccel/docs/Create_Runtime_AMI.md#runtime-ami-compatibility-table) 또는 [Vitis](Vitis/docs/Create_Runtime_AMI.md#runtime-ami-compatibility-table)에서 확인할 수 있습니다.

### End of life Announcements

| Xilinx Tool version | State | Statement                                                                                                                                                                 | 
|-----------|-----------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 2017.1 | 🚫 Deprecated on 09/01/2018 | Developer kit versions prior to v1.3.7 and Developer AMI prior to v1.4 (2017.1) [reached end-of-life](https://forums.aws.amazon.com/ann.jspa?annID=6068).                 |
| 2017.4 | 🚫 Deprecated on 12/31/2021 | [Support for Xilinx 2017.4 toolsets was deprecated on 12/31/2021](https://forums.aws.amazon.com/ann.jspa?annID=8949). |
| 2020.1 and below | Discontinued on 02/2022 | Removed the ability for customers to newly subscribe to 2020.1 and below AMI versions to remove exposure to [CVE-2021-44228](https://www.cve.org/CVERecord?id=CVE-2021-44228) as these versions of tools do not have patches from xilinx |

## Hardware Development Kit (HDK)

[HDK 디렉터리](./hdk/README.md)에는 설명서, 예제, 시뮬레이션, 빌드 및 AFI 생성 스크립트가 포함되어 있어 Amazon FPGA 이미지(AFI) 빌드를 시작할 수 있습니다.  
HDK는 모든 온프레미스 서버 또는 EC2 인스턴스에 설치할 수 있습니다. 
다른 개발자가 공유한 사전 빌드된 AFI를 사용하려는 경우에는 개발자 키트가 필요하지 않습니다.

### AWS Shells

Amazon EC2 FPGA 인스턴스에서는 각 FPGA가 두 개의 파티션으로 나뉩니다:

* 셸(SH) - FPGA 외부 주변 장치, PCIe, DRAM 및 인터럽트를 구현하는 AWS 플랫폼 로직.

* 커스텀 로직(CL) - FPGA 개발자가 만든 커스텀 가속 로직.

개발 프로세스가 끝나면 셸과 CL을 결합하여 Amazon EC2 FPGA 인스턴스에 로드할 수 있는 Amazon FPGA 이미지(AFI)를 생성합니다.

다음 표는 현재 CL을 개발하는 데 사용할 수 있는 셸을 제공합니다. 각 셸은 특정 인터페이스와 기능을 제공하며 현재 표에 나열된 개발 키트 브랜치와 함께 사용해야 합니다.

| Shell Name| Shell Version | Dev Kit Branch | Description|
|--------|--------|---------|-------|
| F1 XDMA Shell | F1.X.1.4 | [master](https://github.com/aws/aws-fpga/) |. DMA를 포함하는, 여기에 나열된 모든 [인터페이스](https://github.com/aws/aws-fpga/blob/master/hdk/docs/AWS_Shell_Interface_Specification.md)를 제공합니다. | 
| F1 Small Shell | F1.S.1.0 | [small_shell](https://github.com/aws/aws-fpga/tree/small_shell) | 여기에 나열된 모든 [인터페이스](https://github.com/aws/aws-fpga/blob/small_shell/hdk/docs/AWS_Shell_Interface_Specification.md)를 제공합니다. 이 셸에는 DMA 엔진이 포함되어 있지 않으며 셸 리소스 사용량을 크게 줄여줍니다. |

자세한 내용은 [FAQ](./FAQs.md#general-aws-fpga-shell-faqs)를 확인하세요.

## Software-defined Development Environment

소프트웨어 정의 개발 환경을 통해 고객은 C/C++/OpenCL 코드를 커널로 FPGA에 컴파일하고 OpenCL API를 사용하여 데이터를 FPGA에 전달할 수 있습니다. 
FPGA 경험이 없는 소프트웨어 개발자도 익숙한 개발 환경을 통해 클라우드 애플리케이션을 강화할 수 있습니다.

또한 이 개발 환경에서는 C/C++ 및 RTL 가속기 설계를 C/C++ 소프트웨어 기반 개발 환경으로 혼합할 수 있습니다. 이 방법을 사용하면 C/C++를 사용하여 더 빠르게 프로토타이핑하는 동시에 RTL 내에서 중요 블록의 수동 최적화를 지원할 수 있습니다. 이 접근 방식은 소프트웨어 컴파일러 최적화 방법을 사용하여 시간에 중요한 기능을 최적화하는 것과 유사합니다.

Xilinx SDAccel을 시작하려면 [소프트웨어 정의 개발 환경 설명서](SDAccel/README.md)를 검토하세요.
자일링스 Vitis를 시작하려면 [Vitis 통합 개발 환경 readme](Vitis/README.md)를 검토하세요.

## Runtime Tools (SDK)

[SDK 디렉토리](./sdk/README.md)에는 EC2 FPGA 인스턴스에서 실행하는 데 필요한 런타임 환경이 포함되어 있습니다. 여기에는 FPGA 인스턴스에 로드된 AFI를 관리하기 위한 드라이버와 도구가 포함되어 있습니다. SDK는 AFI 개발 프로세스 중에는 필요하지 않으며, AFI가 EC2 FPGA 인스턴스에 로드된 후에만 필요합니다. 다음과 같은 SDK 리소스가 제공됩니다:
  * Linux 커널 드라이버 - 개발자 키트에는 세 가지 드라이버가 포함되어 있습니다:
    * [XDMA 드라이버](sdk/linux_kernel_drivers/xdma/README.md) - HDK 가속기와 주고받는 DMA 인터페이스.
  * [FPGA 라이브러리](sdk/userspace/fpga_libs) - C/C++ 호스트 애플리케이션에서 사용하는 API.
  * [FPGA 관리 툴](sdk/userspace/fpga_mgmt_tools/README.md) - F1 인스턴스의 런타임 FPGA 이미지 로딩/지우기, 메트릭 수집 및 디버그 인터페이스를 위한 AFI 관리 API입니다.

# Amazon EC2 F1 Platform Features
* 1-8개의 자일링스 울트라스케일+ VU9P 기반 FPGA 슬롯
* FPGA 슬롯당, 커스텀 로직(CL)에 사용 가능한 인터페이스:
    * x16 PCIe 3세대 인터페이스 1개
    * 4개의 DDR4 RDIMM 인터페이스(72비트, ECC 포함, 각 16GiB, 총 64GiB)
    * 모든 인터페이스에서 AXI4 프로토콜 지원
* 모든 CL-셸 인터페이스를 구동하는 사용자 정의 클록 주파수
* 여러 개의 자유 실행 보조 클럭
* 커스텀 로직(CL)에 대한 PCI-E 엔드포인트 프레젠테이션
    * 관리 PF(물리적 기능)
    * 애플리케이션 PF
* 가상 JTAG, 가상 LED, 가상 DIP 스위치
* 쉘(SH)과 커스텀 로직(CL) 간 PCI-E 인터페이스.
    * SH에서 CL으로 512비트 AXI4 인바운드 인터페이스
    * CL에서 SH로의 512비트 AXI4 아웃바운드 인터페이스
    * 레지스터 액세스를 위한 여러 32비트 AXI-Lite 버스(다양한 PCIe BAR에 매핑)
    * 셸에서 설정한 최대 페이로드 크기
    * 셸에서 설정한 최대 읽기 요청 크기
    * AXI4 오류 처리 
* SH와 CL 간 DDR 인터페이스
    * CL과 SH 간 512비트 AXI4 인터페이스
    * SH에 구현된 1개의 DDR 컨트롤러(항상 사용 가능)
    * CL에 구현된 3개의 DDR 컨트롤러(구현된 컨트롤러 수 구성 가능)

# Getting Started

### Getting familiar with AWS
AWS를 사용해 본 적이 없다면 [AWS 시작하기 교육](https://aws.amazon.com/getting-started/)부터 시작하여 [AWS EC2](https://aws.amazon.com/ec2/) 및 [AWS S3](https://aws.amazon.com/s3/) 서비스의 기본 사항을 중점적으로 학습하는 것이 좋습니다. 
이러한 서비스의 기본 사항을 이해하면 AWS F1 및 FPGA 개발자 키트로 작업하기가 더 쉬워집니다.

FPGA Image generation and EC2 F1 instances are supported in the us-east-1 (N. Virginia), us-west-2 (Oregon), eu-west-1 (Ireland) and us-gov-west-1 ([GovCloud US](https://aws.amazon.com/govcloud-us/)) [regions](https://aws.amazon.com/about-aws/global-infrastructure/).

> ⚠️ <b>NOTE:</b> 기본적으로 AWS 계정의 EC2 F1 인스턴스 시작 제한은 0입니다. 
> F1 인스턴스를 사용하기 전에 [지원 케이스](https://console.aws.amazon.com/support/home#/case/create)를 열어 F1 인스턴스를 시작할 수 있도록 EC2 인스턴스 한도를 늘려야 합니다.

### Setting up development environment for the first time 

[FPGA 개발자 AMI](https://aws.amazon.com/marketplace/pp/B06VVYBLZZ) 또는 on-premise를 사용하여 AWS EC2에서 개발할 수 있습니다. 

> ℹ️ <b>INFO:</b> EC2에서 [빌드 인스턴스](#fpga-developer-ami)를 사용하여 FPGA 개발자 AMI로 시작하는 것이 좋으며, 여기에는 빠르게 개발에 착수할 수 있도록 Xilinx 툴 및 라이선스가 설정되어 있습니다.

> ℹ️ <b>INFO:</b> 온프레미스 개발의 경우 [사용할 수 있는 자일링스 툴 및 라이선스](./docs/on_premise_licensing_help.md)가 있어야 합니다.

1. 먼저 빌드 인스턴스를 시작하여 개발을 시작하세요. 
    > 💡 <b>TIP:</b> 이 인스턴스는 F1 인스턴스일 필요는 없습니다. F1 인스턴스는 디자인 빌드 및 AFI 생성 단계를 거친 후 AFI(Amazon FPGA 이미지)를 실행하는 데만 필요합니다.
    
    > ℹ️ <b>INFO:</b> GUI 개발 흐름을 따라야 하는 경우, GUI 데스크톱 설정에 대한 단계별 가이드를 제공하는 [개발자 리소스](./developer_resources/README.md)를 확인하십시오.
1. 인스턴스에서 [FPGA 개발자 키트]를 클론합니다.
    ```git clone https://github.com/aws/aws-fpga.git```
1. 다음 섹션의 빠른 시작을 따릅니다.

### Quickstarts
자체 AWS FPGA 설계를 생성하기 전에 단계별 빠른 시작 가이드 중 하나를 살펴볼 것을 권장합니다:

| Description | Quickstart | Next Steps |
|----|----|----|
| 자일링스 Vitis를 사용한 소프트웨어 정의 가속기 개발 | [Vitis hello_world Quickstart](Vitis/README.md) | [60+ Vitis examples](./Vitis/examples/), [Vitis Library Examples](./docs/examples/example_list.md) |
| 자일링스 SDAccel을 사용한 소프트웨어 정의 가속기 개발 | [SDAccel hello_world Quickstart](SDAccel/README.md) | [60+ SDAccel examples](./SDAccel/examples/) |
| 맞춤형 하드웨어 개발(HDK) | [HDK hello_world Quickstart](hdk/README.md) | [CL to Shell and DRAM connectivity example](./hdk/cl/examples/cl_dram_dma), [Virtual Ethernet Application](./sdk/apps/virtual-ethernet) using the [Streaming Data Engine](./hdk/cl/examples/cl_sde) |
| IP 통합기/하이 레벨 설계(HLx) | [IPI hello_world Quickstart](hdk/cl/examples/cl_hello_world_hlx/README.md) | [IPI GUI Examples](hdk/docs/IPI_GUI_Examples.md) |

ℹ️ <b>INFO:</b> 하이레벨 합성, 바이티스 라이브러리, 앱 노트 및 워크샵을 사용한 보다 심층적인 애플리케이션과 예제는 [예제 목록](./docs/exples/example_list.md)을 참조하세요.

### How Tos
| How To                                                                                | Description                                                                            | 
|---------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------|
| [Migrate Alveo U200 designs to F1 - Vitis](./Vitis/docs/Alveo_to_AWS_F1_Migration.md) | 이 애플리케이션 노트는 알베오 U200 디자인을 F1으로 쉽게 마이그레이션하는 방법을 보여줍니다.          | 
 | [Migrate Alveo U200 designs to F1 - HDK](./hdk/docs/U200_to_F1_migration_HDK.md)      | AWS 제공 셸을 사용하여 U200 비바도 디자인 플로우에서 F1 HDK 플로우로 마이그레이션하는 경로입니다. |                                                                 

# Documentation Overview

이 개발자 키트 전체에 걸쳐 문서가 제공되며, 아래 표에는 개발자가 정보를 찾는 데 도움이 되는 주요 문서 목록이 통합되어 있습니다:

| Topic | Document Name |  Description |
|-----------|-----------|------|
| AWS setup | [Setup AWS CLI and S3 Bucket](./SDAccel/docs/Setup_AWS_CLI_and_S3_Bucket.md) | AFI 생성 준비를 위한 설정 지침 |
| Developer Kit | [RELEASE NOTES](./RELEASE_NOTES.md), [Errata](./ERRATA.md) | 셸을 제외한 모든 개발자 키트 기능에 대한 릴리스 노트 및 오류 정보  |
| Developer Kit | [Errata](./ERRATA.md) | 셸을 제외한 모든 개발자 키트 기능에 대한 오류 수정 사항  |
| F1 Shell | [AWS Shell RELEASE NOTES](./hdk/docs/AWS_Shell_RELEASE_NOTES.md) | F1 셸의 릴리스 노트 |
| F1 Shell | [AWS Shell ERRATA](./hdk/docs/AWS_Shell_ERRATA.md) | Errata for F1 shell |
| F1 Shell | [AWS Shell Interface Specification](./hdk/docs/AWS_Shell_Interface_Specification.md) | AFI를 구축하는 HDK 개발자를 위한 Shell-CL 인터페이스 사양 |
| F1 Shell - Timeout and AXI Protocol Protection | [How to detect a shell timeout](hdk/docs/HOWTO_detect_shell_timeout.md) | 셸은 일정 시간이 지나거나 불법적인 트랜잭션이 발생하면 트랜잭션을 종료합니다.  시간 초과로 인한 CL 문제를 디버깅하는 데 도움이 되는 데이터를 감지하고 수집하는 방법을 설명합니다. |
| Vitis | [Debug Vitis Kernel](./Vitis/docs/Debug_Vitis_Kernel.md) | Vitis 커널 디버깅에 대한 지침 |
| Vitis | [Create Runtime AMI](./Vitis/docs/Create_Runtime_AMI.md) | Xilinx Vitis 사용 시 런타임 AMI 생성에 대한 지침|
| Vitis | [XRT Instructions](./Vitis/docs/XRT_installation_instructions.md) | F1을 위한 MPD 데몬 고려 사항이 포함된 XRT 빌드, 설치에 대한 지침 |
| SDAccel | [Debug RTL Kernel](./SDAccel/docs/Debug_RTL_Kernel.md) | SDAccel을 사용한 RTL 커널 디버깅에 대한 지침 |
| SDAccel | [Create Runtime AMI](./SDAccel/docs/Create_Runtime_AMI.md) | Xilinx SDAccel 사용 시 런타임 AMI 생성에 대한 지침 |
| HDK - Host Application | [Programmer View](./hdk/docs/Programmer_View.md) | CL 인터페이스 사양에 대한 호스트 애플리케이션 |
| HDK - CL Debug | [Debug using Virtual JTAG](./hdk/docs/Virtual_JTAG_XVC.md) | 가상 JTAG(칩스코프)를 사용하여 CL 디버깅하기  |
| HDK - Simulation | [Simulating CL Designs](./hdk/docs/RTL_Simulating_CL_Designs.md) | Shell-CL 시뮬레이션 사양 |
| HDK - Driver | [README](./sdk/linux_kernel_drivers/xdma/README.md) | HDK 예제에서 사용하는 DMA 드라이버(XDMA)를 설명하고 설치 가이드 링크를 포함합니다. |
| AFI | [AFI Management SDK](./sdk/userspace/fpga_mgmt_tools/README.md) | F1 인스턴스에서 AFI를 관리하기 위한 CLI 문서 |
| AFI - EC2 CLI | [copy\_fpga\_image](./hdk/docs/copy_fpga_image.md), [delete\_fpga\_image](./hdk/docs/delete_fpga_image.md), [describe\_fpga\_images](./hdk/docs/describe_fpga_images.md), [fpga\_image\_attributes](./hdk/docs/fpga_image_attributes.md) | AFI 관리를 위한 CLI 문서 |
| AFI - Creation Error Codes | [create\_fpga\_image\_error\_codes](hdk/docs/create_fpga_image_error_codes.md) | AFI 관리를 위한 CLI 문서 |
| AFI - Power | [FPGA Power, recovering from clock gating](./hdk/docs/afi_power.md) | 개발자가 FPGA 전력 사용량을 이해하고 F1 인스턴스의 전력 위반을 방지하며 클럭 게이트 슬롯에서 복구하는 데 도움을 줍니다. |
| On-premise Development | [Tools, Licenses required for on-premise development](./docs/on_premise_licensing_help.md) | [FPGA 개발자 AMI](https://aws.amazon.com/marketplace/pp/B06VVYBLZZ)를 사용하지 않고 온프레미스에서 AFI를 개발하려는 개발자를 위한 가이드 |
| Frequently asked questions | [FAQ](./FAQs.md)| 개발자 피드백과 일반적인 AWS 포럼 질문을 기반으로 Q/A가 추가됩니다.  |
