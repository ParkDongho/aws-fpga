# AWS FPGA PCIe Memory Map

FPGA는 AWS EC2 인스턴스에 PCIe로 연결되며, 각 FPGA 슬롯은 [AWS 셸 사양](./AWS_Shell_Interface_Specification.md)에 정의된 대로 각각 여러 PCIe 기본 주소 레지스터(BAR)를 가진 두 개의 PCIe 물리적 기능(PF)을 가진 단일 FPGA를 제공합니다.

이 문서에서는 각 BAR의 실제 크기와 속성에 대해 설명하며, 실제 애플리케이션에서 매핑하는 방법에 대한 몇 가지 예시를 제공합니다.

이러한 모든 PCIe BAR는 EC2 인스턴스 메모리 매핑 I/O(MMIO) 공간에 매핑되어 있지만, 액세스하기 전에 Linux 커널 또는 사용자 공간 애플리케이션에 매핑해야 합니다. 다양한 소프트웨어가 FPGA PCIe 메모리와 상호 작용하는 방법은 [소프트웨어 프로그래머 뷰](./Programmer_View.md)를 참조하세요.

## Memory map per Slot
```
--- FPGA Slot X  
  |----- AppPF  
  |   |------- BAR0  
  |   |         * 32-bit BAR, non-prefetchable
  |   |         * 32MiB (0 to 0x1FF-FFFF)
  |   |         * Maps to OCL AXI-L of the CL
  |   |         * Typically used for CL application registers or OpenCL kernels  
  |   |------- BAR1
  |   |         * 32-bit BAR, non-prefetchable
  |   |         * 2MiB (0 to 0x1F-FFFF)
  |   |         * Maps to BAR1 AXI-L of the CL
  |   |         * Typically used for CL application registers 
  |   |------- BAR2
  |   |         * 64-bit BAR, prefetchable
  |   |         * 64KiB (0 to 0xFFFF)
  |   |         * NOT exposed to CL, used by internal DMA inside the Shell
  |   |------- BAR4
  |             * 64-bit BAR, prefetchable
  |             * 128GiB (0 to 0x1F-FFFF-FFFF)
  |             * First 127GiB are exposed to CL, via pcis_dma AXI bus
  |             * The upper 1GiB is reserved for future use
  |
  |
  |----- MgmtPF  
     |------- BAR0  
     |         * 64-bit BAR, prefetchable
     |         * 16KiB (0 to 0x3FFF)
     |         * Maps to internal functions used by the FPGA management tools
     |         * Not mapped to CL
     |------- BAR2
     |         * 64-bit BAR, prefetchable
     |         * 16KiB (0 to 0x3FFF)
     |         * Maps to internal functions used by the FPGA management tools
     |         * Not mapped to CL
     |------- BAR4
               * 64-bit BAR, prefetchable
               * 4MiB (0 to 0x3FFFFF)
               * Maps to CL through SDA AXI-L
               * Could be used by Developer applications, or if using the AWS Runtime Environment (like SDAccel case), it will be used for performance monitoring.
```
