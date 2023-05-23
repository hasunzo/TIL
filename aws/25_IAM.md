# AWS Organizations
- 글로벌 서비스로 다수의 AWS 계정을 동시에 관리할 수 있게 해줌
- 조직을 생성하면 조직의 메인 계정이 관리 계정
- 조직에 가입한 기타 계정이나 조직에서 생성한 계정은 멤버 계정이라고 부르며 멤버 계정은 한 조직에만 소속됨
- Consolidated Billing across all accounts - single payment method
- Pricing benefits from aggregated usage (volume discount for EC2, S3...)
- 계정 간에 예약 인스턴스와 Savings Plans 할인이 공유됨
- API is available to automate AWS account creation
## Organizational Units (OU) - Examples
- 비즈니스 단위에 따라 OU를 조직하는 경우
  - 관리하는 계정 아래에 세일즈, 리테일 및 재무 OU를 두고 OU별로 멤버 계정을 생성할 수 있음
- Environmental Lifecycle
  - 프로덕션 OU, 테스트 OU, 개발 OU를 만들고 각 OU에 계정을 만들기
- Project-Based
  - 프로젝트별 OU와 멤버 계정을 생성하는 프로젝트 기반 OU도 가능
## Advantages
- 다수의 계정을 가지므로 다수의 VPC를 가진 단일 계정에 비해 보안이 뛰어남
- 계정이 VPC보다 독립적이기 때문
- 청구 목적의 태그 기준을 적용할 수 있음
- 한 번에 모든 계정에 대해 CloudTail을 활성화해서 모든 로그를 중앙 S3 계정으로 전송할 수 있음
- CloudWatch Logs를 중앙 로깅 계정으로 전송할 수 있음
- 자동으로 관리 목적의 계정 간 역할을 수립할 수 있음
- 관리 계정에서 모든 멤버 계정을 관리할 수 있음
## Security: Service Control Policies (SCP)
- 특정 OU 또는 계정에 적용되는 IAM 정책으로 해당 사용자와 역할 모두가 계정 내에서 할 수 있는 일을 제한
- SCP는 모든 대상에 적용되나 영구적인 관리자 권한을 갖는 관리 계정에는 적용되지 않음
- SCP는 기본적으로 아무것도 허용하지 않음. 구체적인 허용 항목을 설정해야 작동
## SCP Hierarchy
- 일반적으로 루트 OU에는 FullAWSAccess(전체 AWS 액세스)SCP를 적용
- 따라서 루트 OU 내의 게정은 모든 계정에 대한 전체 권한을 가짐
- 관리 계정 프로덕션 계정의 계정 A, 계정 B, 계정 C에 대한 모든 권한을 가짐
- Root OU - FullAWSAccess SCP
- Management Account - DenyAccessAthena SCP
- Prod - DenyRedshift SCP
- 정책에 명시적 거부가 하나라도 포함되면 기본적으로 액세스가 거부됨
## SCP Examples Blocklist and Allowlist strategies
- SCP에는 '차단 목록'과 '허용 목록'두 가지 전략이 있음
- 차단 목록은 계정이 사용하지 못하게 할 서비스를 지정함
  - Allow, *(별표)로 모든 서비스의 모든 작업을 허용한 다음 Deny 문을 추가해 DynamoDB에 대한 액세스를 거부하는 식
- 허용 목록은 모든 액션을 허용하지 않고 가령 EC2와 CloudWatch만 허용하는 방식
  - 해당 SCP가 적용된 계정에서는 EC2와 CloudWatch만이 허용되며 다른 서비스는 사용할 수 없고 사용하려면 명시적 허용이 필요함

# IAM Conditions
- IAM 조건은 IAM 내부의 정책에 적용되며 사용자 정책일 수도 있고 예를 들면 S3 버킷에 대한 리소스 정책일 수 있음
- 엔드포인트 정책 등 어떤 정책도 될 수 있음
### aws:Sourcelp
- API 호출이 생성되는 클라이언트 IP를 제한하는 데 사용되는 조건
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "NotIpAddress": {
          "aws:SourceIp": ["192.0.2.0/24", "203.0.113.0/24"] 
        }
      }
    }
  ]
}
```
- 정책을 보면 모든 작업과 리소스를 거부(Deny)하지만 조건이 있음
- 두 개의 IP 주소 범위에 포함되지 않는 IP 주소는 거부
- 이는 클라이언트가 해당 IP 주소 내에서 API 호출을 하지 않으면 API 호출이 거부된다는 뜻
- 회사 네트워크에서만 AWS를 사용하도록 제한하는 등..
### aws:RequestedRegion
- AWS로 시작하므로 글로벌 조건이며 API 호출 리전을 제한
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": ["ec2:*", "rds:*", "dynamodb:*"],
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "aws:requestedRegion": ["eu-central-1", "eu-west-1"]
        }
      }
    }
  ]
}
```
- eu-central-1와 eu-west-1 리전에서 오는 EC2, RDS, DynamoDB 호출을 제한
- 이 조건을 조직 SCP에 보다 글로벌하게 적용해서 특정 리전의 액세스를 허용하거나 거부할 수 있음
### ec2:ResourceTag
- 태그의 접두사가 EC2. EC2 인스턴스의 태그에 적용됨
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["ec2:startInstances", "ec2:StopInstances"],
      "Resource": "arn:aws:ec2:us-east-1:123456789012:instance/*",
      "Condition": {
        "StringEquals": {
          "ec2:ResourceTag/Project": "DataAnalytics",
          "aws:PrincipalTag/Department": "Data"
        }
      }
    }
  ]
}
```
- ResourceTag/Project가 DataAnalytics일 때 모든 인스턴스의 시작과 종료를 허용
- EC2 인스턴스가 올바른 태그를 갖고 있으면, 즉 Project가 DataAnalytics이면 허용됨
- 다음 줄의 aws:PrincipleTag는 사용자 태그에 적용
  - EC2 인스턴스 태그가 아닌 사용자 태그
### aws:MultiFactorAuthPresent
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "ec2:*",
      "Resource": "*"
    },
    {
      "Effect": "Deny",
      "Action": ["ec2:StopInstances", "ec2:TerminateInstances"],
      "Resource": "*",
      "Condition": {
        "BoolIfExists": {
          "aws:MultiFactorAuthPresent": false
        }
      }
    }
  ]
}
```
- EC2에서 모든 작업을 할 수 있지만 멀티팩터 인증이 있어야만 인스턴스를 중지하고 종료할 수 있음
- MFA가 false(거짓)일 떄 거부
# IAM for S3
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["ec2:ListBucket"],
      "Resource": "arn:aws:s3:::test"
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:GetObject",
        "s3:DeleteObject"
      ],
      "Resource": "arn:aws:s3:::text/*"
    }
  ]
}
```
- 첫 번째 선언목은 목록 버킷이고 s3:::test라는 ARN에 적용
- 버킷 수준의 권한이므로 버킷을 특정해야 함
- GetObject, PutObject, DeleteObject는 버킷 내의 객체에 적용되므로 ARN이 달라짐
  - 버킷 내의 모든 객체를 나타내는 /*이 들어감
  - 객체 수준 권한이기 때문에 ARN이 달라진 것
# Resource Policies & aws:PrincipalOrgID
- aws:PrincipalOrgID라는 조건은 AWS 조직의 멤버 계정에만 리소스 정책이 적용되도록 제한됨
- 다수의 계정이 포함된 조직이 있고 아래와 같은 정책이 적용된 S3 버킷이 있음
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:PutObject", "s3:GetObject"],
      "Resource": "arn:aws:s3:::2022-financial-data/*",
      "Condition": {
        "StringEquals": {
          "aws:PrincipalOrgID": ["o-yyyyyyyyy"]
        }
      }
    }
  ]
}
```
- PrincipleOrgID의 게정에서 API 호출이 생성된 경우에만 PutObject, GetObject 작업을 할 수 있음
- 조직 내의 멤버 계정만 S3 버킷에 액세스할 수 있고 조직 외부의 사용자는 이 S3 버킷 정책에 의해 엑세스가 거부됨
## IAM Roles vs Resource Based Policies
### Cross account 교차 계정에서 API 호출을 수행하는 방법:
- S3 버킷에 S3 정책을 적용하는 것처럼 리소스에 리소스 기반 정책을 추가하거나
- 리소스에 액세스할 수 있는 역할을 사용
- 예시
  - 계정 A의 사용자가 Amazon S3 버킷에 대한 액세스 권한이 있는 계정 B의 역할을 사용
    - IAM 역할이 S3 버킷에 액세스
  - 계정 A의 사용자가 계정 B의 Amazon S3 버킷에 적용된 버킷 정책을 통해 S3 버킷에 액세스
    - S3 버킷 정책이 사용자의 액세스를 허용
## IAM Roles vs Resource-Based Policies
- 역할을 맡으면 기존의 권한을 모두 포기하고 해당 역할에 할당된 권한을 상속
  - 예를 들어 어떤 작업을 수행하는 역할을 맡게 되면 해당 역할에 부여된 작업은 할 수 있지만 기존의 권한은 사용할 수 없음
- 리소스 기반 정책을 사용하면 본인이 역할을 맡지 않으므로 권한을 포기할 필요가 없음
  - 예를 들어 계정 A의 사용자가 본인 계정의 DynamoDB 테이블을 스캔해 계정 B의 S3 버킷에 넣을 경우에는 리소스 기반 정책을 사용하는 것이 좋음
  - 역할을 맡을 필요가 없으므로 DynamoDB 테이블을 스캔하고 다른 계정의 S3 버킷에 쓰기 작업도 할 수 있기 때문
- Supported by: Amazon S3 buckets, SNS topice, SQS queues, etc...
## Amazon EventBridge - Security
- Amazon EventBridge에는 원하는 작업을 하려면 대상에 대한 권한을 요구하는 규칙이 있음
- Resource-based policy: Lambda, SNS, SQS, CloudWatch Logs, API Gateway ...
- IAM role: Kinesis stream, Systems Manager RUn Command, ECS task...

# IAM Permission Boundaries 권한 경계
- 사용자와 역할만 지원하고 그룹은 지원하지 않음
- IAM 개체의 최대 권한을 정의하는 고급 기능
### IAM Permission Boundary
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:*",
        "cloudwatch:*",
        "ec2:*"
      ],
      "Resource": "*"
    }
  ]
}
```
- S3, CloudWatch, EC2를 모두 허용하고 있음
- 이걸 IAM 사용자에 연결하면 권한 경계를 설정하는 것
- S3, CloudWatch, EC2 내의 작업만 할 수 있게 됨
### IAM Permissions Through IAM Policy
```json
{
  "Version": "2012-10-17",
  "Statement": {
      "Effect": "Allow",
      "Action": "iam:CreateUser",
      "Resource": "*"
    }
}
```
- 더불어 IAM 정책을 통해 IAM 권한을 지정해야 함
- iam:CreateUser 액션과 * 리소스를 같은 사용자에 연결하면 권한 경계와 IAM 정책을 통한 권한이 완성됨
### 결과는?
- 아무 권한도 생기지 않음
- IAM 정책은 IAM 권한 경계 밖에 있기 때문에 사용자가 다른 IAM 사용자를 생성할 수 없음
- IAM 권한 경계는 AWS Organizations SCP와 함께 사용될 수 있음
### Use cases
- 관리자가 아닌 사용자에 책임을 위임하기 위해 권한 경계 내에서 새 IAM 사용자를 생성하거나 개발자가 권한을 스스로 부여하고 관리하는데 특권을 남용하는 것을 막기 위해 스스로를 관리자로 만들지 못하게 할 수 있음
- 계정에 SCP를 적용해서 계정의 모든 사용자를 제한하지 않고 AWS Organizations의 특정 사용자만 제한할 수 있음
## IAM Policy Evaluation Logic
1. Deny evaluation 명시적 거부
2. Organizations SCPs
3. Resource-based policies 리소스 기반 정책
4. Identity-based policies 자격 증명 기반 정책
5. IAM permissions boundaries IAM 권한 경계
6. Session policies 세션 정책
## Example IAM Policy
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": "sqs:*",
      "Effect": "Deny",
      "Resource": "*"
    },
    {
      "Action": [
        "sqs:DeleteQueue"
      ],
      "Effect": "Allow",
      "Resource": "*"
    }
  ]
}
```

# Amazon Cognito
- Cognito는 사용자에게 웹 및 모바일 앱과 상호 작용할 수 있는 자격 증명을 부여
- 일반적으로 이 사용자들은 AWS 계정 외부에 있음
- 모르는 사용자들에게 자격 증명을 부여해 사용자를 인식(Cognito) 함
### Cognito User Pools:
- 앱 사용자에게 가입 기능을 제공
- API Gateway 및 애플리케이션 로드 밸런서와 원활히 총합
### Cognito Identity Pools (Federated Identity):
- 앱에 등록된 사용자에게 임시 AWS 자격 증명을 제공해서 일부 AWS 리소스에 직접 액세스할 수 있도록 해줌
- Cognito 사용자 풀과 원활히 통합됨
### Cognito vs IAM:
- "수백 명의 사용자", "모바일 사용자", "SAML을 통한 인증"같은 키워드가 나오면 Cognito
## Cognito User Pools (CUP) - User Features
- 웹 및 모바일 앱을 대상으로 하는 서버리스 사용자 데이터베이스
- 사용자 이름 또는 이메일, 비밀번호의 조합으로 간단한 록인 절차를 정의
- 비밀번호 재설정 기능, 이메일 및 전화번호 검증 및 사용자 멀티팩터 인증 가능
- 소셜 로그인 가능
## Cognito User Pools (CUP) - Integrations
- Cognito 사용자 풀은 API Gateway나 애플리케이션 로드 밸런서와 통합
- API Gateway 예시
  - 사용자는 Cognito 사용자 풀에 접속해서 토큰을 받고 검증을 위해 토큰을 API Gateway에 전달
  - 확인이 끝나면 사용자 자격 증명으로 변환되어 백엔드의 Lambda 함수로 전달
  - Lambda 함수는 처리할 사용자가 잘 인증된 구체적인 사용자라는 사실을 인식
- Cognito 사용자 풀을 애플리케이션 로드 밸런서 위에 배치하는 예시
  - 애플리케이션이 Cognito 사용자 풀에 연결한 다음 애플리케이션 로드 밸런서에 전달해서 유효한 로그인인지 확인
  - 유효하다면 요청을 백엔드로 리다이렉트하고 사용자의 자격 증명과 함께 추가 헤더를 전송
  - API Gateway나 ALB를 통해 사용자의 로그인을 한곳에서 확실히 검증할 수 있고 
  - 검증 책임을 백엔드로부터 백엔드의 로드를 밸런싱하는 실제 위치, 즉 API gateway 또는 ALB로 옮긴 것
## Cognito Identity Pools (Federated Identities) 자격 증명 풀
- 사용자에게 자격 증명을 제공하지만 API Gateway나 애플리케이션 로드 밸런서를 통해서 애플리케이션에 액세스하지 않고 
- 임시 AWS 자격 증명을 사용해 AWS 계정에 직접 액세스함
- 따라서 사용자는 Cognito 사용자 풀 내의 사용자가 될 수도 있고 타사 로그인이 될 수도 있음
- 직접 또는 API Gateway를 통해 서비스에 액세스할 수도 있음
- 자격 증명에 적용되는 IAM 정책은 Cognito 자격 증명 풀 서비스에 사전 정의되어 있음
- user_id를 기반으로 사용자 정의하여 세분화된 제어를 할 수도 있음
- 원한다면 기본 IAM 역할을 정의할 수도 있음
- 게스트 사용자나 특정 역할이 정의되지 않은 인증된 사용자는 기본 IAM 역할을 상속하게 됨
## Cognito Identity Pools 과정
- 웹이나 모바일 애플리케이션에서 S3 버킷 또는 DynamoDB 테이블에 직접 액세스하고 싶을 때 Cognito 자격 증명 풀 사용
1. 웹, 모바일 앱이 로그인해서 토큰을 받음
   - Cognito 사용자 풀에 대한 로그인일 수도 있고 소셜 자격 증명 제공자나 SAML, OpenID Connect 등등..
2. 토큰을 Cognito 자격 증명 풀 서비스에 전달해서 토큰을 임시 AWS 자격 증명과 교환
3. Cognito 자격 증명 풀은 전달 받은 토큰이 올바른지, 즉 유효한 로그인인지 평가
4. 해당 사용자에게 적용되는 IAM 정책을 생성
- 관련 IAM 정책이 적용된 임시 자격 증명 덕분에 AWS의 S3 버킷이나 DynamoDB 테이블에 직접 액세스할 수 있는 것
  - API Gateway나 애플리케이션 로드 밸런서를 거치지 않고도
## Cognito Identity Pools Row Level Security in DynamoDB
- DynamoDB에서 행 수준 보안을 활성화할 수 있음

# AWS IAM Identity Center (successor to AWS Single Sign-On)
### One login (single sign-on) for all your
- 한 번만 로그인
- AWS 조직이나 비즈니스 클라우드 애플리케이션의 모든 AWS 계정에서 싱글 사인온을 할 수 있음
- EC2 Windows 인스턴스에 대해서도 싱글 로그인을 제공
- 시험에서는 여러 AWS 계정을 한 번에 로그인하는 것에 대해 나옴
### Identity Providers
- 이 로그인을 위해 사용자는 두 위치에 저장될 수 있음
- IAM 자격 증명 센터에 내장된 ID 저장소
- 서드파티 ID 공급자: Active Directory (AD), OneLogin, Okta...
## 작동 과정
1. 브라우저 인터페이스는 AWS IAM 자격 증명 센터의 로그인 페이지에 연결
2. 다른 사용자 스토어와 통합
  - 클라우드 또는 온프레미스에 있는 Active Directory 
  - 여기에서 Active Directory를 사용하여 사용자와 그룹을 관리할 수 있음
  - 또는 IAM 자격 증명 센터 사용: 여기는 빌트인 자격 증명 스토어. 사용자나 그룹을 정의
3. ID 센터를 AWS 조직이나 Windows EC2 인스턴스, 비즈니스 클라우드 애플리케이션, 사용자 정의 SAML2.0 지원 애플리케이션 등과 통합
## AWS IAM Identity Center Find-grained Permissions and Assignments
### Multi-Account Permissions 다중 계정 권한
- 조직에서 여러 계정에 대한 액세스를 관리할 수 있음
- 권한 셋을 사용하여 사용자를 그룹에 할당하는 하나 이상의 IAM 정책을 정의
  - AWS에서 사용자가 무엇에 액세스할 수 있는지 정의하기 위해서 
### Application Assignments 애플리케이션 할당
- 어떤 사용자 또는 그룹이 어떤 애플리케이션에 액세스할 수 있는지 정의할 수 있음
- 필요한 URL, 인증서, 메타데이터가 제공. 기본적으로 지원됨
### Attribute-Based Access Control (ABAC) 속성 기반 액세스 제어
- IAM 자격 증명 센터 스토어에 저장된 사용자 속성을 기반으로 세분화된 권한을 갖게 되는 것
- 태그 같은 걸 갖게 되는 것
- 이를 사용하여 사용자를 비용 센터에 할당하거나 주니어나 시니어와 같은 직함을 주거나 로케일을 줘서 특정 리전에만 액세스하도록 할 수 있음
- 사용 사례는 IAM 권한 셋을 한 번만 정의하고 이 속성을 활용하여 사용자 또는 그룹의 AWS 액세스만 수정하는 것.

# What is Microsoft Active Directory (AD)?
- AD 도메인 서비스를 사용하는 모든 Windows 서버에 사용되는 소프트웨어
- 객체의 데이터베이스이며 사용자 계정, 컴퓨터, 프린터, 파일 공유, 보안 그룹이 객체가 될 수 있음
- 전체 Microsoft 생태계에서 관리되는 온프레미스의 모든 사용자는 Microsoft Active Directory의 관리 대상이 됨
- 또한 중앙 집중식 보안 관리로 계정 생성, 권한 할당 등의 작업이 가능
- 모든 객체는 트리로 구성되며 트리의 그룹을 포리스트라고 함
- Objects are organized in trees
- A group of trees is a forest
- 예시
  - 도메인 컨트롤러에 계정 생성
  - 사용자 이름은 John 비밀번호는 Password
  - 다른 모든 Windows 머신은 동일한 네트워크에 있고 도메인 컨트롤러에 연결됨
  - 첫 번쨰 머신에서 John과 Password를 사용하면 이 머신은 로그인 정보가 있는지 컨트롤러를 확인한 다음 로그인 허용
  - 모든 머신은 도메인 컨트롤러에 연결되어 있으므로 사용자는 어떤 단일 머신에서도 액세스할 수 있음
# AWS Directory Services
- AWS에 액티브 디렉터리를 생성하는 서비스
### AWS Managed Microsoft AD
- AWS에 자체 액티브 디렉터리를 생성
- 로컬에서 관리할 수 있으며 멀티팩터 인증을 지원함
- 독립 실행형 Active Directory로 사용자가 있는 온프레미스 AD와 신뢰 관계 구축
- AWS 관리형 AD의 인증된 사용자가 AWS 관리형이 아닌 계정을 사용할 때 온프레미스 AD에서 계정을 검색할 수 있고
- 반대로 온프레미스 AD 사용자가 AWS 계정을 사용해 온프레미스 AD에서 인증하려 할 떄도 신뢰 관계에 의거해 AWS AD에서 검색할 수 있음
### AD Connector
- 디렉터리 게이트웨이 = 프록시
- 온프레미스 AD에 리다이렉트하며 MFA(멀티팩터 인증)을 지원함
- 사용자는 온프레미스 AD에서만 관리
- 이름처럼 연결하는 역할을 함
### Simple AD
- AWS의 AD 호환 관리형 디렉터리로 Microsoft 디렉터리를 사용하지 않으며 온프레미스 AD와도 결합되지 않음
- 온프레미스 AD가 없으나 AWS 클라우드에 액티브 디렉터리가 필요한 경우 독립형인 Simple AD 서비스를 사용함
- 액티브 디렉터리를 사용해 Windows를 실행할 EC2 인스턴스를 생성하고 이 Windows 인스턴스는 네트워크용 도메인 컨트롤러와 결합되어 모든 로그인 정보와 자격 증명 등을 공유함
- 이런 이유로 AWS에 디럭터리를 둠
- Windows를 실행하는 EC2 인스턴스와 디렉터리를 가까이 위치시키기 위함
## IAM Identity Center - Active Directory Setup
- IAM Identity Center와 액티브 디렉터리 통합 방법
### Connect to and AWS Managed Microsoft AD (Directory Service)
- Directory 서비스를 통해 AWS 관리형 AD 연결할 경우의 통합방법
- IAM Identity Center를 AWS 관리형 Microfost AD에 통합하고 연결하도록 설정
### Connect to a Self-Managed Directory
- 액티브 디렉터리를 클라우드에서 관리하지 않고 온프레미스에 자체 관리형 디렉터리가 있는 경우엔
- AWS 관리형 Microsoft AD를 사용해 양방향 신뢰 관계를 구축
  - Directory 서비스를 이용하여 관리형 Microsoft AD를 생성하고 온프레미스에 있는 AD와 관리형 AD 간에 양방향 신뢰 관계를 구축한 다음
  - IAM Identity Center에서 싱글 사인온으로 간단히 통합하면 됨
- AD 커넥터를 활용
  - AD 커넥터는 IAM Identity Center와 연결하는 역할을 하고 연결한 다음에는 자동으로 모든 요청을 자체 디렉터리로 프록시 함
- 클라우드에 있는 액티브 디렉터리에서 사용자를 관리하고 싶다면 첫번째 솔루션
- API 호출만 프록시하려면 두번째 솔루션 (지연시간이 길어지긴 함)

# AWS Control Tower
- AWS Control Tower 서비스를 사용하면 모범 사례를 기반으로 안전하고 규정을 준수하는 다중 계정 AWS 환경을 손쉽게 설정하고 관리할 수 있음
- 다중 계정을 생성하기 위해 Control Tower 서비스는 AWS Organization 서비스를 사용해 계정을 자동 생성함
### Benefits:
- 클릭 몇 번으로 환경을 자동으로 설정
- 원하는 모든 것을 미리 구성 가능
- 가드레일을 사용해 자동으로 지속적인 정책 관리
- 정책 위반을 감지하고 자동으로 교정
- 대화형 대시보드를 통해 모든 계정의 전반적인 규정 준수를 모니터링
## AWS Control Tower - Guardrails
- AWS에서 다중 계정을 설정할 때 특정 항목에 관해 한 번에 모두 제한하거나 특정 유형의 규정 준수 사항을 모니터링하고자 한다
  - 이때 가드레일을 사용하면 Control Tower 환경 내의 모든 계정에 관한 거버넌스를 얻을 수 있음
### Preventive Guardrail 예방 가드레일
- 계정을 무언가로부터 보호하는 것이므로 제한적이기 때문에 AWS Organization 서비스의 서비스 제한 정책인 SCP를 사용해 모든 계정에 적용함
- 예를 들어, 예방 가드레일을 생성해 모든 계정에서 리전을 제한하고 특정 리전에서만 작업하도록 할 수 있음
- Control Tower에서 Organization에 SCPs를 사용하도록 하는 것
### Detective Guardrail 탐지 가드레일
- 규정을 준수하지 않는 것을 탐지하는 것
- AWS Config 서비스를 사용함
- 계정에서 태그가 지정되지 않은 리소스를 식별한다고 하면 Control Tower로 탐지 가드레일을 설정하고 AWS Config를 사용하면 
- 모든 멤버 계정에 Config가 배포되어 규칙과 태그가 지정되지 않은 리소스를 모니터링
- 규정을 준수하지 않으면 SNS 주제를 트리거해서 관리자로 알림을 받거나 SNS 주제도 Lambda 함수를 실행해 Lambda 함수가 자동으로 문제를 교정함
- 태그가 지정되지 않은 리소스에 태그를 추가하도록