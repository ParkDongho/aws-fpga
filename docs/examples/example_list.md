## Example Applications List

| Accelerator Application | Example | Development Environment | Description |
| --------|---------|---------|-------|
| Custom hardware | [cl\_hello\_world](../../hdk/cl/examples/cl_hello_world) | HDK - RTL (Verilog) | 최소한의 하드웨어를 사용한 간단한 [시작 예제](../../hdk/README.md) |
| Custom hardware | [cl\_dram\_dma](../../hdk/cl/examples/cl_dram_dma) | HDK - RTL (Verilog) | F1 셸에 대한 CL 연결 및 모든 DDR과의 연결을 시연합니다. |
| Custom hardware | [IP integration example using a GUI - cl\_dram\_dma\_hlx](../../hdk/cl/examples/cl_dram_dma_hlx) | HLx - Verilog  | F1 셸에 대한 CL 연결 및 Vivado IP 통합기 GUI를 사용하여 DRAM에 대한 연결/연결을 시연합니다. |
| Custom hardware | [Virtual Ethernet Application](../../sdk/apps/virtual-ethernet) | [Streaming Data Engine](../../hdk/cl/examples/cl_sde) | 가상 이더넷 프레임워크는 네트워크 인터페이스(또는 모든 소스)에서 FPGA로 이더넷 프레임을 스트리밍하여 처리한 후 특정 대상으로 다시 내보낼 수 있도록 지원합니다. 이를 위한 사용 사례로는 심층 패킷 검사, 소프트웨어 정의 네트워킹, 스트림 암호화 또는 압축 등이 있습니다. |
| Custom hardware | [Pipelined Workload Applications - cl\_dram\_dma\_data\_retention](../../hdk/docs/data_retention.md)| [HDK](../../hdk/cl/examples/cl_dram_dma/software/runtime/test_dram_dma_retention.c) [SDAccel](../../SDAccel/examples/aws/data_retention) | 가속기를 교체하는 동안 DRAM의 데이터를 보존하는 방법을 시연합니다. 임시 가속기 파이프라인을 사용하는 애플리케이션은 이 기능을 활용하여 FPGA 이미지 스왑 간의 지연 시간을 줄일 수 있습니다.  |
| High Level Synthesis | [Digital Up-Converter - cl\_hls\_dds\_hlx](../../hdk/cl/examples/cl_hls_dds_hlx) | HLx - C-to-RTL  | RTL(Verilog)로 합성된 C로 작성된 예제 애플리케이션을 시연합니다. |
| Custom Hardware with Software Defined Acceleration | [RTL Kernels](https://github.com/Xilinx/Vitis_Accel_Examples/tree/master/rtl_kernels) | Vitis - RTL(Verilog) + C/C++/OpenCL | 이 예제는 소프트웨어 정의 워크플로에서 새로운 하드웨어 설계(RTL)를 개발하는 방법을 보여줍니다.|
## Application Notes 

App Note | Description |
|---------|---------|
| [Using PCIe Peer-2-Peer connectivity](https://github.com/awslabs/aws-fpga-app-notes/tree/master/Using-PCIe-Peer2Peer) | 이 앱 노트는 F1.16XL 인스턴스에서 PCIe P2P 연결을 사용하는 방법을 보여줍니다. |
| [Using PCIM Port](https://github.com/awslabs/aws-fpga-app-notes/tree/master/Using-PCIM-Port) | 이 앱 노트는 카드와 호스트 메모리 간에 데이터를 전송하기 위해 PCIM AXI 포트를 사용하는 방법을 보여줍니다. |
| [Using PCIe User Interrupts](https://github.com/awslabs/aws-fpga-app-notes/tree/master/Using-PCIe-Interrupts) | 이 앱 노트에서는 개발자가 커스텀 인터럽트 서비스 루틴(ISR)을 작성하는 데 필요한 기본 커널 호출에 대해 설명하고 이러한 호출을 보여주는 예제를 제공합니다. |
| [Using PCIe Write Combining](https://github.com/awslabs/aws-fpga-app-notes/tree/master/Using-PCIe-Write-Combining) | 이 앱 노트에서는 F1 가속기용 소프트웨어에서 쓰기 결합을 사용하는 시기와 쓰기 결합을 활용하는 방법을 설명합니다. |

## Workshops

* [ReInvent:19 Workshop](https://github.com/awslabs/aws-fpga-app-notes/tree/master/reInvent19_Developer_Workshop)
* [ReInvent:18 Workshop](https://github.com/awslabs/aws-fpga-app-notes/tree/master/reInvent18_Developer_Workshop)
