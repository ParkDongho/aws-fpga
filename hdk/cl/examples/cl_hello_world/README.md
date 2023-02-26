# Hello World CL Example


## **⚠️ 참고:** F1을 처음 사용하는 경우 [CL 예제 중 하나에서 Amazon FPGA 이미지 (AFI)를 만드는 방법 : 단계별 가이드](./../../../README.md)를 먼저 읽어주세요!!

## Table of Contents

1. [Overview](#overview)
2. [Functional Description](#description)
3. [Hello World Example Metadata](#metadata)


<a name="overview"></a>
## Overview

이 *hello_world* 예제는 인스턴스가 커스텀 로직(CL)에서 레지스터를 "peek" 및 "poke"할 수 있도록 하는 커스텀 로직(CL)을 빌드합니다.
이 예시는 가상 LED 및 가상 DIP 스위치의 기본 사용 사례를 보여줍니다.

<a name="description"></a>
## Functional Description

cl_hello_world 예제는 기본적인 셸-CL 연결, 메모리 매핑 레지스터 인스턴스화, 가상 LED 및 DIP 스위치 사용을 보여줍니다.
cl_hello_world 예제에서는 OCL AXI-L 인터페이스에 연결된 [FPGA AppPF BAR0 메모리 공간](../../../docs/AWS_Fpga_Pcie_Memory_Map.md)에 두 개의 레지스터를 구현합니다. 
두 레지스터는 다음과 같습니다:

1. Hello World Register (offset 0x500)
2. Virtual LED Register (offset 0x504)

Hello World 레지스터는 32비트 읽기/쓰기 레지스터입니다. 그러나 레지스터가 올바르게 액세스되고 있음을 보여주기 위해 레지스터에 대해 반환되는 읽기 데이터는 바이트 스왑됩니다.

가상 LED 레지스터는 헬로 월드 레지스터의 하위 16비트를 섀도 처리하여 헬로 월드 레지스터의 15:0 비트와 동일한 값을 보유하도록 하는 16비트 읽기 전용 레지스터입니다.

cl_hello_world 설계는 [cl_ports.vh](./../../../common/shell_stable/design/interfaces/cl_ports.vh) 파일에 설명된 두 개의 신호로 구성된 가상 LED 및 DIP 스위치 인터페이스를 활용합니다:


```
   input[15:0] sh_cl_status_vdip,               //가상 DIP 스위치.  FPGA 관리 PF 및 툴을 통해 제어됩니다.
   output logic[15:0] cl_sh_status_vled,        //FPGA 관리 PF 및 툴을 통해 모니터링되는 가상 LED
```

이 예제에서 가상 LED 레지스터는 가상 LED 신호인 cl_sh_status_vled를 구동하는 데 사용됩니다. 
또한 가상 DIP 스위치인 sh_cl_status_vdip은 가상 LED로 전송되는 가상 LED 레지스터 값을 게이팅하는 데 사용됩니다. 
예를 들어, sh_cl_status_vdip을 16'h00FF로 설정하면 가상 LED 레지스터의 하위 8비트만 가상 LED 신호 cl_sh_status_vled에 신호가 전달됩니다. 

F1에서 실행하는 동안 개발자는 FPGA 툴 `fpga-get-virtual-led`를 사용하여 CL-to-Shell 인터페이스에서 LED 값을 읽을 수 있습니다. 
반면 `fpga-set-virtual-dip-switch` 툴은 셸-CL 인터페이스의 DIP 스위치 값을 설정하는 데 사용됩니다.

  
### Unused interfaces

Hello World 예제에서는 대부분의 AWS 셸 인터페이스를 사용하지 않으므로 사용하지 않는 신호가 묶여 있습니다.
`cl_hello_world.sv` 파일의 끝에는 사용하지 않는 모든 인터페이스에 대한 타이오프를 처리하기 위해 인터페이스별 `.inc` 파일에 대한 특정 `include` 명령이 있습니다.


<a name="metadata"></a>
## Hello World Example Metadata

다음 표에는 AWS에 AFI로 등록하는 데 필요한 CL에 대한 정보가 표시되어 있습니다.
또는 이 CL에 대해 미리 생성된 AFI를 직접 사용할 수도 있습니다.


| Key   | Value     |
|-----------|------|
| Shell Version | 0x04261818 |
| PCI Device ID | 0xF000 |
| PCI Vendor ID | 0x1D0F (Amazon) |
| PCI Subsystem ID | 0x1D51 |
| PCI Subsystem Vendor ID | 0xFEDD |
| Pre-generated AFI ID | afi-03d11a4ea66e883ef |
| Pre-generated AGFI ID | agfi-0fcf87119b8e97bf3 |

