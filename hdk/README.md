# AWS FPGA Hardware Development Kit (HDK)

## Table of Contents
1. [HDK 개요]( #overview)
2. [시작하기]( #gettingstarted)
    - [AWS 계정, F1/EC2 인스턴스, 온프레미스, AWS IAM 권한, AWS CLI 및 S3 설정(일회성 설정)](#iss)
    - [HDK 설치 및 설정 환경](#setup)
    - [예제 리뷰](#examples)
3. [CL 예제 중 하나에서 Amazon FPGA 이미지(AFI)를 생성하는 방법 3: 단계별 가이드](#endtoend)
    - [FPGA 인스턴스에서 CL 예제를 실행하는 빠른 경로](#fastpath)
    - [1단계: 예제 중 하나를 선택하고 해당 디렉토리로 이동](#step1)  
    - [2단계. CL 빌드](#step2)
    - [3단계: 설계 체크포인트를 AWS에 제출하여 AFI 생성](#step3)
    - [4단계. AWS FPGA 관리 도구 설정](#step4)
    - [5단계. AFI 로드](#step5)
    - [CL 예제 소프트웨어를 이용한 검증](#step6)
4. [커스텀 로직(CL) RTL 설계 시뮬레이션](#simcl)
5. [커스텀 로직 설계 시작하기](#buildcl)(Verilog 또는 VHDL 사용, RTL 흐름)

<a name="overview"></a>
## HDK 개요

* HDK는 개발자에게 F1에서 완전 맞춤형 하드웨어 가속기를 빌드할 수 있는 정보, 예제 및 스크립트를 제공합니다.  완전 커스텀 하드웨어 가속기는 개발하는 데 몇 달이 걸릴 수 있으며 개발자는 이에 익숙해야 합니다:
  * 스크립팅(셸, tcl)
  * RTL(Verilog 또는 VHDL) 개발
  * 합성 툴 및 타이밍 임계 경로를 식별하고 타이밍을 맞추기 위해 하드웨어를 최적화하는 반복 프로세스
  * FPGA, DMA, DDR, AXI 프로토콜 및 Linux 드라이버 설계와 관련된 개념 숙지
  * RTL 시뮬레이션 
  * 시뮬레이션 디버그 또는 FPGA 런타임 파형 뷰어 디버그 방법에 대한 경험
* 이러한 분야에 익숙하지 않은 개발자는 [소프트웨어 정의 가속](../Vitis/README.md)부터 시작해야 합니다.    
* 위에 나열된 영역에 익숙하지 않은 기존 RTL IP를 사용하는 개발자는 [소프트웨어 정의 가속](../Vitis/README.md)을 사용하여 RTL 커널 개발부터 시작해야 합니다.
* 보다 빠른 HDK 개발 경로를 원하는 개발자는 [소프트웨어 정의 가속](../Vitis/README.md)을 사용하여 RTL 커널 개발부터 시작해야 합니다. 

* [문서 디렉토리](./docs)는 AWS 셸(SH)-커스텀 로직(CL) 인터페이스에 대한 사양을 제공합니다:
  * [셸 인터페이스](./docs/AWS_Shell_Interface_Specification.md)
  * [셸 주소 맵](./docs/AWS_Fpga_Pcie_Memory_Map.md)
  * [프로그래머의 FPGA 뷰](./docs/Programmer_View.md)
  * [가상 JTAG](./docs/Virtual_JTAG_XVC.md)
  * 테스트벤치 및 셸 시뮬레이션 모델을 사용한 RTL 설계 시뮬레이션](./docs/RTL_Simulating_CL_Designs.md)
  * [전력 분석](./docs/afi_power.md)
  * [셸 타임아웃 감지](./docs/HOWTO_detect_shell_timeout.md)
  
* [공통 디렉터리](./common)에는 공통 환경 설정 스크립트, 공통 빌드 스크립트 및 제약 조건 파일, DRAM 컨트롤러와 같은 IP 라이브러리가 포함됩니다. 이 디렉터리에는 `shell_stable` 디렉터리 아래에 참조되는 프로덕션 셸이 포함되어 있습니다.  AWS 셸 디자인 체크포인트(DCP)는 HDK 설정 중에 S3에서 공통 디렉터리로 다운로드됩니다.
  * 개발자는 `/common` 디렉토리 아래의 파일을 변경할 필요가 없습니다.
  * 셸_안정` 디렉터리에는 개발자가 현재 프로덕션 셸을 사용하여 CL을 빌드하는 데 필요한 파일이 포함되어 있습니다.

* 커스텀 로직(cl) 디렉토리](./cl)는 커스텀 로직이 개발될 것으로 예상되는 곳입니다(Verilog 또는 VHDL을 사용하는 RTL 기반 개발의 경우). 여기에는 [예제 디렉토리](./cl/examples) 아래에 여러 예제가 포함되어 있으며, [개발자_디자인 디렉토리](./cl/developer_designs) 아래에 개발자의 자체 커스텀 로직에 대한 자리 표시자가 있습니다.
예제에 대한 자세한 내용은 [예제 테이블](./cl/examples/cl_examples_list.md)을 참조하세요.

<a name="gettingstarted"></a>
## 시작하기

<a name="iss"></a>
#### AWS 계정, F1/EC2 인스턴스, 온프레미스, AWS IAM 권한, AWS CLI 및 S3 설정(일회성 설정)
* [AWS 계정 설정](https://aws.amazon.com/free/)
* 비바도와 함께 사전 설치되어 있는 [FPGA 개발자 AMI](https://aws.amazon.com/marketplace/pp/B06VVYBLZZ)와 필요한 라이선스를 사용하여 인스턴스를 실행합니다.  AWS FPGA 인스턴스 내부에 사용되는 FPGA의 크기가 크기 때문에 구현 도구에는 32GiB 메모리가 필요합니다(예: c4.4xlarge, m4.2xlarge, r4.xlarge, t2.2xlarge). c4.4xlarge와 c4.8xlarge는 각각 30, 60GiB의 메모리로 가장 빠른 실행 시간을 제공합니다. 비용을 절감하려는 개발자는 t2.2xlarge와 같은 저비용 인스턴스에서 코딩을 시작하고 시뮬레이션을 실행한 다음 앞서 언급한 대용량 인스턴스로 이동하여 가속 코드 합성을 실행할 수 있습니다.  온프레미스 지침](../docs/on_premise_licensing_help.md)에 따라 Xilinx에서 라이선스를 구매하고 설치합니다.
* 호환성 표는 개발자 키트 버전과 [FPGA Developer AMI](https://aws.amazon.com/marketplace/pp/B06VVYBLZZ) 버전의 매핑에 대해 설명합니다:  

| 개발자 키트 버전 | 지원되는 툴 버전 | 호환되는 FPGA 개발자 AMI 버전 |
|-----------|-----------|------|
| 1.3.0-1.3.6 | 2017.1(Deprecated) | v1.3.5(Deprecated) |
| 1.3.7-1.3.X | 2017.1(Deprecated) | v1.3.5-v1.3.X(Deprecated) |
| 1.3.7-1.4.15a | 2017.4 | v1.4.0-v1.4.X (Xilinx Vivado 2017.4) |
| 1.4.3-1.4.15a | 2018.2 | v1.5.0 (Xilinx Vivado 2018.2) |
| 1.4.8-1.4.15a | 2018.3 | v1.6.0 (Xilinx Vivado 2018.3) |
| 1.4.11-1.4.x | 2019.1 | v1.7.0 (Xilinx Vivado 2019.1) |
| 1.4.11-1.4.x | 2019.2 | v1.8.x (Xilinx Vivado 2019.2) |
| 1.4.16-1.4.x | 2020.1 | v1.9.x (Xilinx Vivado 2020.1) |
| 1.4.18-1.4.x | 2020.2 | v1.10.x (Xilinx Vivado 2020.2) |


* FPGA 개발자 키트 버전은 [hdk_version.txt](./hdk_version.txt)에 나열되어 있습니다.

* FPGA 개발자 키트 지원 툴 버전은 [supported\_vivado\_versions](../supported_vivado_versions.txt)에 나열됩니다.

* FPGA 이미지 생성을 위한 AWS IAM 권한 설정(CreateFpgaImage 및 DescribeFpgaImages). [EC2 API 권한은 더 자세히 설명되어 있습니다](http://docs.aws.amazon.com/AWSEC2/latest/APIReference/ec2-api-permissions.html)를 참조하십시오.  이 빠른 시작을 진행하기 전에 AWS IAM 권한의 유효성을 검사하는 것이 좋습니다.  DescribeFpgaImages API](docs/describe_fpga_images.md)를 호출하여 IAM 권한이 올바른지 확인할 수 있습니다.

* [AWS CLI 및 S3 버킷 설정](../SDAccel/docs/Setup_AWS_CLI_and_S3_Bucket.md)을 설치하여 AFI 생성을 활성화합니다.
  * AWS CLI 설치는 [AWS CLI 설치 가이드](http://docs.aws.amazon.com/cli/latest/userguide/installing.html)를 참고하시기 바랍니다.
    ```
    $ aws configure     # 를 사용하여 자격 증명(console.aws.amazon.com 페이지에 있음)과 기본 리전을 설정합니다.
    ```

프로필 기본 지역을 재정의하려면 aws-cli [region](http://docs.aws.amazon.com/cli/latest/userguide/cli-command-line.html) 명령줄 인수를 사용합니다.  지원되는 지역은 us-east-1, us-west-2, eu-west-1 및 us-gov-west-1입니다.

<a name="setup"></a>
#### HDK 설치 및 설정 환경

AWS FPGA HDK를 실행하여 인스턴스에 복제할 수 있습니다:

> FPGA 개발자 AMI를 사용하는 경우 다음을 추가합니다:  
> `AWS_FPGA_REPO_DIR=/home/centos/src/project_data/aws-fpga`를 추가합니다.
    
```bash
    $ git clone https://github.com/aws/aws-fpga.git $AWS_FPGA_REPO_DIR
    $ cd $AWS_FPGA_REPO_DIR
    $ source hdk_setup.sh
```

hdk_setup.sh` 소싱은 다음을 수행합니다:
* HDK의 예제 전체에서 사용되는 필수 환경 변수를 설정합니다.  
* S3에서 DDR 시뮬레이션 모델과 DCP를 다운로드합니다.  

새 단말기 또는 기존 단말기는 올바른 환경 변수를 설정하기 위해 `hdk_setup.sh`를 다시 실행해야 합니다.  

<a name="examples"></a>
#### 예제 리뷰 

[Examples readme](./cl/exples/cl_exples_list.md)에는 개발자가 사용할 수 있는 모든 예제에 대한 개요가 나와 있습니다.

<a name="endtoend"></a>
## CL 예제 중 하나에서 Amazon FPGA 이미지(AFI)를 생성하는 방법: 단계별 가이드

<a name="fastpath"></a>
#### FPGA 인스턴스에서 CL 예제를 실행하는 빠른 경로

개발 흐름을 건너뛰고 FPGA 인스턴스에서 예제 실행을 시작하려는 개발자를 위한 것입니다.  개발 프로세스에 관심이 없는 경우 1~3단계를 건너뛰어도 됩니다.  4단계부터 6단계까지는 미리 설계된 AFI 예제 중 하나를 사용하는 방법을 보여줍니다. 
공개 AFI를 사용하면 개발자는 빌드 흐름 단계를 건너뛰고 4단계로 이동할 수 있습니다. [공개 AFI는 각 예제마다 사용할 수 있으며, 예제/README](cl/examples/cl_hello_world/README.md#metadata)에서 찾을 수 있습니다.

<a name="step1"></a>
#### 1단계. 예제 중 하나를 선택하고 예제 디렉토리에서 시작합니다.

HDK 헬로 월드 예제를 사용하여 이 단계별 가이드를 완료하는 것이 좋습니다.  그런 다음 동일한 가이드를 사용하여 [cl\_dram\_dma](cl/examples/cl_dram_dma)를 사용하여 개발하세요.  준비가 되면 제공된 예제 중 하나를 복사하고 디자인 파일, 스크립트 및 제약 조건 디렉터리를 수정합니다.

```
    $ cd $HDK_DIR/cl/examples/cl_hello_world    # CL_HELLO_WORLD를 CL_DRAM_DMA, CL_URAM_EXAMPLE 또는 CL_HELLO_WORLD_VHDL로 변경할 수 있습니다.
    $ export CL_DIR=$(pwd)
```

빌드 스크립트가 해당 값에 의존하므로 CL_DIR 환경 변수를 설정하는 것이 중요합니다.
각 예제에서는 권장 디렉터리 구조를 따라 HDK 시뮬레이션 및 빌드 스크립트에 예상되는 구조와 일치하도록 합니다.

<a name="step2"></a>
#### 2단계. CL 빌드

빌드 프로세스를 시작하기 전에 이 [체크리스트](./cl/CHECKLIST_BEFORE_BUILDING_CL.md)를 참조해야 합니다.

**참고** *이 단계를 수행하려면 자일링스 Vivado 툴 및 라이선스가 설치되어 있어야 합니다*.
```
    $ vivado -mode batch        # Vivado가 설치되었는지 확인합니다.
```

`aws_build_dcp_from_cl.sh` 스크립트를 실행하면 CL 설계를 타겟 FPGA의 타이밍 및 배치 제약 조건을 충족하는 완성된 설계 체크포인트로 변환하는 전체 구현 프로세스가 수행됩니다.
출력은 DCP 파일과 기타 로그/매니페스트 파일로 구성된 타르볼 파일로, 형식은 `YY_MM_DD-hhmm.Developer_CL.tar`입니다.
이 파일은 AFI를 생성하기 위해 AWS에 제출됩니다. 기본적으로 빌드 스크립트는 125MHz의 메인 클럭을 사용하는 클럭 그룹 A 레시피 A0을 사용합니다.

```    
    $ cd $CL_DIR/build/scripts
    $ ./aws_build_dcp_from_cl.sh
```

250MHz 메인 클럭을 사용하기 위해 개발자는 다음 예제에서와 같이 A1 클럭 그룹 A 레시피를 지정할 수 있습니다:

```
    $ cd $CL_DIR/build/scripts
    $ ./aws_build_dcp_from_cl.sh -clock_recipe_a A1
```

다른 클럭 레시피도 지정할 수 있습니다. [클럭 그룹 레시피 표](./docs/clock_recipes.csv)에 대한 자세한 내용과 다른 레시피를 지정하는 방법은 다음 [README](./common/shell_v04261818/new_cl_template/build/README.md)에서 확인할 수 있습니다.

**참고**: *DCP 생성은 완료하는 데 최대 몇 시간이 걸릴 수 있으므로 `aws_build_dcp_from_cl.sh`는 `nohup` 컨텍스트 내에서 주 빌드 프로세스(`vivado`)를 실행합니다: 이렇게 하면 SSH 세션이 실행 도중에 종료되더라도 빌드가 계속 실행될 수 있습니다*.

빌드가 완료되면 이메일로 알림을 받으려면:

1. SNS를 통한 알림을 설정합니다:

```
    $ pip install --user --upgrade boto3 # boto3 package is required by the notify_via_sns script
    $ export EMAIL=your.email@example.com
    $ $AWS_FPGA_REPO_DIR/shared/bin/scripts/notify_via_sns.py

```

2. 이메일 주소를 확인하고 구독을 확인합니다.
3. aws_build_dcp_from_cl.sh` 호출 시, `-notify` 스위치를 추가합니다.
4. 빌드가 완료되면 "빌드가 완료되었습니다"라는 이메일이 발송됩니다.
5. 각 예제에 대해 알려진 경고는 $CL_DIR/build/scripts 디렉터리에 있는 warnings.txt 파일에 문서화되어 있습니다.
   [cl\_hello\_world warnings](cl/examples/cl_hello_world/build/scripts/warnings.txt )
   [cl\_dram\_dma warnings](cl/examples/cl_dram_dma/build/scripts/warnings.txt )
   [cl\_uram\_example warnings](cl/examples/cl_uram_example/build/scripts/warnings.txt )

<a name="step3"></a>
#### 3단계. AWS에 디자인 체크포인트를 제출하여 AFI 생성하기

DCP를 제출하려면 디자인을 제출하기 위한 S3 버킷을 생성하고 해당 버킷에 타볼 파일을 업로드합니다.
다음 정보를 준비해야 합니다:

1. 로직 디자인 이름 *(선택 사항)*.
2. 로직 디자인에 대한 일반적인 설명 *(선택 사항)*.
3. S3에서 타르볼 파일 객체의 위치.
4. AWS가 AFI 생성 로그를 다시 기록할 S3 디렉터리 위치.
5. AFI가 생성될 AWS 리전.  다른 리전으로 AFI를 복사하려면 [copy-fpga-image](./docs/copy_fpga_image.md) API를 사용합니다.

타르볼 파일을 S3에 업로드하려면 [S3에서 지원하는 도구](http://docs.aws.amazon.com/AmazonS3/latest/dev/UploadingObjects.html) 중 하나를 사용할 수 있습니다.

예를 들어 다음과 같이 AWS CLI를 사용할 수 있습니다:

타볼을 위한 버킷과 폴더를 생성한 다음 S3로 복사합니다.
```
    $ aws s3 mb s3://<bucket-name> --region <region>   # Create an S3 bucket (choose a unique bucket name)
    $ aws s3 mb s3://<bucket-name>/<dcp-folder-name>/   # Create folder for your tarball files
    $ aws s3 cp $CL_DIR/build/checkpoints/to_aws/*.Developer_CL.tar \       # Upload the file to S3
             s3://<bucket-name>/<dcp-folder-name>/

     NOTE: The trailing '/' is required after <dcp-folder-name>
```
로그 파일을 위한 폴더 만들기        
```    
    $ aws s3 mb s3://<bucket-name>/<logs-folder-name>/  # Create a folder to keep your logs
    $ touch LOGS_FILES_GO_HERE.txt                     # Create a temp file
    $ aws s3 cp LOGS_FILES_GO_HERE.txt s3://<bucket-name>/<logs-folder-name>/  #Which creates the folder on S3
    
    NOTE: The trailing '/' is required after <logs-folder-name>
```             

AFI 생성을 시작합니다. 
```
    $ aws ec2 create-fpga-image \
        --region <region> \
        --name <afi-name> \
        --description <afi-description> \
        --input-storage-location Bucket=<dcp-bucket-name>,Key=<path-to-tarball> \
        --logs-storage-location Bucket=<logs-bucket-name>,Key=<path-to-logs> \
	[ --client-token <value> ] \
	[ --dry-run | --no-dry-run ]
	
    NOTE: <path-to-tarball> is <dcp-folder-name>/<tar-file-name>
          <path-to-logs> is <logs-folder-name>
```

이 명령의 출력에는 AFI를 참조하는 두 개의 식별자가 포함됩니다:
- **FPGA 이미지 식별자** 또는 **AFI ID**: 이 식별자는 AWS EC2 CLI 명령과 AWS SDK API를 통해 AFI를 관리하는 데 사용되는 기본 ID입니다.
    이 ID는 지역별로 다르며, 즉 AFI가 여러 지역에 걸쳐 복사되는 경우 각 지역마다 다른 고유 AFI ID를 갖게 됩니다.
    예시적인 AFI ID는 **`afi-06d0ffc989feeea2a`**입니다.
- **글로럴 FPGA 이미지 식별자** 또는 **AGFI ID**: F1 인스턴스 내에서 AFI를 참조하는 데 사용되는 글로벌 ID입니다.
    예를 들어, FPGA 슬롯에서 AFI를 로드하거나 지우려면 AGFI ID를 사용합니다.
    AGFI ID는 (설계상) 전역이므로 AFI/AMI 조합을 여러 리전으로 복사할 수 있으며, 추가 설정 없이도 작동합니다.
    예시적인 AGFI ID는 **`agfi-0f0e045f919413242`**입니다.

[describe-fpga-images](./docs/describe_fpga_images.md) API를 사용하면 백그라운드 AFI 생성 프로세스 중에 AFI 상태를 확인할 수 있습니다.
`create-fpga-image`가 반환하는 **FPGA 이미지 식별자**를 제공해야 합니다:
```
    $ aws ec2 describe-fpga-images --fpga-image-ids afi-06d0ffc989feeea2a
```

[wait_for_afi.py](./docs/wait_for_afi.md) 스크립트를 사용하여 AFI 생성이 완료될 때까지 기다린 다음 선택적으로 다음과 같이 이메일을 보낼 수 있습니다.
결과를 이메일로 전송합니다.

AFI 생성이 완료되고 AFI 상태가 `available`으로 설정된 경우에만 인스턴스에 AFI를 로드할 수 있습니다: 
```
    {
        "FpgaImages": [
            {
			    ...
                "State": {
                    "Code": "available"
                },
			    ...
                "FpgaImageId": "afi-06d0ffc989feeea2a",
			    ...
            }
        ]
    }

```

AFI 생성이 완료되면 AWS는 개발자가 제공한 버킷 위치(```s3://<버킷 이름>/<로그 폴더 이름>```)에 로그를 넣습니다. 이러한 로그가 있으면 생성 프로세스가 완료되었음을 나타냅니다. AFI의 상태(예: 사용 가능 또는 실패)를 나타내는 "상태" 파일 또는 생성 프로세스 중에 발생한 오류를 자세히 설명하는 Vivado 로그를 찾아보세요.  AFI 생성 문제에 대한 도움말은 [create-fpga-image 오류 코드](./docs/create_fpga_image_error_codes.md)를 참조하십시오.

**참고**: *인스턴스에서 AFI를 즉시 로드하려고 시도하면 `Invalid AFI ID` 오류가 발생합니다.
AFI가 성공적으로 생성되었는지 확인할 때까지 기다리세요.

**참고**: *AFI가 생성된 지역과 다른 지역에서 AFI를 로드하려고 하면 `Invalid AFI ID` 오류가 발생합니다.  AFI는 리전으로 복사해야 합니다*.
[copy-fpga-image](./docs/copy_fpga_image.md) API를 사용하면 AFI를 다른 리전으로 복사하여 시간이 많이 걸리는 `create-fpga-image` 프로세스를 피할 수 있습니다. 또한 복사하면 소스 글로벌 AFI ID를 보존하고 인스턴스 코드나 스크립트의 지역별 변경 사항을 최소화할 수 있습니다.

#### F1 인스턴스 내에서 등록된 AFI를 로드하고 테스트하는 방법을 단계별로 안내합니다.

다음 단계를 수행하려면 F1 인스턴스를 시작해야 합니다.
AWS는 FPGA 관리 도구가 포함된 최신 Amazon Linux 인스턴스를 시작하거나, HDK와 SDK가 모두 포함된 FPGA 개발자 AMI를 사용하는 것을 권장합니다.

<a name="step4"></a>
#### 4단계. AWS FPGA 관리 도구 설정

AFI를 FPGA에 로드하려면 FPGA 관리 툴이 필요합니다.  F1 인스턴스를 실행하는 데 사용되는 AMI에 따라 이러한 단계가 이미 완료되었을 수 있습니다.
```
    $ git clone https://github.com/aws/aws-fpga.git $AWS_FPGA_REPO_DIR
    $ cd $AWS_FPGA_REPO_DIR
    $ source sdk_setup.sh
```
AWS CLI 설치는 [AWS CLI 설치 가이드](http://docs.aws.amazon.com/cli/latest/userguide/installing.html)를 참고하세요.
```
    $ aws configure         # 다음을 통해 자격 증명(console.aws.amazon.com 페이지에 있음)과 인스턴스 지역(us-east-1, us-west-2, eu-west-1 또는 us-gov-west-1)을 설정합니다.
```
<a name="step5"></a>
#### 5단계. AFI 로드

이제 F1 인스턴스 내에서 FPGA 관리 툴을 사용하여 특정 슬롯의 FPGA에 AFI를 로드할 수 있습니다.
이전에 슬롯에 로드한 AFI를 모두 지워야 합니다:
```
    $ sudo fpga-clear-local-image  -S 0
```

또한 `fpga-describe-local-이미지` 명령을 호출하여 특정 슬롯에 로드된 AFI(있는 경우)를 확인할 수 있습니다.
예를 들어 슬롯이 지워진 경우(이 예제에서는 `slot 0`) 다음과 유사한 출력이 표시됩니다:

```
    $ sudo fpga-describe-local-image -S 0 -H

    Type  FpgaImageSlot  FpgaImageId             StatusName    StatusCode   ErrorName    ErrorCode   ShVersion
    AFI          0       none                    cleared           1        ok               0       <shell_version>
    Type  FpgaImageSlot  VendorId    DeviceId    DBDF
    AFIDEVICE    0       0x1d0f      0x1042      0000:00:0f.0
```

fpga-describe-local-image API 호출이 'Busy' 상태를 반환하면 FPGA가 여전히 백그라운드에서 이전 작업을 수행하고 있는 것입니다. 위와 같이 상태가 'Cleared'가 될 때까지 기다려주세요.

이제 FPGA `슬롯 0`에 AFI를 로드해 보겠습니다:

```
    $ sudo fpga-load-local-image -S 0 -I agfi-0fcf87119b8e97bf3
```


**참고**: *FPGA 관리 툴은 AFI ID가 아닌 AGFI ID를 사용합니다.*.

이제 AFI가 제대로 로드되었는지 확인할 수 있습니다.  출력에는 FPGA 이미지 "로드" 작업 후 "로드된" 상태의 FPGA가 표시됩니다.  "-R" 옵션은 고유한 AFI 공급업체 및 장치 ID를 노출하기 위해 PCI 장치 제거 및 재검색을 수행합니다.
```
    $ sudo fpga-describe-local-image -S 0 -R -H
    Type  FpgaImageSlot  FpgaImageId             StatusName    StatusCode   ErrorName    ErrorCode   ShVersion
    AFI          0       agfi-0fcf87119b8e97bf3  loaded            0        ok               0       0x04261818
    Type  FpgaImageSlot  VendorId    DeviceId    DBDF
    AFIDEVICE    0       0x1d0f      0xf000      0000:00:1d.0
```
    
<a name="step6"></a>
#### 6단계. CL 예제 소프트웨어를 사용하여 검증

사전 설치된 Xilinx 런타임 환경(XRT)과 함께 제공되는 AMI 1.5.0 이상 인스턴스를 사용하는 개발자는 XDMA 드라이버를 설치하기 전에 [참고 사항](../sdk/linux_kernel_drivers/xdma/xdma_install.md#xdmainstallfail)을 참고해야 합니다.

각 CL 예제는 `$CL_DIR/software/runtime/` 하위 디렉토리에 런타임 소프트웨어와 함께 제공됩니다. 로드된 AFI와 일치하는 런타임 애플리케이션을 빌드해야 합니다.   

```
    $ cd $CL_DIR/software/runtime/
    $ make all
    $ sudo ./test_hello_world
```

예제별 추가 정보는 $CL_DIR/README.md에 있는 README.md를 검토하세요.

<a name="simcl"></a>
## 커스텀 로직 디자인 시뮬레이션(RTL 시뮬레이션)

Vivado XSIM 시뮬레이터를 사용하거나 자체 시뮬레이터(예: Synopsys의 VCS, Mentor의 Questa 또는 Cadence Incisive)를 가져올 수 있습니다.
[RTL 시뮬레이션 환경 설정](./docs/RTL_Simulating_CL_Designs.md#소개)에 따라 시뮬레이션을 실행하세요.

<a name="buildcl"></a>
## 나만의 커스텀 로직 설계 시작(RTL 흐름, Verilog 또는 VHDL 사용)

* 새 설계를 시작하기 전에 AWS 셸(SH)에서 커스텀 로직(CL)으로의 [인터페이스](./docs/AWS_Shell_Interface_Specification.md)의 사양을 검토하세요.
* [디버그 흐름](docs/Virtual_JTAG_XVC.md)을 시도하고 [셸 타임아웃 동작](docs/HOWTO_detect_shell_timeout.md)을 이해합니다.
* 준비가 되면 예제를 [나만의 CL 디자인 시작하기](./cl/developer_designs/Starting_Your_Own_CL.md)에 복사하고 간단한 수정을 통해 개발 요구 사항에 맞게 하드웨어 개발자 키트를 커스터마이징하는 데 익숙해지도록 합니다.



