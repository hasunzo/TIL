# Amazon CloudWatch Metrics
- CloudWatch는 AWS의 모든 서비스에 대한 지표를 제공
- Metric은 모니터링할 변수
- EC2 인스턴스 지표로는 CPUUtilization, NetworkIn 등
- Amazon S3의 지표로는 버킷 크기 등
- Metric은 namespaces에 속함
- Dimension(측정 기준) is an attribute of a metric (instance id, environment, etc...)
- Metric(지표)당 최대 측정 기준은 10개
- 지표는 시간을 기반으로 하므로 타임스탬프가 꼭 있어야 하고 지표가 많아지면 CloudWatch 대시보드에 추가해 모든 지표를 한 번에 볼 수 있음
- CloudWatch 사용자 지정 지표를 만들 수도 있음
- AWS의 서비스에서 제공되는 지표만 보는 대신 자체 지표를 만들 수 있음
  - EC2 인스턴스로부터 메모리 사용량을 추출
## CloudWatch Metric Streams
- CloudWatch 지표는 Kinesis Data Firehose로 거의 실시간으로 스트리밍
- 별도의 설정이 필요함
- Kinesis Data Firehose에서 Amazon S3 버킷으로 지표를 보내 Amazon Athena를 사용해 지표를 분석
- Amazon Redshift를 사용해 지표로 데이터 웨어하우스를 만들거나 Amazon OpenSearch를 사용해 대시보드를 생성하고 지표 분석을 할 수 있음

# CloudWatch Logs
- Log groups: 로그들을 로그 그룹으로 그룹화.
- Log stream: instances within application / log files / containers
- Can define log expiration policies (never expire, 30 days, etc...)
- CloudWatch Logs can send logs to:
  - Amazon S3 (exports)
  - Kinesis Data Streams
  - Kinesis Data Firehose
  - AWS Lambda
  - ElasticSearch
## CloudWatch Logs - Sources
- 어떤 유형의 로그를 여기에 저장할 수 있을까?
- SDK, CloudWatch Logs 에이전트
- 통합 CloudWatch 에이전트를 통해 로그를 보낼 수 있음
- Elastic Beanstalk: 애플리케이션의 로그를 CloudWatch에 전송
- ECS: 컨테이너의 로그를 CloudWatch에 전송
- AWS Lambda: 함수 자체에서 로그를 보냄
- VPC Flow Logs: VPC specific logs
- API Gateway: 받은 요청 모두
- CloudTrail based on filter
- Route53: Log DNS queries
## CloudWatch Logs Metric Filter & Insights
- CloudWatch Logs에서 필터 표현식을 쓸 수 있음
- 예를 들어 로그 내 특정 IP를 찾을 수 있음
- 특정 IP가 찍힌 로그를 찾거나 또는 'ERROR'라는 문구를 가진 모든 로그를 찾을 수 있음
- 이 지표 필터를 통해 출현 빈도를 계산해 지표를 만들 수 있음
- 만들어진 지표는 CloudWatch 경보로 연동 가능
- CloudWatch Logs Insights
  - 로그를 쿼리하고 이 쿼리를 대시보드에 바로 추가할 수 있음
  - 자주 쓰이는 쿼리들도 AWS에서 추가해 놓음
## CloudWatch Logs - S3 Export
- CloudWatch에서 S3로 보낼 때 내보내기가 가능해질 때까지 최대 12시간은 걸릴 수 있음
- API 호출은 CreateExportTask
- 시간이 따로 설정돼 있어 실시간이 아님
- CloudWatch Logs에서 로그를 스트림하고 싶다면 구독 필터를 사용해야 함
## CloudWatch Logs Subscriptions
- 구독 필터란 CloudWatch Logs 상단에 적용하여 이를 목적지로 보내는 필터를 말함
- 목적지의 예를 들면 Lambda 함수
  - 직접 정의하든 AWS 제공을 쓰든 Amazon ES에 데이터를 보낼 때 씀
- Kinesis Data Firehose는 Amazon S3에 근 실시간 전송을 할 때 씀
  - CloudWatch에서 S3로 내보내기한 것보다 훨씬 빠름
- Kinesis Data Streams로 Kinesis Data Firehose에 데이터를 보낼 수도 있음
- 그 외에도 KDF, KDA, EC2, Lambda 등이 있음
## CloudWatch Logs Aggregation Multi-Account & Multi Region
- 여러 계정과 리전간 로그를 집계할 수 있음
- 구독 필터를 가진 리전1에 계정A가 있다면 공통 계정을 통해 Kinesis Data Streams로 보내짐
- 계정B와 리전 2, 3도 마찬가지
- 모든 로그를 모아 Kinesis Data Stremas 그리고 Data Firehose 이후에는 Amazon S3로 보낼 수 있음

# CloudWatch Logs for EC2
- EC2 인스턴스에서 CloudWatch로는 기본적으로 어떤 로그도 옮겨지지 않음
- EC2 인스턴스에 에이전트라는 프로그램을 실행시켜 원하는 로그파일을 푸시해야 함
- CloudWatch Logs 에이전트가 EC2 인스턴스에서 작동해 CloudWatch Logs로 로그를 보내는 것
- 로그를 보낼 수 있게 해주는 IAM 역할 필요
## CloudWatch Logs Agent & Unified Agent
### CloudWatch Logs Agent
- Old version of the agent
- can only send to CloudWatch Logs
### CloudWatch Unified Agent 통합 에이전트
- Collect additional system-level metrics such as RAM, processes, etc...
- Collect logs to send to CLoudWatch Logs
- 지표와 로그를 둘 다 사용
## CloudWatch Unified Agent - Metrics
- CPU
- DIsk metrics
- RAM
- Netstat
- Processes
- Swap Space

# CLoudWatch Alarms
- Alarms are used to trigger notifications for any metrics
- 경보 상태
  - OK : 트리거되지 않았음
  - INSUFFICIENT_DATA : 상태를 결정할 데이터가 부족함
  - ALARM : 임계값이 위반되어 알림이 보내지는 상태

# Amazon EventBridge (formerly CloudWatch Events)
- 클라우드에서 CRON 작업을 예약할 수 있음 (scheduled scripts)
- 한시간마다 Lambda 함수를 트리거해서 스트립트를 실행한다거나.. 등
- 이벤트 패턴에 반응. 특정 작업을 수행하는 서비스에 반응하는 이벤트 규칙
  - 콘솔의 IAM 루트 사용자 로그인 이벤트에 반응
  - 이벤트가 발생하면 SNS 주제로 메시지를 보내고 이메일 알림을 받을 수 있음
- Trigger Lambda functions, send SQS/SNS messages
- Amazon EventBridge는 기본 이벤트 버스
  - 이벤트를 기본 이벤트 버스로 전송하는 AWS의 서비스
- 파트너 이벤트 버스
  - 파트너와 통합된 AWS 서비스. 대부분의 파트너는 소프트웨어 서비스 제공자

# CloudWatch Container Insights
- 컨테이너로부터 지표와 로그를 수집, 집계, 요약하는 서비스
- Amazon ECS나 Amazon EKS, EC2의 Kubernetes 플랫폼에 직접 실행하는 컨테이너에서 사용
- ECS와 EKS의 Fargate에 배포된 컨테이너에서도 사용
- 컨테이너로부터 지표와 로그를 손쉽게 추출해서 CloudWatch에 세분화된 대스보드를 만들 수 있음
- ECS, EKS, EC2의 Kubernetes, Fargate로부터 오는 로그와 지표를 위한 것
- Kubernetes를 사용한다면 실행할 에이전트가 필요함

# CloudWatch Lambda Insights
- AWS Lambda에서 실행되는 서버리스 애플리케이션을 위한 모니터링과 트러블 슈팅 솔루션
- CPU 시간, 메모리 디스크, 네트워크, 콜드 스타트나 Lambda 작업자 종료와 같은 정보를 포함한 시스템 수준의 지표를 수집, 집계, 요약
- Lambda 함수를 위해 Lambda 계층으로 제공
- Lambda Insights라는 대시보드를 생성해 Lambda 함수의 성능을 모니터링한다는 것만 알면 됨

# CloudWatch Contributor Insights
- Contributor(기고자) 데이터를 표시하는 시계열 데이터를 생성하고 로그를 분석하는 서비스
- VPC 로그, DNS 로그 등 AWS가 생성한 모든 로그에서 작동
- 네트워크 로그나 VPC 로그를 확인해서 사용량이 가장 많은 네트워크 사용자를 찾을 수 있음
- DNS 로그에서는 오류를 가장 많이 생성하는 URL을 찾을 수 있음
- 사용량이 많은 네트워크 사용자를 식별하는 방법
  - VPC Flow Logs -> CloudWatch Logs -> CloudWatch Contributor Insights
- CloudWatch Logs를 통해 상위 N개의 기고자(Contributor)를 찾음

# CloudWatch Application Insights
- 모니터링하는 애플리케이션의 잠재적인 문제와 진행 중인 문제를 분리할 수 있도록 자동화된 대시보드를 제공
- 애플리케이션에 문제가 있는 경우 CloudWatch Application Insights는 자동으로 대시보드를 생성하여 서비스의 잠재적인 문제를 보여 줌

# AWS CloudTrail
- AWS 계정의 거버넌스, 감사 및 규정 준수를 도움
- CloudTrail은 기본적으로 활성화돼 있고 콘솔, SDK, CLI뿐만 아니라 기타 AWS 서비스에서 발생한 AWS 계정 내의 모든 이벤트 및 API 호출 기록을 받아 볼 수 있음
- CloudTrail의 로그를 CloudWatch Logs나 Amazon S3로 옮길 수 있음
- 전체 또는 단일 리전에 적용되는 트레일을 생성해 모든 리전에 걸친 이벤트 기록을 한곳으로 모을 수 있음 (S3 버킷)
- 누군가가 AWS에서 무언가를 삭제했을 떄 대처할 수 있음
- 예를 들어 EC2 인스턴스가 종료됐을 경우 누가 했는지를 알고 싶으면 CloudTrail을 들여다보면 됨
- API 호출이 남아 있으므로 누가 언제 문제를 일으켰는지 알아낼 수 있음

# CloudWatch vs CloudTrail vs Config
### CloudWatch
- Performance monitoring (metrics, CPU, network, etc...) & dashboards
- Events & Alerting
- Log Aggregation & Analysis
### CloudTrail
- Record API calls made within your Account by everyone
- Can define trails for specific resources
- Global
### Config
- Record configuration changes
- Evaluate resources against compliance rules
- Get timeline of changes and compliance
## For an Elastic Load Balancer
### CloudWatch
- Monitoring Incoming connections metric
- Visualize error codes as a % over time
- Make a dashboard to get an idea of your load balancer performance
### Config
- Track security group rules for the Load Balancer
- Track configuration changes for the Load Balancer
- Ensure an SSL certificate is always assigned to the Load Balancer (compliance)
### CLoudTrail
- Track who made any changes to the Load Balancer with API calls