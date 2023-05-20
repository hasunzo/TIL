# Amazon Athena
- Amazon S3 버킷에 저장된 데이터 분석에 사용하는 서버리스 쿼리 서비스
- 데이터를 분석하려면 표준 SQL 언어로 파일을 쿼리해야 함
- Athena는 SQL 언어를 사용하는 Presto 엔진에 빌드되고,
- 사용자가 S3 버킷에 데이터를 로드하거나 내가 S3 버킷에 데이터를 로드하면 Athena 서비스를 사용해 이동하지 않고 Amazon S3에서 데이터를 쿼리하고 분석할 수 있음
- 즉 Athene는 서버리스로 S3 버킷의 데이터를 바로 분석함
- CSV, JSON, ORC, Avro Parquet 등 다양한 형식을 지원
- Pricing: $5.00 per TB of data scanned
- 전체 서비스가 서버리스여서 데이터베이스를 프로비저닝 할 필요가 없음
- Athena는 Amazon QuickSight라는 도구와 함께 사용하는 일이 많음 - 보고서와 대시보드 생성
- QuickSight는 S3 버킷에 연결된 Athena 다음에 배치됨
- Amazon Athena의 사용 사례
  - 임시 쿼리 수행
  - 비즈니스 인텔리전스 분석 및 보고
  - AWS 서비스에서 발생하는 모든 로그를 쿼리하고 분석할 수 있음
  - VPC 흐름 로그, 로드 밸런서 로그, CloudTrail 추적 등
- Exam Tip: Amazon S3 데이터 분석이 나오면 Athena를 떠올리자
## Amazon Athena - Performance Improvement
- Use columnar data for cost-savings (less scan)
  - 데이터를 적게 스캔할 유형의 데이터를 사용
  - 열 기반 데이터 유형 사용하면 필요한 열만 스캔하므로 비용을 절감할 수 있음
  - Apache Parquet or ORC is recommended
  - Huge performance improvement
  - Use Glue to convert your data your Parquet or ORC
- Compress data for smaller retrievals (bzip2, gzip, lz4, snappy, zlip, zstd...)
- Partition datasets in S3 for easy querying on virtual columns
  - 특정 열을 항상 쿼리한다면 데이터 세트를 분할하기
  - S3 버킷에 있는 전체 경로를 슬래시로 분할하는 것
  - Example: S3://athena-examples/flight/parquet/year=1991/month=1/day=1
  - 데이터를 쿼리할 때 Amazon S3의 어떤 폴더(어떤 경로)로 데이터를 스캔할지 정확히 알 수 있음
- Use larger files (> 128 MB) to minimize overhead
  - 파일이 클수록 스캔과 검색이 쉬우므로 128MB 이상의 파일을 사용해야 함
## Amazon Athena - Federated Query (연합 쿼리)
- 관계형 데이터베이스나 비관계형 데이터베이스 객체, 사용자 지정 데이터 원본을 쿼리할 수 있음
- AWS나 온프레미스에서 어떻게 사용?
  - 데이터 원본 커넥터를 사용
  - 데이터 원본 커넥터는 Lambda 함수로 다른 서비스에서 연합 쿼리를 실행함
  - CloudWatch Logs, DynamoDB, RDS 등에서 실행하며 매우 강력
- Athena에서 바로 Amazon S3뿐만 아니라 모든 온프레미스 데이터베이스를 쿼리할 수 있으며 쿼리를 조인하거나 경쟁할 수도 있음
- 쿼리 결과는 사후 분석을 위해 Amazon S3 버킷에 저장할 수 있음

# Amazon Redshift
- Redshift는 PostgreSQL 기술 기반이지만 온라인 트랜잭션 처리(OLTP)에 사용되지는 않음
- 온라인 분석 처리를 의미하는 OLAP 유형의 데이터베이스이며 분석과 데이터 웨어하우징에 사용함
- 다른 웨어하우징보다 성능이 10배 이상 좋고 데이터가 PB 규모로 확장되므로 모든 데이터를 Redshift에 로드하면 Redshift에서 빠르게 분석할 수 있음
- Redshift는 성능 향상이 가능한데 이는 열 기반 데이터 스토리지를 사용하기 때문
  - 행 기반이 아니라 병렬 쿼리 엔진이 있는 것
  - Redshift 클러스터에서 프로비저닝한 인스턴스에 대한 비용만 지불하면 됨
- 쿼리를 수행할 때 SQL 문을 사용할 수 있음
- Amazon Quicksight같은 BI 도구나 Tableau 같은 도구도 Redshift와 통합할 수 있음
- vs Athena: faster queries / joins / aggregations thanks to indexes
## Redshift Cluster
- Leader node 리더 노드
  - 쿼리를 계획하고 결과를 집계
- Compute node 컴퓨팅 노드
  - 실제로 쿼리를 실행하고 결과를 리더 노드에 보냄
- Redshift 클러스터는 노드 크기를 미리 프로비저닝해야 하고 비용을 절감하려면 예약 인스턴스를 사용하면 됨
- 리더 노드에 쿼리를 SQL 형식으로 제출하면 백엔드에서 쿼리가 발생함
## Redshift - Snapshots & DR
- Redshift에는 다중 AZ 모드가 없고 클러스터가 한 개의 가용 영역에 있으므로 Redshift에 재해 복구 전략을 적용하려면 스냅샷을 사용해야 함
- 스냅샷은 클러스터의 지정 시간 백업으로 Amazon S3 내부에 저장되며 증분함
- 변경된 사항만 저장되므로 많은 공간을 절약할 수 있음
- 새로운 Redshift 클러스터에 스냅샷을 복원할 수 있으며 스냅샷에는 두 가지 모드가 있음
  - 수동
    - 직접 스냅샷을 삭제하기 전까지 스냅샷이 유지됨
  - 자동
    - 스냅샷이 8시간마다 또는 5GB마다 자동으로 생성되도록 일정을 예약
    - 자동화된 스냅샷의 보존 기간을 설정
  - 자동이든 수동이든 클러스터의 스냅샷을 다른 AWS 리전에 자동으로 복사하도록 Redshift를 구성하여 재해 복구 전략을 적용할 수 있음
  - 원본 Redshift 클러스터와 다른 리전이 있을 때 스냅샷을 생성하면 새로운 리전에 자동으로 복사되고
  - 복사된 스냅샷에서 새 Redshift 클러스터를 복원할 수 있음
## Loading data into Redshift: Large inserts are MUCH better
- Amazon Kinesis Data Firehose
  - Firehose가 다양한 소스로부터 데이터를 받아서 Redshift에 보내는 것
  - Amazon S3 버킷에 데이터 쓰기 -> 자동으로 Kinesis Data Firehose가 S3 복사 명령 실행 -> Redshift에 데이터 로드
- S3 using COPY command
  - S3 데이터 로드 -> Redshift에서 복사 명령 실행 -> IAM 역할을 사용해 S3 버킷에서 Amazon Redshift 클러스터로 데이터 복사
  - 데이터 복사는 Internet을 이용하는 방법과 VPC 라우팅 사용 방법 두 가지가 있다
- EC2 Instance JDBC driver
  - 애플리케이션에 Redshift 클러스터에 써야 하는 EC2 인스턴스가 있을 때 이 방법을 사용
  - 이떄 Amazon Redshift에 큰 배치로 데이터를 쓰는 것이 좋음
  - 이 유형의 데이터베이스에 한 번에 한 행씩 쓰는 건 비효율적
## Redshift Spectrum
- Amazon S3에 있는 데이터를 Redshift를 사용해 분석은 하지만 Redshift에 로드는 하지 않음
- 더 많은 처리 능력을 사용하기 위해서
- Redshift Spectrum을 사용하려면 쿼리를 시작할 수 있는 Redshift 클러스터가 있어야 함
- 일단 쿼리를 시작하면 S3에 있는 데이터에 쿼리를 실행한 수천 개의 Redshift Spectrum 노드에 쿼리가 제출됨
- 예시
  - Redshift 클러스터에 리더 노드와 여러 개의 컴퓨팅 노드가 있음
  - 분석할 데이터는 Amazon S3에 있으며 Redshift 클러스터에서 쿼리를 실행할 것
  - 쿼리하려는 테이블이 S3에 있으므로 쿼리는 From S3. 으로 시작해야 함
  - 그러면 Spectrum이 자동으로 시작되고 쿼리는 Amazon S3에서 데이터를 읽고 집계하는 수천 개의 Redshift Spectrum 노드에 제출된
  - 제출이 완료되면 결과가 Amazon Redshift 클러스터를 통해 쿼리를 시작한 곳으로 전송됨
- 이 기능을 사용하면 Amazon S3에서 Redshift로 데이터를 로드하지 않아도 되므로 클러스터에서 프로비저닝한 것보다 더 많은 처리 능력을 활용할 수 있음

# Amazon OpenSearch Service fS
- Amazon ElasticSearch의 후속 서비스
- 기본 키나 데이터베이스의 인덱스로만 데이터를 쿼리할 수 있는 DynamoDB와 달리,
- OpenSearch로는 부분적으로 일치하는 필드를 포함해 모든 필드를 검색할 수 있음
- OpenSearch는 애플리케이션에서 검색 기능을 제공할 때 많이 사용되고 일반적으로 다른 데이터베이스를 보완하는 데 사용됨
- OpenSearch는 검색에 사용되지만 OpenSearch를 사용하면 쿼리를 분석할 수도 있음
- OpenSearch를 생성하고 사용하려면 인스턴스의 클러스터를 생성해야 함. 서버리스가 아님
- 자체 쿼리 언어가 있어 SQL을 지원하지 않음
- Kinesis Data Firehose AWS IoT, CloudWatch Logs나 사용자 지정 애플리케이션의 데이터를 주입할 수 있음
- Cognito, IAM과 통합해 제공하는 보안을 통해 저장 데이터 암호와의 전송 중 암호화가 가능함

# Amazon EMR
- Elastic MapReduce
- AWS에서 빅 데이터 작업을 위한 하둡 클러스터 생성에 사용
- 방대의 양의 데이터를 분석하고 처리
- 하둡 클러스터가 있는 빅 데이터 관련된 내용이 나오면 Amazon EMR
- 하둡 클러스터는 프로비저닝해야 하며 수백 개의 EC2 인스턴스로 구성될 수 있음
- 왜 EMR을 사용하는가?
  - EMR은 빅 데이터 전문가가 사용하는 여러 도구와 함께 제공되는데
  - Apache Spark, HBase, Presto Apache Flink는 설정이 어려운데
  - Amazon EMR이 상기 서비스에 관한 프로비저닝과 구성을 대신 처리해 줌
  - 전체 클러스터를 자동으로 확장할 수 있고 스팟 인스턴스와 통합되므로 가격 할인 혜택을 받을 수도 있음
- Use cases:
  - 데이터 처리, 기계 학습, 웹 인덱싱 빅 데이터 작업
  - 모든 작업은 하둡, Spark, HBase, Presto, Flink와 같은 빅데이터 관련 기술을 사용
## Amazon EMR - Node types & purchasing
- Master Node: Manage the cluster, coordinate, manage health - long running
- Core Node: Run tasks and store data - long running
- Task Node (optional): Just to run tasks - usually Spot
- Purchasing options:
  - On-demand: reliable, predictable, won't be terminated
  - Reserved (min 1 year): cost savings (EMR will automatically use if available)
  - Spot Instances: cheaper, can be terminated, less reliable

# Amazon QuickSight
- 서버리스 머신 러닝 기반 비즈니스 인텔리전스 서비스
- 대화형 대시보드 생성
- 대시보드 생성하고 소유한 데이터 소스와 연결할 수 있음
- Use cases:
  - 비즈니스 분석, 시가고하 구현, 시각화된 정보를 통한 임시 분석 수행
  - 데이터를 활용한 비즈니스 인사이트 획득
- RDS, Aurora, Athena, Redshift, S3 등 데이터 소스에 연결 가능
- JSON, Excel, CSV, TSV 파일, 로그 형식의 ELF 및 CLF 등의 데이터 소스를 가져올 수 있음
- IAM 사용자는 관리용으로만 사용되고 QuickSight 내에서 사용자와 그룹을 정의하고 대시보드 생성하면 됨
- 액세스 권한이 있는 사용자는 기본 데이터를 볼 수도 있음

# AWS Glue
- 추출과 변환 로드 서비스를 관리하며 ETL 서비스라고도 함
- 완전 서버리스 서비스
- S3 버킷이나 Amazon RDS 데이터베이스에 있는 데이터를 데이터 웨어하우스인 Redshift에 로드할 경우
  - Glue를 사용해 추출한 다음 일부 데이터를 필터링하거나 열을 추가하는 등 원하는 대로 데이터를 변형할 수 있음
  - 그런 다음 최종 출력 데이터를 Redshift 데이터 웨어하우스에 로드함
## AWS Glue - Convert data into Parquet format
- Parquet 형식으로 변환하면 Amazon Athena가 파일을 훨씬 더 잘 분석함
- 전체 과정을 자동화할 수도 있음
- 파일이 S3 버킷에 삽입될 때마다 Lambda 함수로 이벤트 알림을 보내 Glue ETL 작업을 트리거하는 것
- Lambda 함수 대신 EventBridge를 사용해도 됨
## Glue - things to know at a high-level
- Glue Job Bookmarks: prevent re-processing old data
  - 이전 데이터의 재처리를 방지
- Glue Elastic Views:
  - Combine and replicate data across multiple data stores using SQL
  - RDS 데이터베이스와 Aurora 데이터베이스 Amazon S3에 걸치 뷰를 생성
- Glue DataBrew:
  - 사전 빌드된 변환을 사용해 데이터를 정리하고 정규화
- Glue Studio:
  - Glue에서 ETL 작업을 생성, 실행 및 모니터링하는 GUI
- Glue Streaming ETL
  - Apache Spark Structured Streaming 위에 빌드되며 ETL 작업을 배치 작업이 아니라 스트리밍 작업으로 실행할 수 있음
  - Kinesis Data Streaming Kafka 또는 AWS의 관리형 Kafka인 MSK에서 Glue 스트리밍 ETL을 사용해 데이터를 읽을 수 있음

# AWS Lake Formation
- Data lake = 모든 데이터를 한곳으로 모아 주는 중앙 집중식 저장소
- Lake Formation은 데이터 레이크 생성을 수월하게 해 주는 완전 관리형 서비스
- Data lake에서의 데이터 검색, 정제, 변환 주입을 도움
- 데이터 수집, 정제나 카탈로깅, 복제 같은 복잡한 수작업을 자동화
- 기계 학습(ML) 변환 기능으로 중복제거 수행
- Out-of-the-box source blueprints: Amazon S3, Amazon RDS, Relational & NoSQL DB
- 모든 데이터를 한곳에서 처리하는 것 외에도 애플리케이션에서 행, 열 수준의 세분화된 액세스 제어를 할 수 있음
- Glue 위에 빌드되는 계층이긴 하지만 Glue와 직접 상호 작용하진 않음
## Lake Formation을 사용하는 이유는 중앙화된 권한이다
- 보안을 관리할 곳이 많아지면 엉망이 됨 -> Lake Formation 사용하자
- 액세스 제어 기능과 열 및 행 수준 보안이 있음
- Lake Formation에 주입된 모든 데이터는 중앙 S3 버킷에 저장되지만 모든 액세스 제어와 행, 열 수준 보안은 Lake Formation 내에서 관리됨
- 따라서 Lake Formation에 연결하는 서비스는 읽기 권한이 있는 데이터만 볼 수 있게 됨

# Kinesis Data Analytics for SQL applications
- 중앙에 위치하여 Kinesis Data Streams와 Kinesis Data Firehose 데이터 소스에서 데이터를 읽음
- 둘 중 한군데서 데이터를 읽어 온 다음 SQL 문에 적용하여 실시간 분석을 처리
- Amazon S3 버킷의 데이터를 참조해 참조 데이터를 조인
- 여러 대상에 데이터 전송
- 데이터 전송 방법 두가지
  - Kinesis Data Streams
    - Kinesis Data Analytics의 실시간 쿼리로 스트림 생성
    - EC2에서 실행하는 애플리케이션이나 AWS Lambda로 스트리밍하는 데이터를 실시간으로 처리할 수 있음
  - Kinesis Data Firehose로 바로 전송
    - Kinesis Data Firehose로 바로 보내면 Amazon S3 Amazon Redshift이나 Amazon OpenSearch 또는 기타 Firehose 대상에 전송됨
  - 데이터 소스로는 Kinesis Data Streams와 Firehose가 있음
  - Amazon S3로 데이터를 강화할 수 있음
  - 완전 관리형 서비스이므로 서버를 프로비저닝하지 않음
  - 오토 스케일링이 가능하며 Kinesis Data Analytics에 전송된 데이터만큼 비용을 지불함
  - Kinesis Data Streams나 Kinesis Data Firehose에 데이터를 출력할 수 있음
  - Use cases:
    - Time-series analytics 시계열 분석
    - 실시간 대시보드
    - 실시간 지표

# Kinesis Data Analytics for Apache Flink
- Kinesis Data Analytics에서 Apache Flink를 사용할 수 있음
- Apache Flink를 사용하면 Java, Scala, SQL로 애플리케이션을 작성하고 스트리밍 데이터를 처리, 분석할 수 있음
- Flink는 코드로 작성해야 하는 특별한 애플리케이션
- Flink 애플리케이션을 Kinesis Data Analytics의 Flink 전용 클러스터에서 실행할 수 있음 (백그라운드에서)
- Apache Flink을 사용해 두 개의 메인 데이터 소스인 Kinesis Data Streams나 Amazon MSK의 데이터를 읽을 수 있음
- AWS의 관리형 클러스터에서 Apache Flink 애플리케이션을 실행할 수 있음
- 고급 쿼리 능력이 필요할 때 사용
- Kinesis Data Streams나 AWS의 관리형 Kafka인 Amazon MSK 같은 서비스로부터 스트리밍 데이터를 읽는 능력이 필요할 때 사용
- 이 서비스를 사용하면 컴퓨팅 리소스를 자동 프로비저닝할 수 있음
- 병렬 연산과 오토 스케일링을 할 수 있음
- 체크포인트와 스냅샷으로 구현되는 애플리케이션 백업이 가능
- Apache Flink 프로그래밍 기능을 사용할 수도 있음
- Apache Flink는 Kinesis Data Streams나 Amazon MSK의 데이터는 읽지만 Kinesis Data Firehose의 데이터는 읽지 못함
- Kinesis Data Firehose에서 데이터를 읽고 실시간 분석하려면 SQL 애플리케이션용 Kinesis Data Analytics를 사용해야 함

# Amazon Managed Streaming for Apache Kafka (Amazon MSK)
- Amazon Kinesis의 대안
- 데이터를 스트리밍함
- MSK는 AWS의 완전 관리형 Kafka 클러스터 서비스
- 그떄그때 클러스터를 생성, 업데이트, 삭제함
- MSK는 클러스터 내 브로커 노드와 Zookeeper 브로커 노드를 생성 및 관리
- 고가용성을 위해 VPC의 클러스터를 최대 세 개의 다중 AZ 전역에 배포함
- 일반 Kafka 장애를 자동 복구하는 기능
- EBS 볼륨에 데이터를 저장할 수 있음
- 클릭 한 번으로 AWS에 Kafka를 배포할 수 있음
- MSK가 리소스를 자동으로 프로비저닝하고 컴퓨팅과 스토리지를 스케일링함
## Apache Kafka at a high level
- Apache Kafka는 데이터를 스트리밍하는 방식
- Kafka 클러스터는 여러 브로커로 구성되고 데이터를 생산하는 생산자는 Kinesis, IoT, RDS 등의 데이터를 클러스터에 주입함
- Kafka 주제로 데이터를 전송하면 해당 데이터는 다른 브로커로 완전 복제됨
- Kafka 주제는 실시간으로 데이터를 스트리밍하고 소비자는 데이터를 소비하기 위해 주제를 폴링함
## Kinesis Data Streams vs. Amazon MSK
### Kinesis Data Streams
- 1MB message size limit
- Data Streams with Shards
- Shard Splitting & Merging
- TLS In-flight encryption
- KMS at-rest encryption
### Amazon MSK
- 1MB default, configure for higher (ex: 10MB)
- Kafka Topics with Partitions
- Can only add partitions to a topic
- PLAINTEXT or TLS In-flight Encryption
- KMS at-rest encryption
## Amazon MSK Consumers
- Apache Flink용 Kinesis Data Analytics를 사용해 Apache Flink 앱을 실행하고 MSK 클러스터의 데이터를 읽어 오면 됨
- AWS Glue로 ETL 작업을 스트리밍해도 됨
- Amazon MSK를 이벤트 소스로 이용하려면 Lambda 함수를 사용할 수 있음
- 자체 Kafka 소비자를 생성해 원하는 플랫폼에서 실행할 수도 있음
  - Amazon EC2 인스턴스나 EC2 클러스터, EKS 클러스터