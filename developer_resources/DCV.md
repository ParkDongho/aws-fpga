# NICE DCV를 사용한 GUI FPGA 개발 환경
이 가이드는 NICE DCV를 사용하여 FPGA 개발자 AMI를 사용하여 GUI FPGA 개발 환경을 설정하는 단계를 보여줍니다.
      
## Overview

[NICE DCV](https://docs.aws.amazon.com/dcv/latest/adminguide/what-is-dcv.html)를 사용하여 FPGA 개발자 AMI 인스턴스에 가상 데스크톱을 만들 수 있습니다.

[NICE DCV](https://docs.aws.amazon.com/dcv/latest/adminguide/what-is-dcv.html)는 고성능 원격 디스플레이 프로토콜로, 고객에게 다양한 네트워크 조건에서 모든 클라우드 또는 데이터 센터에서 모든 장치로 원격 데스크톱 및 애플리케이션 스트리밍을 안전하게 제공할 수 있는 방법을 제공합니다. 

고객은 NICE DCV와 Amazon EC2를 사용하여 EC2 인스턴스에서 그래픽 집약적인 애플리케이션을 원격으로 실행하고 그 결과를 더 간단한 클라이언트 머신으로 스트리밍할 수 있으므로 고가의 전용 워크스테이션이 필요하지 않습니다. 광범위한 HPC 워크로드의 고객들이 원격 시각화 요구사항에 NICE DCV를 사용합니다. NICE DCV 스트리밍 프로토콜은 Amazon AppStream 2.0 및 AWS RoboMaker와 같은 인기 서비스에서도 활용되고 있습니다.

[DCV 관리자 가이드](https://docs.aws.amazon.com/dcv/latest/adminguide/what-is-dcv.html) 및 [사용자 가이드](https://docs.aws.amazon.com/dcv/latest/userguide/getting-started.html) 는 DCV 구성 및 사용 방법에 대한 공식 리소스입니다.

설치 과정은 아래에 요약되어 있으므로 참고하시기 바랍니다.

**NOTE**:
이러한 단계는 새 버전의 DCV 서버 및 클라이언트가 출시되면 변경될 수 있습니다.
문제가 발생하면 [공식 DCV 문서](https://docs.aws.amazon.com/dcv/latest/adminguide/what-is-dcv.html)를 참조하세요.

## Installation Process

1. 인스턴스에 NICE DCV 엔드포인트에 대한 액세스 권한을 부여하는 [IAM 역할로 FPGA 개발자 AMI 인스턴스 설정](https://docs.aws.amazon.com/dcv/latest/adminguide/setting-up-license.html#setting-up-license-ec2)을 수행합니다.

    NICE DCV는 EC2에서 무료로 사용할 수 있습니다.

    NICE DCV 서버는 Amazon EC2 인스턴스에서 실행 중임을 자동으로 감지하고 주기적으로 Amazon S3 버킷에 연결하여 유효한 라이선스를 사용할 수 있는지 확인합니다. IAM 역할은 이 기능을 활성화합니다.
    
    위 가이드에 언급된 단계에 따라 다음 정책을 사용하여 인스턴스에 IAM 역할을 연결하세요:
    ```
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": "s3:GetObject",
                "Resource": "arn:aws:s3:::dcv-license.region/*"
            }
        ]
    }
    ```
    **참고: [NICE DCV 라이선스 설정 가이드](https://docs.aws.amazon.com/dcv/latest/adminguide/setting-up-license.html#setting-up-license-ec2)에 언급된 DCV 버킷에 액세스하지 않으면 서버 라이선스의 유효기간은 15일에 불과합니다.

1. FPGA 개발자 AMI 인스턴스에서 [인스턴스 보안 그룹 업데이트](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-network-security.html#adding-security-group-rule)를 통해 TCP 포트 **8443** 인그레스를 허용합니다.

1. [NICE DCV 사전 요구 사항 설치](https://docs.aws.amazon.com/dcv/latest/adminguide/setting-up-installing-linux-prereq.html)

   ```
   sudo yum -y install kernel-devel
   sudo yum -y groupinstall "GNOME Desktop"
   sudo yum -y install glx-utils
   ```

1. [NICE DCV 서버 설치](https://docs.aws.amazon.com/dcv/latest/adminguide/setting-up-installing-linux-server.html)

   ```
   sudo rpm --import https://d1uj6qtbmh3dt5.cloudfront.net/NICE-GPG-KEY
   wget https://d1uj6qtbmh3dt5.cloudfront.net/2021.2/Servers/nice-dcv-2021.2-11048-el7-x86_64.tgz
   tar -xvzf nice-dcv-2021.2-11048-el7-x86_64.tgz && cd nice-dcv-2021.2-11048-el7-x86_64
   sudo yum install nice-dcv-server-2021.2.11048-1.el7.x86_64.rpm
   sudo yum install nice-xdcv-2021.2.406-1.el7.x86_64.rpm

   sudo systemctl enable dcvserver
   sudo systemctl start dcvserver
   ```

1. 비밀번호 설정

   ```
   sudo passwd centos
   ```

1. 방화벽 설정 변경
   
   Options: 
   
   * 방화벽을 비활성화하여 모든 연결 허용
   ```
   sudo systemctl stop firewalld
   sudo systemctl disable firewalld
   ```
   
   * tcp 포트 8443에 대해서만 방화벽을 엽니다.
   
   ```
   sudo systemctl start firewalld
   sudo systemctl enable firewalld
   sudo firewall-cmd --zone=public --add-port=8443/tcp --permanent
   sudo firewall-cmd --reload
   ```

1. 연결할 가상 세션을 만듭니다.    
   
   **참고: 인스턴스를 다시 시작하면 새 세션을 만들어야 합니다.** 

   ```
   dcv create-session --type virtual --user centos centos --owner centos
   ```

1. DCV 원격 데스크톱 세션에 연결

    1. **웹 브라우저 사용**
    
       * [지원되는 웹 브라우저](https://docs.aws.amazon.com/dcv/latest/adminguide/what-is-dcv.html#what-is-dcv-requirements)를 사용하고 있는지 확인하세요.
       
       * 보안 URL, 공인 IP 주소, 올바른 포트(8443)를 사용하여 연결합니다. 예: `https://111.222.333.444:8443`
    
          **참고:** 연결할 때 안전한 연결을 위해 `https` 프로토콜을 사용해야 합니다.              

    1. **NICE DCV 클라이언트 사용**
    
       * [DCV 클라이언트](https://download.nice-dcv.com/)를 다운로드하여 설치합니다.
       
       * 공인 IP 주소와 올바른 포트(8443)를 사용하여 연결합니다.

          로그인 화면 예시 (DCV 클라이언트의 경우, `111.222.333.444:8443`과 같은 IP:포트를 사용하여 먼저 연결해야 합니다):
    
          ![DCV Login](images/dcv_login.png)

1. 로그인하면 새 GUI 데스크톱이 표시됩니다:

    ![DCV Desktop](images/dcv_desktop.png)
