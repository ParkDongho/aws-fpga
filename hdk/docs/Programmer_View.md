# AWS FPGA: 커스텀 로직에 대한 프로그래머의 관점

이 문서는 Linux 사용자 공간에서 실행되는 애플리케이션이 런타임 중에 FPGA 사용자 정의 로직(CL)과 인터페이스하는 방법을 설명합니다.

AWS FPGA를 사용하는 데 필요한 두 가지 부분은 관리와 런타임이며, 다음 그림은 이러한 구성 요소와 기본 FPGA 하드웨어와의 관계에 대한 개략적인 보기를 제공합니다.

![alt tag](./images/AWS_FPGA_Software_Overview.jpg)

1. **관리 인터페이스**: AFI 로드/지우기, AFI 상태 확인, AFI 디버그, 에뮬레이트 LED 및 에뮬레이트 DIP 스위치에 필요합니다. 관리 인터페이스는 세 가지 방식 중 하나로 제공되며, 하나 이상을 동시에 사용할 수 있습니다:

- **\[A\]** 리눅스 셸 명령으로 [FPGA 관리 도구](../../sdk/userspace/fpga_mgmt_tools/README.md)를 실행합니다.
  
- **\[B\]** 개발자의 C/C++ 애플리케이션과 함께 컴파일할 수 있는 [FPGA 관리 라이브러리](../../sdk/userspace/fpga_libs/fpga_mgmt/)라는 C-라이브러리로 제공됩니다.
  
- **\[C\]** OpenCL 런타임 라이브러리](../../Vitis)와 사전 통합됨
  
2. **Runtime code**: 커스텀 로직에서 읽기/쓰기, 인터럽트 처리, DMA 사용에 필요합니다. 이것은 다음에 의해 제공됩니다:
  
- **\[D\]** [FPGA PCIe Lib](../../sdk/userspace/fpga_libs/fpga_pci/)는 읽기/쓰기, 레지스터 공간 또는 메시지 전달과 같은 Linux 애플리케이션 공간에서 AppPF PCIe BAR 뒤의 FPGA 메모리 공간에 액세스하는 데 사용되는 C-라이브러리입니다. 이 라이브러리는 컴파일하여 개발자의 C/C++ 애플리케이션과 연결할 수 있습니다.
  
- **\[E\]** [DMA 인터페이스](../../sdk/linux_kernel_drivers/xdma/README.md)(open()/read()/write()와 같은 표준 POSIX API를 사용하는)는 모든 C/C++ 애플리케이션에서 DMA를 사용하여 데이터를 전송하는 데 사용할 수 있습니다. 이 DMA 인터페이스를 사용하려면 **\[G\]** 항목으로 표시된 [XDMA 커널 드라이버](../../sdk/linux_kernel_drivers/xdma/xdma_install.md)를 설치해야 합니다.
  
- **\[F\]** 모든 C/C++ 애플리케이션에서 사용할 수 있도록 open() 및 poll()과 같은 표준 POSIX API를 사용하는 [사용자 공간 인터럽트/이벤트 알림](../../sdk/linux_kernel_drivers/xdma/user_defined_interrupts_README.md)입니다. 이 인터럽트/이벤트 인터페이스를 사용하려면 **\[G\]** 항목으로 표시된 [XDMA 커널 드라이버](../../sdk/linux_kernel_drivers/xdma/xdma_install.md)를 설치해야 합니다.
  
- **\[I\]** Xilinx Vitis에서 생성된 것과 같은 openCL 런타임 애플리케이션과 연결되는 openCL ICD 라이브러리입니다. 
  
  
  

