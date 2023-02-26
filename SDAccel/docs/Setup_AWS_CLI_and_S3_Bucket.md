## CLI 설정 및 S3 버킷 만들기
개발자는 AFI 생성을 위해 S3 버킷을 생성해야 합니다. 이 버킷에는 AFI 생성 서비스에서 생성된 tar 파일과 로그가 포함됩니다. 

AWS CLI를 설치하려면 [여기](http://docs.aws.amazon.com/cli/latest/userguide/installing.html)의 지침을 따르세요.

AWS SDAccel 스크립트는 JSON 출력 형식을 요구하며, 다른 출력 형식(예: 텍스트, 테이블)을 사용하면 스크립트가 제대로 작동하지 않습니다.  JSON은 AWS CLI의 기본 출력 형식입니다.

```
    $ aws configure         # 을 클릭하여 자격 증명(console.aws.amazon.com 페이지에 있음), 지역(us-east-1) 및 출력(json)을 설정합니다. 
```
이 S3 버킷은 AWS 스크립트에서 AFI 생성을 위해 AWS에 DCP를 업로드하는 데 사용되며, 이는 tar 파일로 패키징됩니다.  
버킷을 생성하는 것으로 시작하세요:
```
    $ aws s3 mb s3://<bucket-name> --region us-east-1  # Create an S3 bucket (choose a unique bucket name)
    $ touch FILES_GO_HERE.txt                          # Create a temp file
    $ aws s3 cp FILES_GO_HERE.txt s3://<bucket-name>/<dcp-folder-name>/  # Choose a dcp folder name 
```
AFI 생성 프로세스는 로그를 생성하고 S3 버킷에 저장합니다. 이 로그는 AFI 생성에 실패할 경우 디버그에 사용할 수 있습니다.  
다음으로 로그 파일을 위한 폴더를 만듭니다:        
```    
    $ touch LOGS_FILES_GO_HERE.txt                     # Create a temp file
    $ aws s3 cp LOGS_FILES_GO_HERE.txt s3://<bucket-name>/<logs-folder-name>/  # Choose a logs folder name
```             
AFI가 성공적으로 생성되면 필요에 따라 tar 파일과 로그를 자유롭게 삭제할 수 있습니다.  이러한 파일을 삭제해도 AFI는 삭제되거나 수정되지 않습니다.
