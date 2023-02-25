
# Using AWS XDMA in C/C++ application

## Table of Content

1. [개요](#오버뷰)
2. [빠른 예제](#quickExample)
3. [상세 설명](#detailed)
    - [파일 설명자](#fd)
    - [설정 및 해제](#openclose)
    - [쓰기 호출](#write)
    - [읽기 호출](#read)
    - [폴링 호출](#poll)
    - [동시성, 멀티 스레딩](#concurrency)
    - [오류 처리](#error)
4. [자주 묻는 질문](#faqs)

<a name="overview"></a>
# Overview

XDMA 드라이버는 FPGA와 인스턴스의 CPU 메모리 간에 데이터를 전송하기 위한 옵션으로 제공됩니다.

XDMA 드라이버 목표: <br />
1. 오픈 소스 드라이버 <br />
2. 설치 및 사용이 간편하며 드라이버 개발 전문 지식이 필요하지 않습니다. <br />
3. 고성능을 위한 멀티 채널 인터페이스 <br />
4. 인스턴스당 여러 개의 FPGA 및 FPGA당 여러 채널 지원 <br />

XDMA 드라이버 소스 코드는 AWS FPGA HDK 및 SDK와 함께 배포됩니다.

[XDMA 설치 가이드](./xdma_install.md)는 XDMA 설치 컴파일, 설치 및 문제 해결 방법에 대한 자세한 지침을 제공합니다.

**참고: XDMA 사용은 필수가 아닙니다. AWS는 CPU와 FPGA 간의 직접 통신을 위해 메모리 매핑된 PCIe 주소 공간을 제공합니다.**

다양한 CPU-FPGA 통신 옵션과 사용 가능한 다양한 옵션에 대한 자세한 설명은 [프로그래머 뷰](../../../hdk/docs/Programmer_View.md)를 참조하세요.

<a name="quickExample"></a>
# Quick Example

XDMA의 세부 사양을 살펴보기 전에 개발자가 XDMA를 사용하는 방법에 대한 짧고 직관적인 예제를 살펴보겠습니다:

아래 프로그램은 표준 Linux 시스템 호출 `open()`을 사용하여 파일 기술자(fd)를 생성하고, 한 쌍의 XDMA 채널(하나는 `read()`용, 다른 하나는 `write()`용)에 매핑합니다.  XDMA 하드웨어 엔진의 이름은 `XDMA Core`입니다.  XDMA 쓰기 채널은 H2C(호스트 대 코어)라고 합니다.  XDMA 읽기 채널은 C2H(Core to Host)라고 합니다. 코어는 FPGA를, 호스트는 인스턴스 CPU를 의미합니다.


```C
#include <stdlib.h>
#include <stdio.h>
#include <fcntl.h>
#include <errno.h>
#include <unistd.h>

#define BUF_SIZE              256
#define OFFSET_IN_FPGA_DRAM   0x10000000

static char *rand_str(char *str, size_t size)  // randomize a string of size <size>
{
    const char charset[] = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRTSUVWXYZ1234567890";
    int i;

    for(i = 0; i < size; i++) {
        int key = rand() % (int) (sizeof charset - 1);
        str[i] = charset[key];
    }

    str[size-1] = '\0';
    return str;
}


int main()
{
    char* srcBuf;
    char* dstBuf;
    int read_fd;
    int write_fd;
    int i;
    int ret;

    srcBuf = (char*)malloc(BUF_SIZE * sizeof(char));
    dstBuf = (char*)malloc(BUF_SIZE * sizeof(char));

    /* Initialize srcBuf */
    rand_str(srcBuf, BUF_SIZE);

    /* XDMA 쓰기 채널 열기(호스트에서 코어로) */
    if ((write_fd = open("/dev/xdma0_h2c_0",O_WRONLY)) == -1) {
        perror("open failed with errno");
    }
    
    /* XDMA 읽기 채널 열기(코어-호스트 간) */
    if ((read_fd = open("/dev/xdma0_c2h_0",O_RDONLY)) == -1) {
        perror("open failed with errno");
    }

    /* 전체 소스 버퍼를 오프셋 OFFSET_IN_FPGA_DRAM에 기록합니다. */
    ret = pwrite(write_fd , srcBuf, BUF_SIZE, OFFSET_IN_FPGA_DRAM);
    if (ret < 0) {
        perror("write failed with errno");
    }
    
    printf("Tried to write %u bytes, succeeded in writing %u bytes\n", BUF_SIZE, ret);

    ret = pread(read_fd, dstBuf, BUF_SIZE, OFFSET_IN_FPGA_DRAM);
    if (ret < 0) {
        perror("read failed with errno");
    }
    
    printf("Tried reading %u byte, succeeded in read %u bytes\n", BUF_SIZE, ret);

    if (close(write_fd) < 0) {
        perror("write_fd close failed with errno");
    }
    if (close(read_fd) < 0) {
        perror("read_fd close failed with errno");
    }

    printf("Data read is %s\n", dstBuf);
    
    return 0;
} 
```

<a name="detailed"></a>
# 상세 설명

<a name="fd"></a>
## 파일 작업을 사용하여 DMA 수행

XDMA 드라이버는 표준 Linux/POSIX 시스템 호출에 따른 간단한 장치 조작을 사용하여 모든 사용자 공간 프로그램에서 사용할 수 있습니다.  각 XDMA 채널에는 H2C 및 C2H 장치 파일 이름(예: `/dev/xdma0_h2c_0` 및 `/dev/xdma0_c2h_0`)이 있으며, Linux 문자 장치 API를 지원합니다.

XDMA 데이터 이동 명령(예: `pread()` 및 `pwrite()`)은 인스턴스 CPU 메모리에 대한 버퍼 포인터 `void*`를 사용하며, 파일 오프셋 `off_t`를 사용하여 FPGA에 쓰기/읽기 주소를 제시합니다.

**참고:** EC2 F1 인스턴스에서 파일 오프셋은 AppPF BAR4 128GB 주소 공간을 기준으로 FPGA의 쓰기-투-읽기 주소를 나타냅니다. DMA는 다른 PCIe BAR 공간에 액세스할 수 없습니다. [FPGA PCIe 메모리 주소 맵](../../../hdk/docs/AWS_Fpga_Pcie_Memory_Map.md)을 참조하십시오.


<a name="openclose"></a>
## 초기화 및 해체 API

초기화는 표준 파일 열기 API를 사용하여 수행됩니다:
`int open(const char *pathname, int flags);`

여기서 파일 이름은 H2C 또는 C2H 장치 파일 중 하나입니다(예: `/dev/xdmaX_h2c_Y` 또는 `/dev/xdmaX_c2h_Y`). (X는 FPGA 슬롯, Y는 특정 채널), 해당 H2C 또는 C2H 채널 방향에 대해 `O_WRONLY` 또는 `O_RDONLY` 플래그만 권장됩니다.  다른 모든 플래그는 무시됩니다.

여러 스레드 또는 프로세스가 동일한 파일을 열 수 있으며, 조정/직렬화를 보장하는 것은 개발자의 책임입니다.

해당 `close()`는 DMA 채널을 해제하는 데 사용됩니다.

<a name="write"></a>
## Write APIs

쓰기를 위한 두 가지 표준 Linux/POSIX API는 다음과 같습니다:

***ssize_t write(int fd, void\* buf, size_t count)*** 

***ssize_t pwrite(int fd, void\* buf, size_t count, off_t offset)***   (Recommended, see [explanation](#seek))

`쓰기()/쓰기()`를 호출하기 전에 파일 기술자(fd)가 성공적으로 열렸어야 합니다.

FPGA에 쓰기 위한 소스 버퍼에 대한 포인터인 `buf`는 임의의 크기와 정렬을 가질 수 있습니다.

XDMA 드라이버는 `buf` 메모리 범위를 하드웨어 DMA가 사용할 수 있는 물리적 주소 목록에 매핑하는 작업을 담당합니다. 

XDMA 드라이버는 사용자 공간 `buf` 메모리를 고정하여 DMA 전송 중에 스왑 아웃되지 않도록 처리합니다.

<a name="read"></a>
## Read APIs 

***ssize_t read(int fd, void\* buf, size_t count)*** 

***ssize_t pread(int fd, void\* buf, size_t count, off_t offset)***   (Recommended, see [explaination](#seek))

`read()`와 `pread()`는 모두 호출을 차단하고 있으며, 호출은 데이터가 반환될 때까지 대기합니다.

읽기는 성공한 바이트 수를 반환하며, 반환값이 카운트와 같지 않은 경우 올바른 오프셋으로 `read()`를 다시 호출하는 것은 사용자의 책임입니다. DMA 타임아웃(10초)이 발생하면 EIO가 반환됩니다. 

Possible errors:<br />
EIO - DMA timeout or transaction failure.<br />
ENOMEM - System is out of memory.<br />

**NOTE:** 앞서 언급한 오류 중 하나라도 발생하면 FPGA 및 XDMA는 알 수 없는 상태로 남게 되며, Linux `dmesg` 로그에서 오류에 대한 자세한 정보를 얻을 수 있습니다(FAQ: XDMA에 오류가 발생했는지 확인하려면 어떻게 해야 하나요? 참조).

<a name="seek"></a>
## Seek API

XDMA 드라이버는 문자 장치 파일 위치를 수정하는 표준 `lseek()` Linux/POSIX 시스템 호출을 구현합니다. 이 위치는 `read()`/`write()`에서 FPGA 메모리 공간을 가리키는 데 사용됩니다. 

**WARNING:** 동시/멀티 스레드 설계에서 `lseek()`을 `read()/write()`로 원자적 후속 조치 없이 동시에 호출할 수 있으므로 적절한 잠금 없이 `lseek()`을 호출하면 오류가 발생하기 쉽습니다.

file_pos는 파일 속성이므로 `write()` 및 `read()` 연산에 의해 성공적으로 쓰거나 읽은 바이트 수만큼 증가합니다.

**개발자는 원자적인 방식으로 lseek 및 쓰기/읽기를 수행하는 `pwrite()` 및 `pread()`를 사용하는 것이 좋습니다.**

<a name="poll"></a>
## Poll API

poll()` 함수는 일치하는 사용자 이벤트에 대해 파일 기술자 집합을 통해 입력을 다중화하는 메커니즘을 애플리케이션에 제공합니다. 이 함수는 사용자가 생성한 인터럽트 이벤트에 대해 XDMA 드라이버에서 사용되며 데이터 전송에는 사용되지 않습니다.

폴린 마스크만 지원되며 이벤트가 발생했음을 알리는 데 사용됩니다.

자세한 내용은 [사용자 정의 인터럽트 이벤트 README](./user_defined_interrupts_README.md)를 참조하세요.

애플리케이션은 향후 사용자 인터럽트에 대한 알림을 받으려면 준비된 파일 기술자의 `pread`를 실행하여 XDMA 드라이버 내에서 `events_irq` 변수를 반환하고 지워야 합니다.  사용자 정의 인터럽트에 `poll` 및 `pread`를 사용하는 예는 [test_dram_dma.c](../../../hdk/cl/examples/cl_dram_dma/software/runtime/test_dram_dma.c) `interrupt_example()` 내에 나와 있습니다.

<a name="concurrency"></a>
## Concurrency and Multi-Threading

XDMA는 여러 프로세스에서 동시 다중 액세스와 한 프로세스 내에서 여러 스레드를 지원합니다.  여러 프로세스가 동일한 파일 기술자에 대해 `open()/close()`를 호출할 수 있습니다.

서로 다른 스레드/프로세스에서 동일한 메모리 영역에 대한 쓰기가 중복되지 않고 조정되도록 하는 것은 개발자의 책임입니다.

다시 말씀드리면, `lseek()` + `read()/write()` 시퀀스보다 `pread()/pwrite()`를 사용하는 것이 좋습니다.

<a name="error"></a>
## Error Handling

드라이버는 일부 오류 사례를 처리하고 다른 오류는 사용자에게 전달합니다.

XDMA는 오류, 특히 FPGA의 커스텀 로직 부분에서 애플리케이션 충돌 또는 버그가 발생했을 때 정상적으로 복구되도록 설계되었습니다. 이 설계는 알려진 모든 경우를 다루려고 시도하지만 복구할 수 없는 코너 케이스가 있을 수 있습니다. XDMA 드라이버는 복구할 수 없는 오류를 나타내는 오류를 Linux `dmesg` 서비스에 출력합니다(FAQ: XDMA에서 오류가 발생했는지 확인하려면 어떻게 해야 하나요? 참조).

#### Error: Application Process Crash 

사용자 공간 응용 프로그램에서 충돌이 발생하면 운영 체제 커널은 프로세스와 관련된 모든 열린 파일 기술자(XDMA 채널)를 분해합니다. 열려 있는 모든 파일 기술자에 대해 릴리스(`close()`와 동일)가 호출됩니다.  기내 DMA 읽기 또는 쓰기가 중단되고 Linux `dmesg`에 오류가 보고됩니다. FPGA 자체와 XDMA 드라이버는 알 수 없는 상태로 남아있을 수 있습니다(FAQ: XDMA에 오류가 발생했는지 확인하려면 어떻게 해야 하나요? 참조).

#### Error: API Time-out

시간 초과 오류는 다음과 같은 몇 가지 위치에서 발생할 수 있습니다:

1. `write()/pwrite()`에서 응용 프로그램이 멈췄습니다.

2. FPGA의 CL 부분에서 `read()`가 멈춰서 read()가 영원히 차단되는 경우.

XDMA 드라이버는 이 경우(10초)에 대한 타임아웃 메커니즘을 가지고 있으며, DMA 전송 중단 처리를 자동으로 트리거하고 앞서 언급한 "응용 프로그램 프로세스 충돌"에서 설명한 것과 동일한 절차에 따릅니다.

<a name="faqs"></a>
# FAQ

**Q: XDMA 드라이버의 소스 코드를 가져와서 컴파일하려면 어떻게 해야 하나요?**

XDMA 드라이버는 [AWS FPGA HDK/SDK](.)에 포함되어 있으며, 일부 Amazon Linux 배포판에는 사전 설치되어 있을 수 있습니다.

자세한 내용은 [설치 가이드](./xdma_install.md)를 참조하세요.

**Q: XDMA 드라이버로 사용 가능한 FPGA를 검색하는 방법은 무엇입니까?**

XDMA 드라이버가 실행되면 사용 가능한 모든 장치가 /dev 디렉터리에 /dev/xdmaX로 나열됩니다.

    `$ ls /dev/xdma*`
    
각 XDMA 장치는 `/dev/xdmaX_h2c_Y` 및 `/dev/xdmaX_c2h_Y` 아래에 여러 채널을 노출하며, 개발자는 이러한 문자 장치로 직접 작업할 수 있습니다.

**Q: `write()`/`pwrite()` 호출이 반환되면 데이터가 FPGA에 도달했다는 보장이 있나요?** 

아니요. `write()`/`pwrite()`에 대한 시스템 호출은 쓰거나 읽은 바이트 수를 반환합니다. 작업이 완료되지 않은 경우 다시 호출하는 것은 호출자의 몫입니다.

**Q: XDMA는 데이터를 삭제할 수 있나요?**

정상 작동 중에 XDMA는 데이터를 삭제하지 않습니다.

XDMA가 데이터를 삭제하는 두 가지 경우는 다음과 같습니다:
1. 이 채널을 관리하는 사용자 프로세스의 갑작스러운 충돌
2. XDMA 전송 중 시간 초과.
  
  
**Q: `read()`/`pread()` 또는 `write()`/`pwrite()`가 시간 초과되나요?**

`read()` 또는 `write` 함수가 -1을 반환하면 오류가 발생한 것이며, 이 오류는 errno 의사 변수에 보고됩니다.

**Q: XDMA에 오류가 발생했는지 어떻게 확인하나요?**

XDMA는 표준 Linux dmesg 서비스를 통해 로그를 출력합니다.

` $ dmesg | grep “xdma” `

**Q: XDMA는 데이터 전송 중에 인터럽트를 사용하나요?**

XDMA 커널 드라이버는 MSI-X 인터럽트를 사용할 수 있으며, 하나의 인터럽트 쌍이 XDMA 읽기/쓰기 채널에 사용됩니다.
XDMA에 사용되는 IRQ 번호를 확인하려면 다음을 수행할 수 있습니다. 

` $ cat /proc/interrupts | grep xdma`

**Q: XDMA는 데이터 전송을 위해 폴링된 DMA 디스크립터 완성 모드를 지원하나요?**
 
XDMA는 인터럽트 및 폴링된 DMA 디스크립터 완성 모드를 모두 지원합니다.  

인터럽트 모드와 폴링 모드의 처리 단계를 축약하여 차이점을 보여줍니다:

Interupt mode:

1.) DMA 디스크립터를 FPGA HW에 전송하고, 2.) wait_event_interruptible_timeout을 통해 호출 스레드를 절전 상태로 전환하고, 3.) 컨텍스트 전환 후 XDMA 인터럽트 서비스 루틴 내에서 DMA 완료 인터럽트를 처리하고 2의 스레드를 깨우고, 4.) 다시 2의 스레드로 컨텍스트 전환하여 DMA IO를 정리할 수 있도록 합니다.

Polled mode:

1.) DMA 디스크립터를 FPGA HW에 전송하고, 2.) 디스크립터 완료를 나타내기 위해 FPGA HW가 업데이트하는 DMA 코히어런트 쓰기백 버퍼를 효율적으로 폴링한 다음 DMA IO를 정리합니다.  쓰기백 버퍼를 폴링하는 스레드는 100회 반복마다 Linux 스케줄러에 제어권을 다시 릴리스하여 vCPU를 사용하지 않도록 하며 타임아웃 기능도 포함하고 있습니다. 
  
처리 단계 및 스레드 컨텍스트 전환의 감소로 인해 폴링된 DMA 기술자 완료 모드는 더 작은 IO 크기(예: 1MB 미만)를 사용하는 특정 애플리케이션 사용 사례에서 인터럽트 모드에 비해 상당한 성능 이점이 있을 수 있습니다.  개발자는 인터럽트 모드와 폴링된 DMA 기술자 완성 모드를 모두 실험해 보고 애플리케이션에 가장 적합한 모드를 사용하는 것이 좋습니다.

XDMA 폴링된 DMA 디스크립터 완성 모드는 XDMA 드라이버 로드 시 활성화됩니다:

` $ insmod xdma.ko poll_mode=1`

XDMA 인터럽트 DMA 디스크립터 완료 모드도 XDMA 드라이버 로드 시 활성화됩니다(기본값):

` $ insmod xdma.ko`
