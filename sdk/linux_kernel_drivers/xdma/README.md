
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

** 참고: XDMA 사용은 필수가 아닙니다. AWS는 CPU와 FPGA 간의 직접 통신을 위해 메모리 매핑된 PCIe 주소 공간을 제공합니다. **

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
## Initialization and Tear Down API

Initialization is done using the standard file-open API:
`int open(const char *pathname, int flags);`

Where file name is one of the H2C or C2H device files (e.g. `/dev/xdmaX_h2c_Y` or `/dev/xdmaX_c2h_Y`). (X is the FPGA slot, Y is the specific channel), with the only flags recommended are `O_WRONLY` or `O_RDONLY` for the corresponding H2C or C2H channel direction.  All other flags are ignored.

Multiple threads or processes can open the same file, and it is the developer's responsibility to ensure coordination/serialization.

A corresponding `close()` is used to release the DMA channel.

<a name="write"></a>
## Write APIs

The two standard Linux/POSIX APIs for write are listed below:

***ssize_t write(int fd, void\* buf, size_t count)*** 

***ssize_t pwrite(int fd, void\* buf, size_t count, off_t offset)***   (Recommended, see [explanation](#seek))

The file-descriptor (fd) must have been opened successfully before calling `write()/pwrite()`.

`buf`, the pointer to the source buffer to write to FPGA can have arbitrary size and alignment.

The XDMA driver is responsible for mapping the `buf` memory range to list of physical addresses that the hardware DMA can use. 

The XDMA driver takes care of pinning the user-space `buf` memory so that it cannot be swapped out during the DMA transfer.

<a name="read"></a>
## Read APIs 

***ssize_t read(int fd, void\* buf, size_t count)*** 

***ssize_t pread(int fd, void\* buf, size_t count, off_t offset)***   (Recommended, see [explaination](#seek))

Both `read()` and `pread()` are blocking calls, and the call waits until data is returned.

Read returns the number of successful bytes, and it is the user responsibility to call `read()` with the correct offset again if the return value is not equal to count. In a case of DMA timeout (10 seconds), EIO is returned. 

Possible errors:<br />
EIO - DMA timeout or transaction failure.<br />
ENOMEM - System is out of memory.<br />

**NOTE:** In case of any of the aforementioned errors, the FPGA and XDMA is left in unknown state, with Linux `dmesg` log potentially providing more insight on the error (see FAQ: How would I check if XDMA encountered errors?).

<a name="seek"></a>
## Seek API

The XDMA driver implements the standard `lseek()` Linux/POSIX system call, which modifies the character device file position. The position is used in `read()`/`write()` to point the FPGA memory space. 

**WARNING: ** Calling `lseek()` without proper locking is prone to errors, as concurrent/multi-threaded design could call `lseek()` concurrently and without an atomic follow up with `read()/write()`.

The file_pos is a file attribute; therefore, it is incremented by both `write()` and `read()` operations by the number of bytes that were successfully written or read.

**Developers are encouraged to use `pwrite()` and `pread()`, which performs lseek and write/read in an atomic way**

<a name="poll"></a>
## Poll API

The `poll()` function provides applications with a mechanism for multiplexing input over a set of file descriptors for matching user events. This is used by the XDMA driver for user generated interrupts events, and not used for data transfers.

Only the POLLIN mask is supported and is used to notify that an event has occurred.

Refer to [User-defined interrupts events README](./user_defined_interrupts_README.md) for more details.

The application MUST issue a `pread` of the ready file descriptor to return and clear the `events_irq` variable within the XDMA driver in order to be notified of future user interrupts.  An example of using `poll` and `pread` for user defined interrupts is provided within the [test_dram_dma.c](../../../hdk/cl/examples/cl_dram_dma/software/runtime/test_dram_dma.c) `interrupt_example()`.

<a name="concurrency"></a>
## Concurrency and Multi-Threading

XDMA supports concurrent multiple access from multiple processes and multiple threads within one process.  Multiple processes can call `open()/close()` to the same file descriptor.

It is the developer's responsibility to make sure write to same memory region from different threads/processes is coordinated and not overlapping.

To re-iterate, use of `pread()/pwrite()` is recommended over a sequence of `lseek()` + `read()/write()`.

<a name="error"></a>
## Error Handling

The driver handles some error cases and passes other errors to the user.

XDMA is designed to attempt a graceful recovery from errors, specifically application crashes or bugs in the Custom Logic portion of the FPGA. While the design attempts to cover all known cases, there may be corner cases that are not recoverable. The XDMA driver prints errors to Linux `dmesg` service indicating an unrecoverable error  (see FAQ: How would I check if XDMA encountered errors?).

#### Error: Application Process Crash 

In case of a crash in the userspace application, the operating system kernel tears down of all open file descriptors (XDMA channels) associated with the process. Release (equivalent of `close()`) is called for every open file descriptor.  In-flight DMA reads or writes are aborted and an error is reported in Linux `dmesg`. The FPGA itself and the XDMA driver may be left in an unknown state (see FAQ: How would I check if XDMA encountered errors?).

#### Error: API Time-out

Timeout errors can occur in few places including:

1. Application stuck on `write()/pwrite()`.

2. A `read()` from CL portion of the FPGA that is stuck, causing the read() to block forever.

The XDMA driver has a timeout mechanism for this case (10 seconds), automatically triggers DMA transfer abort processing, and follows the same procedure description in “Application process crash” mentioned previously.

<a name="faqs"></a>
# FAQ

**Q: How do I get the Source code of the XDMA driver and compile it?**

The XDMA driver is included [AWS FPGA HDK/SDK](.), and may be included pre-installed in some Amazon Linux distributions.

Follow the [installation guide](./xdma_install.md) for more details.

**Q: How to discover the available FPGAs with the XDMA driver?**

Once the XDMA driver is running, all the available devices will be listed in /dev directory as /dev/xdmaX.

    `$ ls /dev/xdma*`
    
Each XDMA device exposes multiple channels under `/dev/xdmaX_h2c_Y` and `/dev/xdmaX_c2h_Y` and the developer can work directly with these character devices.

**Q: When my `write()`/`pwrite()` call is returned, am I guaranteed that the data reached the FPGA?** 

No. System calls to `write()`/`pwrite()` return the number of bytes which were written or read. It is up to the caller to make the call again if the operation did not complete.

**Q: XDMA는 데이터를 삭제할 수 있나요?**

정상 작동 중에 XDMA는 데이터를 삭제하지 않습니다.

XDMA가 데이터를 삭제하는 두 가지 경우는 다음과 같습니다:
1. 이 채널을 관리하는 사용자 프로세스의 갑작스러운 충돌
2. XDMA 전송 중 시간 초과.
  
  
**질문: `read()`/`pread()` 또는 `write()`/`pwrite()`가 시간 초과되나요?**

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

XDMA polled DMA descriptor completion mode is enabled at XDMA driver load time:

` $ insmod xdma.ko poll_mode=1`

XDMA interrupt DMA descriptor completion mode is also enabled at XDMA driver load time (default):

` $ insmod xdma.ko`
