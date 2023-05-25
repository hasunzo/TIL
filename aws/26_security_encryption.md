# Encryption in flight (SSL) 전송 중 암호화
- 매우 민감한 기밀 내용인 경우 전송 중 암호화를 사용
- 신용카드로 온라인 결제 시 네트워크 패킷이 가는 길에 타인의 신용카드 번호가 노출되면 안됨
- 전송 중 암호화가 되면 데이터가 전송되기 전 암호화되고 서버가 데이터를 받으면 복호화
- 전송하는 이와 서버만이 암호화와 복호화 하는 방법을 알고 있음
- SSL 인증서가 암호화를 해주고 다른 방법은 HTTPS가 있음
- Amazon 서비스를 다룰 때마다 HTTPS 엔드포인트가 있어서 전송 중 암호화가 됐음을 보장
- 모든 프로그래밍 언어가 SSL 암호화와 복호화를 할 수 있고 모든 라이브러리가 이 작업을 해줌. 직접 처리할 필요 없음

# Server side encryption at rest 서버 측 저장 데이터 암호화
- 데이터가 서버에 수신된 후 암호화
- 서버가 데이터를 받아서 복호화를 하고 암호 복호화된 형식에 사용
- 서버는 데이터를 디스크에 저장
- 서버가 데이터를 암호화된 형태로 저장
  - 서버가 해킹을 당하면 데이터가 복호화 되어서는 안됨
- 데이터는 클라이언트로 다시 전송되기 전에 복호화됨
- 데이터 키라고 불리는 키 덕분에 데이터는 암호화된 형태로 저장되고 암호 키와 해독 키는 주로 KMS (Key Management Service) 같은 곳에 따로 관리
- 서버는 KMS와 통신할 수 있어야 함
- 객체를 EBS로 전송하는 예시
  - 객체는 어떤 매커니즘으로든 전송
  - EBS는 데이터 키를 써서 해당 데이터의 암호화를 수행하고 함호화된 형태로 저장
  - 해당 데이터를 어떤 이유에서든 검색하려고 할 때 
  - EBS 같은 AWS가 대신 데이터 키를 통해 복호화를 하면
  - 복호화된 데이터가 가령 HTTP, HTTPS를 거쳐 돌아오게 됨
- 서비스의 서버 자체가 암호화와 복호화를 관리하고 데이터 키를 통해 접근

# Client side encryption 클라이언트 측 암호화
- 클라이언트가 데이터 암호화
- 서버는 데이터를 절대 복호화할 수 없고 데이터를 받는 클라이언트에 의해 복호화
- 전체적으로 데이터는 서버에 저장되지만 서버는 데이터늬 내용을 알 수 없음
- 서버는 최선의 방법으로도 데이터를 복호화할 수 없어야 함
- 예시
  - 클라이언트 데이터 키를 사용해서 객체 데이터를 암호화
  - 해당 데이터를 원하는 데이터 저장소로 전송
    - FTP, S3 등..
  - 데이터를 받으면 클라이언트는 암호화된 객체를 받고
  - 데이터 키에 액세스가 있거나 다른 곳으로부터 데이터 키를 찾을 수 있으면 받은 데이터를 복호화하여 복호화된 객체를 얻음
- 서버 데이터 저장소는 데이터를 암호화하거나 복호화할 수 없음
  - 암호화된 데이터를 받기만 할 뿐

# AWS KMS (Key Management Service)
- AWS 서비스로 암호화한다고 하면 KMS 암호화일 가능성이 큼
- KMS 서비스를 사용하면 AWS에서 암호화 키를 관리
- KMS는 권한 부여를 위해 IAM과 완전히 통합됨
- KMS로 암호화한 데이터에 관한 액세스를 더 쉽게 제어하도록 함
- CloudTrail을 통해 키를 사용하기 위해 호출한 모든 API를 감사할 수 있음 (시험출제)
- 저장 데이터를 EBS 볼륨에서 암호화하려면 KMS 통합을 활성화하면 됨
- S3, RDS, SSM에서도 동일하며 암호화가 필요한 거의 모든 서비스에서 동일
- 암호 데이터는 절대 평문으로 저장하면 안됨
- KMS를 사용하기 위해 API 호출을 해도 됨
- AWS CLI나 SDK도 사용 가능
- KMS 키는 사용해서 어떤 파일이든 필요하면 암호화할 수 있음
- 이렇게 암호화된 파일은 코드나 환경변수에 저장
## KMS Keys Types
### Symmetric (AES-256 keys) 대칭
- 대칭 KMS 키에는 데이터 암호화와 복호화에 사용하는 단일 암호화 키만 있어서 KMS와 통합된 모든 AWS 서비스는 대칭(Symmetric) 키를 사용함
- KMS 대칭 키를 생성하거나 사용하면 키 자체에는 절대로 액세스할 수 없고
- 키를 활용 또는 사용하려면 KMS API를 호출해야 함
### Asymmetric (RSA & ECC key pairs) 비대칭
- 2개의 키를 의미
  - 데이터 암호화에 사용하는 퍼블릭 키와 데이터 복호화에 사용하는 프라이빗 키
- 암호화, 복호화와 작업 할당 및 검증에 사용
- 이 경우 KMS에서 퍼블릭 키를 다운로드 할 수 있지만 프라이빗 키에는 액세스 할 수 없음
- 프라이빗 키에 액세스 하려면 API 호출로만 가능
- 비대칭 키의 사용 사례
  - KMS API 키에 액세스할 수 없는 사용자가 AWS 클라우드 외부에서 암호화할 때 사용
  - 퍼블릭 키로 데이터를 암호화하고 외부에서 다시 보내면 여러분은 계정에서 AWS 프라이빗 키로 해당 데이터를 복호화하는 것
### Three types of KMS Keys:
- AWS Managed Key: free (aws/service-name, example: aws/rds or aws/ebs)
- Customer Managed Keys (CMK) created in KMS: $1 / month
- Customer Managed Keys imported (must be 256-bit symmetric key): $1 /month
### + pay for API call to KMS ($0.03 / 10,000 calls)
- KMS로 호출하는 모든 API도 비용을 지불해야 함
### Automatic Key rotation:
- AWS-managed KMS Key: automatic every 1 year 
  - 자동으로 키를 교체하도록 설정 가능
- Customer-managed KMS Key: (must be enabled) automatic every 1 year
  - 무조건 교체되도록 활성화해야 함
  - 활성화한 후에는 자동으로 1년에 한 번씩 교체되며 교체 빈도를 변경할 수는 없음
- Imported KMS Key: only manual rotation possible using alias
## Copying Snapshots across regions
- 올바르게 키를 교체하려면 반드시 KMS 키 별칭을 사용해야 함
- KMS는 리전에 따라 범위가 지정됨
- 이는 KMS 키로 암호화된 EBS 볼륨이 있고 리전은 eu-west-2라고 할 때 다른 리전으로 복사하려면 몇 가지 단계를 거쳐야 함을 의미
  - 먼저 EBS 볼륨의 스냅샷을 생성해야 함
  - 암호화된 스냅샷에서 스냅샷을 생성하면 생성된 스냅샷 또한 동일한 KMS 키로 암호화됨
  - 이제 다른 리전으로 스냅샷을 복사하려면 다른 KMS 키를 사용해서 스냅샷을 다시 암호화해야함
  - 이 부분은 AWS에서 자동으로 처리
  - 다만 동일한 KMS 키가 서로 다른 리전에 있을 수는 없음
  - 스냅샷을 생성. 다른 KSM 키를 사용해 암호화됐고 다른 리전에 있음
  - 이제 KMS로 스냅샷을 자체 EBS 볼륨으로 복원하고 KMS Key B를 ap-southeast-2로 복원
## KMS Key Policies
- KMS 키에 관한 액세스를 제어하는 것으로 S3 버킷 정책과 비슷하지만 차이점이 있음
  - KMS 키에 KMS 키 정책이 없으면 누구도 액세스할 수 없다는 것
### Default KMS Key Policy:
- 사용자 지정 키 정책을 제공하지 않으면 생성됨
- 기본적으로 계정의 모든 사람이 키에 액세스하도록 허용하는 것
### Custom KMS Key Policy:
- Define users, roles that can access the KMS key
- Define who can administer the key
- Useful for cross-account access of your KMS key
## Copying Snapshots across accounts
- Create a Snapshot, encrypted with your own KMS Key (Customer Managed Key)
- Attach a KMS Key Policy to authorize cross-account access
- Share the encrypted snapshot
- (in target) Create a copy of the Snapshot, encrypt it with a CMK in your account

# KMS Multi-Region Keys 다중 리전 키
- AWS KMS에는 다중 리전 키를 둘 수 있음. 선택된 한 리전에 기본 키를 갖는다는 의미
- 가령 us-east-1에 기본 키를 두면 다른 리전으로 복제됨
  - us-west-2, eu-west-1, ap-southeast-2로 복제
- 키 구성 요소가 복제됨
- 다른 리전도 동일한 키를 갖게 되는데 키 ID가 완전히 똑같음
- KMS 다중 리전 키는 다른 AWS 리전에서 사용할 수 있는 KMS 키 세트로 서로 교차해서 사용할 수 있음
- 한 리전에서 암호화한 다음 다른 리전에서 복호화 할 수 있음

# DynamoDB Global Tables and KMS Multi-Region Keys Client-Side encryption
- DynamoDB 전역 테이블과 KMS 다중 리전 키를 클라이언트 측 암호화하는 원리
- 중요한 것은 전체 테이블만 암호화하는 게 아니라는 점
- 저장 데이터 암호화이므로 테이블의 속성을 암호화하여 특정 클라이언트만 사용할 수 있게 됨
- 데이터베이스 관리자도 사용할 수 없음
- 이때 Amazon DynamoDB 암호화 클라이언트를 사용
- 클라이언트 측 암호화 기술을 사용하면 데이터의 특정 필드나 속성을 보호할 수 있음
- 전역 테이블 덕분에 데이터뿐만 아니라 암호화 키도 함께 복제할 수 있음
### 예시 (us-east-1의 데이터를 ap-southeast-2로 복제)
- us-east-1의 KMS 다중 리전 키는 다른 리전에 복제됨 ap-southeast-2
- 클라이언트 애플리케이션에서 DynamoDB에 데이터를 삽입하려면 먼저 속성을 암호화해야 함
- 다중 리전 키를 사용해 암호화할 속성을 암호화
- 대부분의 DynamoDB 테이블 필드는 클라이언트 측 암호화가 되지 않지만 social security number(SSN)는 암호화 될 것.
- DynamoDB 테이블에 액세스할 수 있는 데이터베이스 관리자가 'SSN' 속성을 암호화하는 데 사용한 KMS 키에 액세스할 수 있는 권한이 없다면 액세스할 수 없음
- 데이터베이스 관리자로부터도 보호할 수 있는 것
- DynamoDB 테이블이 전역 테이블인 경우 해당 테이블의 데이터는 다른 리전으로 복제됨 ap-southeast-2
- 다행히 다중 리전 키로 데이터 속성을 암호화하기로 했기 떄문에 ap-southeast-2 리전의 클라이언트 애플리케이션은 열을 검색해서 해당 속성이 암호화됐는지 확인한 후 API 호출을 실행해 복제된 다중 리전 키를 사용해 해당 속성을 복호화함
- ap-southeast-2의 클라이언트 애플리케이션이 KMS로 로컬 API 호출을 보내 해당 속성을 복호화

# Global Aurora and KMS Multi-Region Keys Client-Side encryption
- We can encrypt specific attributes client-side in our Aurora table using the AWS Encryption SDK
- Combined with Aurora Table Tables, the client-side encrypted data is replicated to other regions
- if we use a multi-region key, replicated in the same region as the Global Aurora DB, then clients in these regions can use low-latency API calls to KMS in their region to decrypt the data client-side
- Amazon Aurora DB에 액세스할 수 있는 DB 관리자가 'SSN'열에 액세스하고자 할 때 KMS 키에 대한 액세스 권한이 없으면 해당 데이터를 읽을 수 없음

# S3 Replication Encryption Considerations
- S3 복제와 암호화된 객체의 연관성
- 한 버킷에서 다른 버킷으로 S3 복제를 활성화하면 암호화되지 않은 객체와 SSE-S3로 암호화된 객체가 기본으로 복제됨
- 고객 제공 키인 SSE-C로 객체를 암호화하면 복제되지 않음 (항상 키를 제공해야 하는데 그럴 수 없기 때문)
- For objects encrypted with SSE-KMS
  - 기본적으로 복제되지 않지만 만약 객체를 복제하려면 옵션을 활성화해야 함
  - Specify which KMS Key to encrypt the objects within the target bucket
  - Adapt the KMS Key Policy for the target key
  - S3 복제 서비스를 허용하는 IAM 역할을 생성해서 소스 버킷의 데이터를 먼저 복호화하도록 한 뒤 대상 KMS 키로 대상 버킷의 데이터를 다시 암호화
  - 이렇게 하면 복제가 활성화 -> 이는 수많은 암호화와 복호화가 발생하기 때문
  - KMS 스로틀링 오류가 발생한 경우는 서비스 할당량을 요청해야 함
- You can use multi-region AWS KMS Keys, but they are currently treated as independent keys by Amazon S3
  - the object will still be decrypted and then encrypted

# AMI Sharing Process Encrypted via KMS
- AMI를 다른 계정과 공유하는 과정
- 소스 계정에 있는 AMI는 소스 계정으로부터 KMS Key와 함께 암호화 되어 있음
- 어떤 방식으로 A 계정의 AMI에서 B 계정의 EC2 인스턴스를 시작할까?
  - 먼저 시작 권한으로 AMI 속성을 수정해야 함 - 이 시작 권한은 B 게정에서 AMI를 시작하도록 허용
    - 시작 권한을 수정하고 특정 대상인 AWS 계정 ID를 추가하는 것
  - 그런 다음 B 계정에서 사용하도록 KMS 키도 공유해야 하므로 일반적으로 키 정책으로 실행됨
  - 이제 B 계정에서 KMS 키와 AMI를 모두 사용할 수 있는 충분한 권한을 가진 IAM 역할이나 IAM 사용자를 생성
  - 따라서 DescribeKey API 호출과 ReEncrypted API 호출, CreateGrant, Decrypt API 호출에 관한 KMS 측 액세스 권한이 있어야 함
  - 모두 완료된 후에는 AMI에서 EC2 인스턴스를 시작하면 됨
  - 선택 사항으로 대상 계정에서 자체 계정의 볼륨을 재암호화하는 KMS 키를 이용해 전체를 재암호화할 수 있음

# SSM Parameter Store
- 구성 및 암호를 위한 보안 스토리지
- 구성을 암호화할지 선택할 수 있으므로 KMS 서비스를 이용해 암호를 만들 수 있음
- SSM Parameter Store는 서버리스이며 확장성과 내구성이 있고 SDK도 사용이 용이
- 매개변수를 업데이트할 때 구성과 암호의 버전을 추적할 수도 있음
- IAM을 통해 보안이 제공되며 특정한 경우엔 Amazon EventBridge로 알림을 받을 수도 있음
- CloudFormation과 통합도 됨
  - CloudFormation이 Parameter Store의 매개변수를 스택의 입력 매개변수로 활용할 수 있음
- 애플리케이션과 SSM Parameter Store가 있을 때 평문 구성을 저장할 때
  - EC2 인스턴스 역할 같은 애플리케이션의 IAM 권한이 확인될 것
- 아니면 암호화된 구성이 있을 때
  - SSM Parameter Store가 KMS로 구성을 암호화
- KMS 서비스가 암호화 및 복호화에 사용된다는 뜻
- 물론 암호화와 복호화를 수행하려면 애플리케이션이 기본 KMS 키에 액세스할 수 있어야 함
- 계층 구조가 있는 Parameter Store에 매개변수를 저장할 수 있음
## SSM Parameter Store Hierarchy
- 구조화를 통해 IAM 정책을 간소화하여 애플리케이션이
  - /my-department/ 혹은 my-app/ 전체 경로에 혹은 my-app/ 또는 /my-department/ 환경의 특정 경로에만 액세스할 수 있도록 함
- Amazon Linux 2의 최신 AMI를 찾으려 할 떄 Parameter Store에서 API 호출을 대신해 쓸 수 있음
- 만약 Dev Lambda 함수라는 애플리케이션을 사용한다면 my-app/ 내 dev/의 db-url과 db-password에 액세스를 허용하는 IAM 역할을 할 것
- 여러 IAM 정책과 일부 환경변수 때문에 Prod Lambda 함수를 사용한다면 다른 경로에 있는 prod/의 db-url과 db-password에도 액세스할 수 있음
## Standard and advanced parameter tiers
- 표준
  - 최대 용량: 4KB
  - 매개변수 정책 적용 여부: No
  - Storage Pricing: Free
- 고급
  - 최대 용량: 8KB
  - 매개변수 정책 적용 여부: Yes
  - Storage Pricing: $0.05 per advanced parameter per month
## Parameter Policies (for advanced parameters) 매개변수 정책
- 매개변수 정책을 통해 TTL을 매개변수에 할당할 수 있음
- 사용자가 비밀번호 등의 민감한 정보를 업데이트 또는 삭제하도록 강제
- EventBridge와 통합함으로써 EventBridge에서 알림을 받을 수 있게 됨
### ExpirationNotification (EventBridge)
```json
{
  "Type": "ExpirationNotification",
  "Version": "1.0",
  "Attributes": {
    "Before": "15",
    "Unit": "Days"
  }
}
```

# AWS Secrets Manager
- 암호를 저장하는 최신 서비스
- Secrets Manager는 X일마다 강제로 암호를 교체하는 기능이 있음
- 교체할 암호를 강제 생성 및 자동화할 수 있음
  - 이를 위해 암호를 생성할 Lambda 함수를 정의해야 함
- AWS가 제공하는 서비스와도 통합 가능
  - Amazon RDS, MySQL, PostgreSQL, Aurora 등..
- 데이터베이스에 접근하기 위한 사용자 이름과 비밀번호가 AWS Secrets Manager에 바로 저장되고 교체 등도 가능
- 암호는 KMS 서비스를 통해 암호화됨
- RDS와 Aurora의 통합 혹은 암호에 대한 내용이 시험에 나오면 AWS Secrets Manager를 떠올리기
## AWS Secrets Manager - Multi-Region Secrets
- 복수 AWS 리전에 암호를 복제할 수 있음
- AWS Secrets Manager 서비스가 기본 암호와 동기화된 읽기 전용 복제본을 유지한다는 개념
- 두 리전이 있을 때 기본 리전에 암호를 하나 만들면 보조 리전에 동일한 암호가 복제됨
- A 리전에 문제가 발생하면 암호 복제본을 독립 실행형 암호로 승격할 수 있음
- 여러 리전에 암호가 복제되니 다중 리전 앱을 구축하고 재해 복구 전략도 짤 수 있음
- 한 리전에서 다음 리전으로 복제되는 RDS 데이터베이스가 있다면 동일한 암호로 동일한 RDS 데이터베이스 즉 해당 리전의 해당 데이터베이스에 액세스할 수 있음

# AWS Certificate Manager (ACM)
- ACM은 TLS 인증서를 AWS에서 프로비저닝, 관리 및 배포하게 해줌
- TLS/SSL 인증서는 웹사이트에서 전송 중 암호화를 제공하는 데 쓰임
### 작동원리
- 오토 스케일링 그룹에 연결된 ALB가 있을 때
- 애플리케이션 로드 밸런서(ALB)를 HTTPS 엔드 포인트로서 노출하려 함
- 그러려면 ALB를 AWS Certificate Manager와 통합해 ALB에서 직접 TLS 인증서를 프로비저닝 및 유지관리하도록 하면 됨
- 그러면 사용자가 HTTPS 프로토콜을 사용하는 웹사이트 또는 API에 액세스하게 됨
- ACM은 퍼블릭과 프라이빗 TLS 인증서를 모두 지원하며 퍼블릭 TLS 인증서 사용 시에는 무료로 이용할 수 있음
- 인증서를 자동으로 갱신하는 기능도 있음
- AWS 서비스와 통합되어 있어서 ELB에 TLS 인증서를 불러올 수 있음
- CloudFront 배포나 API Gateway의 모든 API에서도 불러올 수 있음
- EC2 인스턴스에는 ACM을 사용할 수 없음 (퍼블릭 인증서일 경우 추출 불가능)
## ACM - Requesting Public Certificates
1. List domain names to be included in the certificate
2. Select Validation Method: DNS Validation or Email validation
3. It will take a few hours to get verified
4. The Public Certificate will be enrolled for automatic renewal
## ACM - Importing Public Certificates
- ACM에서 퍼블릭 인증서는 어떻게 가져오면 될까
- ACM 외부에서 생성된 인증서를 ACM으로 가져오는 옵션도 제공
- ACM 외부에서 생성되었기 때문에 자동 갱신은 불가능함
- 인증서 만료되기 전에 ACM 서비스가 만료 45일 전부터 매일 만료 이벤트를 EventBridge 서비스에 전송해 줌
- EventBridge에서 Lambda 함수나 SNS 주제 또는 SQS 대기열을 트리거
- 또는 AWS Config을 사용하는 방법도 있음
- Config 서비스에 규칙을 설정하면 Config 서비스가 ACM 서비스를 검사해서 규칙에 위배되는 인증서가 있을 경우 규정 비준수 이벤트가 EventBridge로 전송됨
## ACM - Integration with ALB
- ACM 서비스는 어떻게 ALB와 통합되는 걸까
- 백엔드에 오토스케일링 그룹이 있는 ALB와 ACM으로 프로비저닝 및 유지관리되는 TLS 인증서가 있을 때
- ALB에서는 HTTP에서 HTTPS로의 리디렉션 규칙을 설정할 수 있음
- 사용자가 HTTP 프로토콜에서 애플리케이션 로드밸런서에 액세스할 때 ALB는 HTTPS로 리디렉션하는 요청을 반환
- 그 후 사용자는 HTTPS 프로토콜에서 애플리케이션 로드 밸런서에 액세스하며 AWS Certificate Manager(ACM)에서 나오는 TLS 인증서를 사용함
- 그런 다음 요청이 HTTPS 프로토콜을 통과하면 오토 스케일링 그룹으로 곧바로 전달됨
## API Gateway - Endpoint Types
### Edge-Optimized (default)
- 엣지 최적화 유형
- 이 유형은 글로벌 클라이언트를 위한 유형으로 CloudFront 엣지 로케이션으로 요청을 라우팅하여 지연을 줄이는 방법
- 하나의 리전에만 있는 API Gateway로 보내지는 경우
### Regional
- 리전 엔드 포인트 유형은 클라이언트가 API Gateway와 같은 리전에 있을 때 쓰임
- 이 경우 CloudFront는 사용할 수 없지만 자체 CloudFront 배포를 생성하여 캐싱 및 배포 전략을 제어할 수는 있음
### Private
- 인터페이스 VPC 엔드포인트를(ENI) 통해 VPC 내부에만 액세스할 수 있음
- 또한 프라이빗 모드에서는 API Gateway에 대한 액세스를 정의하는 리소스 정책이 필요함
- ACM은 엣지 최적화 및 리전 엔드포인트에 적합
## ACM - Integration with API Gateway
- ACM을 API Gateway와 통합하려면 우선 API Gateway에 사용자 지정 도메인 이름이라는 리소스를 생성하고 구성해야 함
### Edge-Optimized (default): For global Clients
- 요청이 CloudFront에서 라우팅되고 TLS 인증서가 CloudFront 배포에 연결되기 때문에 TLS 인증서가 반드시 CloudFront와 같은 리전인 us-east-1에 생성되어야 함
- API Gateway가 한 리전에 있고 CloudFront를 통해 배포되는 모든 것과 ACM 인증서는 반드시 us-east-1 리전에 있어야 함
- CloudFront가 us-east-1 리전에 있기 때문에. CloudFront를 위한 인증서는 전부 us-east-1에 있음
- 그리고 Route 53에 CNAME이나 별칭(A-Alias) 레코드를 설정하면 끝
### Regional
- API Gateway와 리전이 같은 클라이언트를 위한 엔드포인트로 API Gateway만 있으므로 TLS 인증서를 API Gateway에 가져올 때는 같은 리전에 속해 있어야 함
- 그리고 Route 53에서 CNAME이나 별칭 레코드가 DNS를 가리키도록 설정하면 됨

# AWS WAF - Web Application Firewall
- AWS 웹 애플리케이션 방화벽
- AWS WAF는 Layer 7에서 일어나는 일반적인 웹 취약점 공격으로부터 웹 애플리케이션을 보호함
- Layer 7 is HTTP (vs Layer 4 is TCP/UDP)
- 배포
  - Application Load Balancer
  - API Gateway
  - CloudFront
  - AppSync GraphQL API
  - Cognito User Pool
## AWS WAF - Web Application Firewall
- 서비스에 방화벽을 배포한 후에는 웹 액세스 제어 목록(ACL)과 규칙을 정의해야 함
- IP Set: up to 10,000 IP address - use multiple Rules for more IPs
- HTTP headers, HTTP body, or URI strings Protects from common attack - SQL injection and Cross-Site Scripting (XSS)
- Size constraints, geo-match (block countries)
- Rate-based rules (to count occurrences of events) - for DDoS protection
- 웹 ACL은 리전에만 적용되며 CloudFront는 글로벌로 정의됨
- 규칙 그룹은 여러 웹 ACL에 추가할 수 있는 재사용 가능한 규칙 모음
## WAF - Fixed IP while using WAF with a Load Balancer
- WAF does not support the Network Load Balancer (Layer 4)
- We can use Global Accelerator for fixed IP and WAF on the ALB
- AWS Global Accelerator로 고정 IP를 할당받은 다음 ALB에서 WAF를 활성화하면 해결할 수 있음
- 웹 애플리케이션 방화벽과 웹 ACL을 연결하는데 애플리케이션 로드 밸런서와 동일한 리전에 배치하면 아키텍처 완성

# AWS Shield: protect from DDoS attack
- 디도스 공격으로부터 스스로를 보호하기 위한 서비스
- 이 서비스는 모든 AWS 고객에서 무료로 활성화되어 있는 서비스로 SYN/UDP Floods나 반사 공격 및 L3/L4 공격으로부터 고객을 보호해 줌
- 고급 보호가 필요한 고객을 위한 AWS Shield 어드밴스드 서비스도 있는데 어드밴스드는 선택적으로 제공되는 디도스 완화 서비스로 조직당 월 3,000달러를 지불해야함
- 어드밴스드는 더욱 정교한 디도스 공격을 막아주고 EC2, ELB, CloudFront, Global Accelerator, Route 53을 보호함
- 어드밴스드는 자동으로 WAF 규칙을 생성, 평가, 배포함으로써 L7 공격을 완화할 수 있음
- 즉 웹 애플리케이션 방화벽이 L7에서 일어나는 디도스 공격을 완화하는 규칙을 자동으로 갖게 된다는 뜻

# AWS Firewall Manager
- AWS Organizations에 있는 모든 계정의 방화벽 규칙을 관리하는 서비스
- 여러 계정의 규칙을 동시에 관리할 수 있음
- 보안 규칙의 집합인 보안 정책을 설정할 수 있는데 ALB, API Gateway, CloudFront 등에 적용되는 웹 애플리케이션 방화벽(WAF) 규칙이나 ALB, CLB, NLB, 엘라스틱 IP, CloudFront를 위한 AWS Shield 어드밴스드 규칙 등이 포함됨
- EC2, 애플리케이션 로드 밸런서 VPC의 ENI 리소스를 위한 보안 그룹을 표준화하는 보안 정책
- AWS Network Firewall (VPC Level)
- Amazon Route 53 Resolver DNS Firewall
- Policies are created at the region level
### 정책은 리전 수준에서 생성되며 조직에 등록된 모든 계정에 적용됨
- 또한 예를 들어 조직에서 애플리케이션 로드 밸런서에 대한 WAF 규칙을 생성한 다음 새 애플리케이션 로드 밸런서를 생성하는 경우 AWS Firewall Manager에서 자동으로 새 ALB에도 같은 규칙을 적용해 줌
## WAF vs Firewall Manager vs Shield
- 모두 포괄적인 계정 정보를 위한 서비스
- 일단 WAF에서는 웹 ACL 규칙을 정의하는데 리소스별 보호를 구성하는 데에는 WAF가 적절
- 하지만 여러 계정에서 WAF를 사용하고 WAF 구성을 가속하고 새 리소스 보호를 자동화하려면 Firewall Manager로 WAF 규칙을 관리하면 됨
- Firewall Manager는 이러한 규칙들을 모든 계정과 모든 리소스에 자동으로 적용해줌
- Shield 어드밴스드는 디도스 공격으로부터 고객을 보호하고 WAF의 기능 외에도 더 많은 기능을 제공함
  - Shield 대응 팀 지원 고급 보고서 제공
  - WAF 규칙 자동 생성 등의 기능을 추가로 이용할 수 있음
  - 디도스 공격을 자주 받는다면 고려
- Firewall Manager는 모든 계정에 Shield 어드밴스드 배포에 도움

# AWS Best Practices for DDoS Resiliency Edge Location Mitigation (BPI, BP3)
### BP1 - CloudFront
- Web Application delivery at the edge
- Protect from DDoS Common Attacks (SYN floods, UDP, reflection...)
### BP1 - Global Accelerator
- 전 세계에서 엣지를 통해 애플리케이션에 액세스할 수 있음
- Global Accelerator는 Shield와 완전히 통합되어 CloudFront가 백엔드와 호환되지 않는 경우 DDoS 공격 방어에 유용하게 쓰임
- 이 경우 Global Accelerator를 앞에 두는데 이렇게 하면 어떤 백엔드를 사용하든 CloudFront나 Global Accelerator로 AWS 엣지에 완전 분산이 가능함
- 엣지 로케이션을 DDoS 공격으로부터 보호할 수 있음
### BP3 - Route 53
- 엣지에 도메인 이름 변환을 글로벌로 설정
- DNS에도 DDoS 보호 메커니즘을 적용할 수 있음
- 엣지에 대한 DDoS 보호를 더 확실히 할 수 있음

# AWS Best Practices for DDoS Resiliency Best practices for DDoS mitigation
- DDoS 완화 모범 사례
### Infrastructure layer defense (BP1, BP3, BP6)
- CloudFront, Global Accelerator, Route 53, Elastic Load Balancing
- 높은 트래픽으로부터 Amazon EC2 인스턴스를 보호함
- EC2 인스턴스에 도달하기도 전에 트래픽을 관리할 수 있음
### Amazon EC2 with Auto Scaling (BP7)
- 오토 스케일링 그룹에 트래픽이 도달한다고 해도 자동으로 확장하여 애플리케이션에서 더 큰 로드를 수용할 수 있음
### Elastic Load Balancing (BP6)
- 엘라스틱 로드 밸런싱을 사용하는 경우에는 ELB가 여러 EC2 인스턴스 간 트래픽을 자동으로 분산시킴
- EC2 인스턴스에 관리 가능한 양의 트래픽이 들어오면서 오토 스케일링 그룹이 이에 따라 확장하는 데에도 무리가 없음

# AWS Best Practices for DDoS Resiliency Application Layer Defense
- 애플리케이션 계층 방어의 예시
### Detect and filter malicious web requests (BP1, BP2)
- CloudFront는 정적 콘텐츠 전송 시 엣지 로케이션에서 전송함으로써 백엔드를 보호함
- 애플리케이션 로드 밸런서나 CloudFront에 WAF를 사용하여 요청 서명에 따라 요청을 필터링 및 차단할 수 있음
- 특정 IP나 특정 요청 유형만 차단할 수도 있음
- WAF 속도 기반 규칙을 사용하면 악성 사용자의 IP를 자동으로 차단할 수 있음
- WAF에 여러 관리형 규칙을 사용하면 평판에 따라 IP를 차단하거나 익명 IP등을 차단할 수 있음
- CloudFront로는 특정 지역을 차단할 수 있음
- CloudFront와 WAF는 관리형 서비스로 요청을 필터링 해줌 -> DDoS 공격으로부터 보호
### Shield Advanced (BP1, BP2, BP6)
- 이 서비스를 활성화하면 자동으로 WAF 규칙을 생성하여 계층 7 공격을 완화함
- 애플리케이션 계층 방어에 유용
- 악성 요청이 들어오지 못하도록 하거나 그 수를 최소화하는 식으로 EC2 인스턴스를 보호할 수 있음

# AWS Best Practices for DDoS Resiliency Attack surface reduction
- 공격 지점은 어떻게 줄일 수 있을까
### Obfuscating AWS resources (BP1, BP4, BP6)
- CloudFront, API Gateway 엘라스틱 로드 밸런싱을 사용하면 백엔드 리소스를 숨길 수 있음
- 공격자는 이 리소스가 Lambda 함수인지 EC2 인스턴스나 ECS 태스크인지 알 수 없음
### Security groups and Network ACLs (BP5)
- 또한 보안 그룹과 네트워크 ACL 등을 설정하여 특정 IP의 트래픽을 필터링할 수 있음
- Elastic IP(탄력적 IP)도 AWS Shield Advanced로 보호할 수 있음
### Protecting API endpoints (BP4)
- 엔드 포인트 자체도 보호할 수 있음
- API Gateway를 사용하면 어떤 백엔드든 숨길 수 있음
- EC2, Lambda 등 무엇인지 알 수 없도록
- 엣지 최적화 모듈을 사용할 경우 이미 글로벌로 설정되어 있음
- CloudFront에 리전 모드를 더해 사용한다면 DDoS 보호에 관한 제어 기능이 더 강화됨
- API Gateway에 WAF를 사용하는 경우 모든 HTTP 요청을 필터링할 수 있음
- API Gateway를 제대로 설정했다면 버스트 제한과 헤더 필터링을 할 수 있음
- 사용자에게 API 키 사용을 강제할 수도 있음
### 304. DDoS Protection Best Practices 강의 보기

# Amazon GuardDuty
- AWS 계정을 보호하는 지능형 위협 탐지 서비스
- 암호화폐 공격을 보호할 수 있음. 관련된 전요 탐지가 있음
- GuardDuty는 VPC 흐름 로그, CloudTrail 로그, DNS 로그 및 EKS 감사 로그를 모두 GuardDuty로 가져 옴
- CloudWatch 이벤트 규칙 덕분에 Lambda 함수나 SNS 주제 알림을 받을 수 있음

# Amazon Inspector
- 자동화된 보안 평가를 실행할 수 있는 서비스
- 의도되지 않은 네트워크 접근 가능성에 대해 분석하고 실행 중인 운영 체제에서 알려진 취약점을 분석 (연속적으로 수행)
- Amazon Inspector는 컨테이너 이미지를 Amazon ECR로 푸시할 때 실행됨
- 컨테이너 이미지가 ECR로 푸시되면 Inspector가 알려진 취약점에 대해 검사함
- Lambda 함수가 배포될 때 함수 코드 및 패키지 종속성에서 소프트웨어 취약성을 다시 분석함
- 중요) EC2 인스턴스, Amazon ECR의 컨테이너 이미지, Lambda 함수에만 사용됨
- 취약성 데이터베이스, CVE를 살펴볼 것임

# Amazon Macie
- 완전 관리형의 데이터 보안 및 데이터 프라이버시 서비스
- 머신 러닝과 패턴 일치를 사용하여 AWS의 민감한 정보를 검색하고 보호
- 민감한 데이터를 경고하는데 예를 들면 개인 식별 정보 같은 것들
- S3 Buckets -> Macie analyze -> CloudWatch Events EventBridge notify