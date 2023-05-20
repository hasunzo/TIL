# AWS Lambda
- 람다는 가상의 함수
- 관리할 서버 없이 코드를 프로비저닝하면 함수가 실행됨
- 온디맨드로 실행됨
- Lambda를 사용하지 않으면 람다 함수가 실행되지 않고 비용 역시 함수가 실행되는 동안만 청구됨
- 스케일링이 자동화
- 더 많은 람다 함수를 통시에 필요로 하는 경우 AWS가 자동으로 프로비저닝해서 람다 함수를 늘려줌
## Lambda 장점
- 가격 책정이 쉬워짐
  - Lambda가 수신하는 요청의 횟수에 따라 과금됨
  - 호출 횟수와 컴퓨팅 시간, 즉 Lambda가 실행된 시간만큼 청구됨
  - 프리티어 : Lambda 요청 1백만 건과 40만 GB-초의 컴퓨팅 시간이 포함
- 여러 가지 프로그래밍 언어를 사용할 수 있음
- CloudWatch 모니터링 통합도 쉬운 편
- 함수당 더 많은 리소스를 프로비저닝 하려면 함수당 최대 10GB의 램을 프로비저닝 할 수 있음
  - 만약 함수의 RAM을 증가시키면 CPU 및 네트워크 품질과 성능도 함께 향상될 것
## Lambda 지원하는 언어
- JavaScript의 Node.js
- Python
- Java (Java 8 compatible)
- C# (.NET Core)
- Golang
- C# / Powershell
- Ruby
- Custom Runtime API (Community supported, example Rust)
- Lambda Container Image
  - 컨테이너 이미지 자체가 Lambda의 런타임 API를 구현해야 함
  - 즉 아무 컨테이너 이미지나 Lambda에서 실해되지는 않고 컨테이너 이미지를 만들 때 전제 조건이 필요함
  - ECS와 Fargate는 계속 임의의 도커 이미지를 실행할 때 더 많이 사용됨
  - Lambda에 컨테이너를 실행해야 할 경우 해당 컨테이너가 Lambda 런타임 API를 구현하지 않으면 ECS나 Fargate에서 컨테이너를 실행해야 함
## AWS Lambda Integrations Main ones
- API Gateway는 REST API를 생성하고 람다 함수 호출
- Kinesis는 Lambda를 이용해 바로 데이터를 변환
- DynamoDB는 트리거를 생성할 때 사용되는데 데이터베이스에 어떤 일이 생기면 람다 함수가 작동되도록 함
- Amazon S3는 언제든 람다 함수를 작동시킴
  - S3에 파일이 생성될 때 등등
- CloudFront용 람다는 Lambda@Edge
- CloudWatch 이벤트와 EventBridge에서는 AWS의 인프라에 어떤 일이 생기고 그 상황에 대응하고자 할 때 상황에 따라 자동화를 실행하려고 람다 함수를 씀
  - 파이프라인이 끊기거나, 상태가 바뀌는 경우 등
- CloudWatch 로그는 어디든 해당 로그를 스트리밍 함
- SNS로 알림과 SNS 토픽에 대처할 수 있음
- SQS로는 SQS 대기열 메시지를 처리할 수 있음
- Cognito는 사용자가 데이터베이스에 로그인할 때마다 응답
## AWS Lambda Example
### 섬네일 생성하기
- Amazon S3에 새 이미지가 업로드되는 이벤트가 생기면 S3 이벤트 알림을 통해 람다 함수가 작동
- 이때 람다 함수에는 섬네일을 생성하는 코드가 있음
- 해당 섬네일은 다른 S3 버킷이나 같은 S3 버킷으로 푸시 및 업로드됨 (원래 이미지보다 작은 크기로)
- 또한 람다 함수는 몇몇 데이터를 DynamoDB에 삽입할 수 있음
- 이미지 이름, 크기, 생성 날짜 등 이미지의 메타 데이터가 삽임됨
- Lambda 덕분에 기능이 자동화되고 또 S3에 새 이미지와 앱이 생성되는 이벤트에 대한 반응형 아키텍처를 얻음
### 서버리스 CRON 작업
- CRON이란 EC2 인스턴스에서 작업을 생성하는 방법
- 5분마다, 월요일 10시마다 등등을 지정함
- 그런데 CRON은 가상 서버에 실행해야 함 EC2 인스턴스 등에..
- 인스턴스가 실행되지 않거나 CRON이 아무 일도 안 하면 인스턴스 시간이 낭비됨
- 따라서 CloudWatch 이벤트 규칙 또는 EventBridge 규칙을 만들고 1시간마다 작동되게 설정해서 1시간마다 람다 함수와 통합되면 태스크를 수행할 수 있게 됨
  - 서버리스 CRON을 만드는 것
- 이때 CloudWatch 이벤트는 서버리스이고 람다 함수 역시 서버리스
## Lambda 가격 책정 예시
- 호출당 청구
  - 처음 1백만 건의 요청은 무료이고 이후 1백만 건 요청마다 20센트가 과금
- 기간당 청구 (in increment of 1ms)
  - 한 달간 첫 40만 GB-초 동안의 컴퓨팅 시간은 무료로 사용
  - 이후에는 60만 GB-초당 1달러가 과금
## Lambda 한도
- 한도는 리전당 존재
### 실행 한도
- 실행 시 메모리 할당량은 128MB - 10GB (메모리는 1MB씩 증가
- 최대 실행 시간은 900초 (15분)
- 환경변수는 4KB까지 가질 수 있음
- 상당히 제한적인 공간이나 Lambda 함수를 생성하는 동안 큰 파일을 가져올 때 사용할 수 있는 임시 공간이 있음
  - /tmp 폴더에 용량이 있으며 크기는 최대 10GB
- Lambda 함수는 최대 1,000개까지 동시 실행이 가능하며 요청 시 증가할 수 있지만 동시성은 미리 예약해 두는 것이 좋음
### 배포 한도
- 압축 시 최대 크기는 50MB
- 압축하지 않았을 때는 250MB
- 이 용량을 넘는 파일의 경우 /tmp 공간을 사용해야 함
- 시작할 떄 크기가 큰 파일이 있으면 /tmp 디렉터리를 사용
- 배포 시에도 환경변수의 한도는 4KB
## Customization At The Edge 엣지에서의 사용자 지정
- 보통은 함수와 애플리케이션을 특정 리전에서 배포하지만 CloudFront를 사용할 때는 엣지 로케이션을 통해 콘텐츠를 배포함
- 모던 애플리케이션에서는 애플리케이션에 도달하기 전에 엣지에서 로직을 실행하도록 요구하기도 함
- 이를 엣지 함수라고 하며 CloudFont 배포에 연결하는 코드임
- 이 함수는 사용자 근처에서 실행하여 지연 시간을 최소화하는 것이 목적
- CloudFront 두 종류
  - CloudFront 함수
  - Lambda@Edge
- 엣지 함수를 사용하면 서버를 관리할 필요가 없음 - 전역으로 배포되기 때문
- 예 : CloudFront의 CDN 콘텐츠를 사용자 지정하는 경우
- 사용한 만큼만 비용을 지불하며 완전 서버리스
### CloudFront Functions & Lambda@Edge Use Cases
- 웹 사이트 보안과 프라이버시
- 엣지에서의 동적 웹 애플리케이션
- 검색 엔진 최적화(SEO)에도 활용 가능
- 오리진 및 데이터 센터 간 지능형 경로에도 활용
- 엣지에서의 봇 완화
- 엣지에서의 실시간 이미지 변환
- A/B 테스트 사용자 인증 및 권한 부여
- 사용자 우선순위 지정 사용자 추적 및 분석
### CloudFront Functions
1. Viewer Request (뷰어 요청)
2. Origin Request (CloudFront가 오리진 요청을 오리진 서버에 전송)
3. Origin Response (오리진 서버가 CloudFront에 오리진 응답을 보냄)
4. Viewer Response (CloudFront가 클라이언트에게 뷰어 응답을 전송)
- 확장성이 높고 지연 시간에 민감한 CDN 사용자 지정에 사용됨
- 시작 시간은 1밀리초 미만이며 초당 백만 개의 요청을 처리
- 뷰어 요청과 응답을 수정할 때만 사용
- CloudFront가 뷰어로부터 요청을 받은 다음에 뷰어 요청을 수정할 수 있음
- CloudFront가 뷰어에게 응답을 보내기 전에 뷰어 응답을 수정할 수 있음
- CloudFront의 네이티브 기능으로 모든 코드가 CloudFront에서 직접 관리됨
- CloudFront 함수는 고성능, 고확장성이 필요할 때 뷰어 요청과 뷰어 응답에만 사용됨
### Lambda@Edge Functions
- Node.js나 Python으로 작성하며 초당 수천 개의 요청을 처리할 수 있음
- 모든 CloudFront 요청 및 응답을 변경할 수 있음
- 뷰어 요청은 CloudFront Functions에서 본 대로
- 오리진 요청은 CloudFront가 오리진에 요청을 전송하기 전에 수정할 수 있음
- 오리진 응답은 CloudFront가 오리진에서 응답을 받은 후에 수정됨
## Lambda by default
- Lambda 함수를 시작하면 VPC 외부에서 시작됨 (VPC는 AWS가 제공하는 서비스)
  - VPC 내에서 리소스에 액세스할 권한이 없음
- RDS 데이터베이스, ElastiCache 캐시 내부 로드 밸런서를 시작하면 Lambda 함수가 해당 서비스에 액세스할 수 없음
- 해결하기 위해선 'VPC에서 Lambda 함수를 시작'
- VPC ID Lambda 함수를 시작하려는 서브넷을 지정하고 Lambda 함수에 보안 그룹을 추가
- 그러면 Lambda가 서브넷에 엘라스틱 네트워크 인터페이스를 생성해 VPC에서 실행되는 (Amazon RDS등..)에 액세스 할 수 있게 됨
## Lambda with RDS Proxy
- Lambda 함수로 직접 해당 DB에 액세스할 수 있음
- 그러나 이 방법으로 RDS 데이터베이스에 직접 액세스하면 Lambda 함수의 수가 너무 많이 생성되고 사라지길 반복 -> 개방된 연결이 많아짐 -> RDS 데이터베이스의 로드가 상승 -> 시간 초과 문제
- 해결하기 위해선 'RDS Proxy 시작'
- 이 프록시가 연결을 한곳으로 모으고 RDS 데이터베이스 인스턴스 연결의 수를 줄임
- Lambda 함수가 RDS 프록시에 연결되고 RDS 프록시가 RDS DB 인스턴스로 연결되므로 아키텍처상의 문제가 해결
### RDS Proxy 장점
- 데이터베이스 연결의 풀링과 공유를 통해 확장성을 향상
- 장애 발생 시 장애 조치 시간을 66%까지 줄여 가용성을 향상시키고 연결을 보존
- RDS 프록시 수준에서 IAM 인증을 강제하여 보안을 높일 수 있음. 자격 증명은 Secrets Manager에 저장됨

# Amazon DynamoDB
- 완전 관리형 데이터베이스
- 데이터가 다중 AZ 간에 복제되므로 가용성이 높음
- NoSQL
- 관계형 데이터베이스는 아니지만 트랜잭션 지원 기능이 있음
- DynamoDB를 이용하면 방대한 워크로드로 확장 가능
  - 데이터베이스가 내부에서 분산되기 때문
- 초당 수백만 개의 요청을 처리하고 수조 개의 행, 수백 TB의 스토리지를 갖게 됨
- 성능은 한 자릿수 밀리초를 자랑하고 일관성 또한 높음
- 보안과 관련된 모든 기능은 IAM과 통합되어 있음
- 보안, 권한 부여, 관리 기능 포함
- 비용이 적게 들고 오토 스케일링 기능이 탑재돼 있음
- 유지 관리나 패치 없이도 항상 사용할 수 있음
- 데이터베이스를 프로비저닝할 필요가 없음
- 항상 사용할 수 있으므로 테이블을 생성해 해당 테이블의 용량만 설정하면 됨
- 액세스가 빈번한 데이터는 Standard 클래스, 빈번하지 않으면 IA 테이블 클래스에 저장
- 데이터베이스가 이미 존재하는 서비스
- DynamoDB에 테이블을 생성하면 각 테이블에 기본 키가 부여됨 기본 키는 생성시 결정
- 행을 무한히 추가할 수 있음
- 각 항목은 속성을 가지고 속성은 열에 표시됨
- 속성은 나중에 추가할 수도 있고 null이 될 수도 있음
- 사전 요구 사항 없이 쉽게 속성 추가 가능
- 최대 크기는 400KB 이므로 큰 객체를 저장하기에는 적합하지 않음
- 스키마를 빠르게 전개해야 할 때 선택
## DynamoDB가 지원하는 데이터 유형
- Scalar Types
  - String, Number, Binary, Boolean, Null
- Document Types
  - List, Map
## DynamoDB - Read/Write Capacity Modes
- DynamoDB를 사용하려면 읽기/쓰기 용량 모드도 설정해야 함
### Provisioned Mode (default)
- 미리 용량을 프로비저닝함
- 초당 읽기/쓰기 요청 수를 예측해서 미리 지정하면 그것이 테이블의 용량이 됨
- 미리 용량을 계획하고 프로비저닝된 RCU의 WCU만큼의 비용을 지불하는 방식
- RCU는 읽기 용량 단위, WCU는 쓰기 용량 단위
- 미리 용량을 계획한 경우에도 오토 스케일링 기능이 있으므로 테이블의 로드에 따라 자동으로 RCU와 WCU를 늘리거나 줄일 수 있음
- 프로비저닝된 모드는 로드를 예측할 수 있고 서서히 전개되며 비용 절감을 원할 때 적합
### On-Demand Mode
- 읽기/쓰기 용량이 워크로드에 따라 자동으로 확장
- 미리 용량 계획을 하지 않으므로 RCU나 WCU 개념 자체가 없음
- 온디맨드 모드에서는 정확히 사용한 만큼의 비용을 지불함
- 모든 읽기와 쓰기에 비용을 지불
- 워크로드를 예측할 수 없거나 급격히 증가하는 경우에 유용
- 수천 개의 트랜잭션을 수백만 개의 트랜잭션으로 1분 내로 확장해야 하는 애플리케이션에는 빠르게 확장되지 않는 프로비저닝된 모드는 적합하지 않음 -> 온디맨드 모드 적합
- 트랜잭션이 없거나 하루에 많아야 4~5회밖에 되지 않는 워크로드라면 트랜잭션 횟수만큼만 비용을 지불하는 온디맨드 모드가 적합
## DynamoDB Accelerator (DAX)
- DynamoDB를 위한 고가용성의 완전 관리형 무결절 인메모리 캐시로 DynamoDB 테이블에 읽기 작업이 많을 때 DAX 클러스터를 생성하고 데이터를 캐싱하여 읽기 혼잡을 해결
- DAX는 캐시 데이터에 마이크로초 수준의 지연 시간을 제공함
- DAX 클러스터는 기존 DynamoDB API와 호환되므로 애플리케이션 로직을 변경할 필요가 없음
- DynamoDB 테이블과 애플리케이션이 있을 때 몇몇 캐시 노드가 연결된 DAX 클러스터를 생성하면 백그라운드에서 DAX 클러스터가 Amazon DynamoDB 테이블에 연결됨
- 캐시의 기본 TTL은 5분으로 설정되어있음 (변경가능)
## DynamoDB Accelerator (DAX) vs. ElastiCache
- DAX는 DynamoDB 앞에 있고 개별 객체 캐시와 쿼리와 스캔 캐시를 처리하는 데 유용
- Amazon ElastiCache 는 집계 결과를 저장할 때 좋음
- Amazon DynamoDB는 대용량의 연산을 저장할 때 좋음
- Amazon DynamoDB에 캐싱 솔루션을 추가할 때는 보통 DAX를 사용함
## DynamoDB - Stream Processing
- 테이블의 모든 수정 사항 즉 생성, 업데이트, 삭제를 포함한 스트림을 생성할 수 있음
- Use cases:
  - 사용자 테이블에 새로운 사용자가 등록됐을 때 환영 이메일을 보내는 등 DynamoDB 테이블의 변경 사항에 실시간으로 반응하는 데 활용
  - 실시간으로 사용 분석
  - 파생 테이블을 삽입
  - 리전 간 복제를 실행
  - DynamoDB 테이블 변경 사항에 대해 Lambda 함수를 실행할 수도 있음
- DynamoDB의 두 가지 스트림 처리
  - DynamoDB Streams
    - 24 hours retention
    - Limited # of consumers
    - Process using AWS Lambda Triggers, or DynamoDB Stream Kinesis adapter
  - Kinesis Data Streams (newer)
    - 1 year retention
    - High # of consumers
    - Process using AWS Lambda, Kinesis Data Analytics, Kineis Data Firehose, AWS Glue Streaming ETL...
## DynamoDB Global Tables
- 글로벌 테이블은 여러 리전 간에 복제가 가능한 테이블
- 테이블을 US-EAST-1과 AP-SOUTHEAST-2에 둘 수 있음
- 두 테이블 간에는 양방향 복제가 가능 (US-EAST-1이나 AP-SOUTHEAST-2 테이블 둘 중 하나에 쓰기를 하면 됨)
- DynamoDB 글로벌 테이블은 복수의 리전에서 짧은 지연 시간으로 액세스할 수 있게 해줌
- 다중 활성 복제가 가능하므로 애플리케이션이 모든 리전에서 테이블을 읽고 쓸 수 있음
- 글로벌 테이블을 활성화하려면 DynamoDB 스트림을 활성화해야 리전 간 테이블을 복제할 수 있는 기본 인프라가 구축됨
## DynamoDB -Time To Live(TTL)
- 만료 타임스탬프가 지나면 자동으로 항목을 삭제하는 기능
- SessionData 라는 테이블에서 ExpTime(TTL)이라는 만료 기간 속성안에 타임스탬프가 들어감
- TTL을 정의한 다음 에포크 타임스탬프에서의 현재 시간이 ExpTime 열을 넘어설 경우 해당 항목을 만료 처리하고 삭제 처리를 진행
- Use Cases:
  - 최근 항목만 저장하도록
  - 2년 후 데이터를 삭제해야 한다는 규정을 따라야 할 때
  - 웹 세션 핸들링
## DynamoDB - Backups for disaster recovery
- PITR (point-in-time-recovery) 지정 시간 복구
  - 활성화를 선택할 수 있고 35일 동안 지속됨
  - 활성화하면 백업 기간 내에는 언제든 지정 시간 복구를 실행할 수 있음
  - 복구를 진행할 경우 새로운 테이블을 생성함
- On-demand backups
  - 직접 삭제할 때까지 보존됨
  - DynamoDB의 성능이나 지연 시간에 영향을 주지 않음
  - 백업을 좀 더 제대로 관리할 수 있는 방법 중 하나로 AWS Backup 서비스가 있음
  - 재해 복구 목적으로 리전 간 백업을 복할 수 있음
  - 복구를 진행할 경우 새로운 테이블을 생성함
## DynamoDB - Integration with Amazon S3
- Export to S3 (must enable PITR)
  - DynamoDB 테이블을 S3로 내보내고 쿼리를 수행하려면 Amazon Athena 엔진을 사용함
  - 지속적인 백업을 활성화한 상태이므로 최근 35일 이내 어떤 시점으로든 테이블을 내보낼 수 있음
  - 테이블을 내보내도 테이블의 읽기 용량이나 성능에 영향을 주지 않음
  - DynamoDB를 Amazon S3로 먼저 내보내기 하여 데이터 분석을 수행할 수 있음
  - 감사 목적으로 스냅샷을 확보할 수 있음
  - 데이터를 DynamoDB로 다시 가져오기 전에 데이터 ETL 등 대규모 변경을 실행할 수 있음
  - 내보낼 때는 DynamoDB JSON이나 ION 형식을 이용함
- Import to S3
  - S3에서 CSV, JSON 그리고 ION 형식으로 내보낸 다음 새로운 DynamoDB 테이블을 생성
  - 쓰기 용량을 소비하지 않고 새로운 테이블이 생성
  - 가져올 때 발생한 오류는 모두 CloudWatch Logs에 기록됨
## Example: Building a Serverless API
- 클라이언트가 직접 Lambda 함수를 지연 호출(invoke)할 수 있게 하려면 클라이언트에게 IAM 권한이 있어야 함
- 클라이언트와 Lambda 함수 사이에 애플리케이션 로드 밸런서를 배치하면 Lambda 함수가 HTTP 엔드포인트에 노출됨
- API Gateway를 사용하는 방법도 있음
- AWS의 서버리스 서비스로 REST API를 생성할 수 있으므로 클라이언트가 퍼블릭 액세스할 수 있음
- API Gateway에 클라이언트가 직접 소통하며 다양한 작업을 할 수 있고 Lambda 함수에 요청을 프록시함
- API Gateway를 사용하는 이유는 HTTP 엔드포인트뿐만 아니라 다양한 기능 예를 들어 인증부터 사용량 계획, 개발 단계 등의 기능을 제공하기 때문

# AWS API Gateway
- AWS lambda + Api Gateway: No infrastructure to manage
- Support for the WebSocket Protocol
- Handle API versioning (v1, v2..)
- Handle different environments (dev, test, prod...)
- Handle security (Authentication and Authorization)
- Create API keys, handle request throttling
- Swagger / Open API import to quickly define APIs
- Transform and validate requests and responses
- Generate SDK and API specifications
- Cache API responses
## API Gateway - Integrations High Level
- Lambda Function
  - Invoke Lambda function
  - Easy way to expose REST API backed by AWS Lambda
- HTTP
  - 백엔드의 HTTP의 엔드포인트를 노출시킬 수 있음
  - Example: internal HTTP API on premise, Application Load Balancer...
    - Why? Add rate limiting, caching, use authentications, API keys, etc...
- AWS Service
  - Expose any AWS API through the API Gateway?
  - Example: start an AWS Step Function workflow, post a message to SQS
    - Why? Add authentication, deploy publicly, rate control..
## API Gateway - AWS Service Integration Kinesis Data Streams example
- Kinesis Data Streams
  - Kinesis Data Streams에 사용자가 데이터를 전송할 수는 있지만 AWS 자격 증명은 가질 수 없도록 보안 설정을 할 수 있음
  - Kinesis Data Streams과 클라이언트 사이에 API Gateway를 두는 것
  - 클라이언트가 API Gateway로 HTTP 요청을 보내면 Kinesis Data Streams에 전송하는 메시지로 구성해 전송함
  - 따로 서버를 관리할 필요가 없음
  - Kinesis Data Streams에서 Kinesis Data Firehose로 레코드를 전송하고 최종적으로 JSON 형식으로 Amazon S3 버킷에 저장
- API Gateway를 통해 AWS 서비시를 외부에 노출시키는 것
## API Gateway - Endpoint Types
- Edge-Optimized (default): For global clients
  - 전 세계 누구나 API Gateway에 액세사 가능
  - 모든 요청이 CloudFront 엣지 로케이션을 통해 라우팅되므로 지연 시간이 개선됨
  - The API Gateway still lives in only one region
- Regional:
  - For clients within the same region
  - CloudFront 엣지 로케이션을 원하지 않을 때 사용
  - 자체 CloudFront 배포를 생성할 수 있음
  - 엣지 최적화 배포와 동일한 결과를 내며 캐싱 전략과 CloudFront 설정에 더 많은 권한을 가질 수 있음
- Private:
  - 프라이빗 API Gateway는 VPC 내에서만 액세스할 수 있음 (ENI)
## API Gateway - Security
- User Authentication through
  - IAM Roles (useful for internal applications)
  - Cognito (identity for external users - example mobile users)
  - Custom Authorizer (your own logic)
- Custom Domain Name HTTPS security through integration with AWS Certificate Manager (ACM)
  - 엣지 최적화 엔드포인트를 사용할 경우 인증서는 us-east-1에 있어야 하고
  - 리전 포인트를 사용한다면 인증서는 API Gateway 단계와 동일한 리전에 있어야 함
  - Route 53에 CNAME이나 A-별칭 레코드를 설정해 도메인 및 API Gateway를 가리키도록 해야 함 

# AWS Step Functions
- Build serverless visual workflow to orchestrate your Lambda functions
- Features: sequence, parallel, conditions, timeouts, error handling...
- Can integrate with EC2, ECS, On-premises servers, API Gateway, SQS queues, etc...
- Possibility of implementing human approval feature
- Use cases: order fulfillment, data processing, web applications, any workflow