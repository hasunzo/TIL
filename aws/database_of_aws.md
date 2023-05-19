# Amazon RDS - Summary
- Managed PostgreSQL / MySQL / Oracle / SQL Server / MariaDB / Custom
- Provisioned RDS Instance Size and EBS Vloume Type & Size
- Auto-scaling capability for Storage
- Support for Read Replicas and Multi AZ
- Security through IAM, Security Groups, KMS, SSL in transit
- Automated Backup with Point in time restore feature (up to 35 days)
- Manual DB Snapshot for longer-term recovery
- Managed and Scheduled maintenance (with downtime)
- Support for IAM Authentication, integration with Secrets Manager
- RDS Custom for access to and customize the underlying instance (Oracle & SQL Server)
- Use case: Store relational datasets (RDBMS / OLTP), perform SQL queries, transactions

# Amazon Aurora - Summary
- Compatible API for PostgreSQL / MySQL, separation of storage and compute
- Storage: data is stored in 6 replicas, across 3 AZ - highly available, self-healing, auto-scaling
- Compute: Cluster of DB Instance across multiple AZ, auto-scaling of Read Replicas
- Cluster: Custom endpoints for writer and reader DB instances
- Same security / monitoring / maintenance features as RDS
- Know the backup & restore options for Aurora
- Aurora Serverless - for unpredictable / intermittent workloads, no capacity planning
- Aurora Multi-Master - for continuous writes failover (high write availability)
- Aurora Global: up to 16 DB Read Instances in each region, < 1 second storage replication
- Aurora Machine Learning: perform ML using SageMaker & Comprehend on Aurora
- Aurora Database Cloning: new cluster from existing one, faster than restoring a snapshot
- Use case: same as RDS, but with less maintenance / more flexibility / more performance / more features

# Amazon ElastiCache - Summary
- Managed Redis / Memcached (similar offering as RDS, but for caches)
- In-memery data store, sub-millisecond latency
- Must provision an EC2 instance type
- Support for Clustering (Redis) and Multi AZ, Read Replicas (sharding)
- Security through IAM, Security Groups, KMS, Redis Auth
- Backup / Snapshot / Point in time restore feature
- Managed and Scheduled maintenance
- Requires some application code changes to be leveraged
  - 시험에서 코드 변경이 필요 없는 캐싱 솔루션을 물어본다면 ElastiCache는 선택하면 안됨!
- Use Case: Key/Value store, Frequent reads, less writes, cache results for DB queries, store session data for websites, cannot use SQL

# Amazon DynamoDB - Summary
- AWS 독점 기술. 관리형 서버리스 NoSQL database, millisecond latency
- 선택할 수 있는 용량모드: 
  - provisioned capacity: 점진적으로 늘어나거나 점진적으로 줄어드는 이중(double) 워크로드가 있을 때 유용
  - on-demand capacity: 워크로드를 예측하기 어려울 때나 데이터베이스의 수요가 급증할 때 유용
- Can replace ElastiCache as a key/value store (storing session data for example, using TTL feature)
- Highly Available, Multi AZ by default, Read and Writes are decoupled, transaction capability
- DAX cluster for read cache, microsecond read latency
- Security, authentication and authorization is done through IAM
- Event Processing: DynamoDB Streams to integrate with AWS Lambda, or Kinesis Data Streams
- Global Table feature: active-active setup(다중활성 설정)
- Automated backups up to 35 days with PITR (restore to new table), or on-demand backups
- Export to S3 without using RCU within the PITH window, import from S3 without using WCU
- 시험에서 스키마를 빠르게 전개해야 하는 경우가 나온다면 유연한 데이터베이스 스키마를 사용해야 하므로 DynamoDB가 탁월한 선택
- Use Case: Serverless applications development (small documents 100s KB), distributed serverless cache, doesn't have SQL query language available

# Amazon S3 - Summary
- S3 is a .. key / value store for objects
- Great for bigger objects, not so great for many small objects
- Serverless, scales infinitely, max object size is 5 TB, versioning capability
- Tiers: S3 standard, S3 Infrequent Access S3 Intelligent, S3 Glacier + lifecycle policy
- Features: Versioning, Encryption, Replication, MFA-Delete, Access Logs...
- Security: IAM, Bucket Policies, ACL, Access Points, Object Lambda, CORS, Object/Vault Lock
- Encryption: SSE-S3, SSE-KMS, SSE-C, client-side, TLS in transit, default encryption
- Batch operations on objects using S3 Batch, listing files using S3 Inventory
- Performance: Multi-part upload, S3 Transfer Acceleration, S3 Select
  - Multi-part upload -  파일을 병렬식으로 업로드
  - S3 Transfer Acceleration - S3 파일을 한 리전에서 다른 리전으로 더 빠르게 전송
  - S3 Select-  Amazon S3에서 필요한 데이터만 검색
- Automation: S3 Event Notifications (SNS, SQS, Lambda, EventBridge)
  - 예를 들면 Amazon S3 버킷에서 새로운 객체가 생성되는 이벤트에 반응할 수 있음
- Use Cases: Static files, eky value store for big files, website hosting

# DocumentDB
- Aurora is an "AWS-implementation" of PostgreSQL / MySQL ..
- DocumentDB is the same for MongoDB (which is a NoSQL database)

- MongoDB is used to store, query, and index JSON data
- similar "deployment concepts" as Aurora
- Fully Managed, highly available with replication across 3 AZ
- DocumentDB storage automatically grows in increments of 10GB, up to 64TB

- Automatically scales to workloads with millions of requests per seconds

# Amazon Neptune
- Fully managed graph database
- A popular graph dataset would be a social network
- Highly available across 3 AZ, with up to 15 read replicas
- Neptune은 소셜 네트워크처럼 고도로 연결된 데이터 셋을 사용하는 애플리케이션을 구축하고 실행하는 데 사용됨
- Neptune은 그래프 데이터 셋에서 복잡하고 어려운 쿼리를 실행하기에 최적화되어 있음
- 데이터베이스에 최대 수십억 개의 관계를 저장할 수 있고, 그래프를 쿼리할 때 지연시간은 밀리초
- 여러 가용 영역에 걸친 애플리케이션에서도 매우 가용성이 높으며 지식 그래프를 저장하는 데도 뛰어남
- 그래프 데이터와 관련된 게 나오면 Neptune

# Amazon Keyspaces (for Apache Cassandra)
- Apache Cassandra is an open-source NoSQL distributed database

- A managed Apache Cassandra-compatible database service
- Serverless, Scalable, highly available, fully managed by AWS
- Automatically scale tables up/down based on the application's traffic
- Tables are replicated 3 times across multiple AZ
- Using the Cassandra Query Language (CQL)
- Single-digit millisecond latency at any scale, 1000s of requests per second
- Capacity: On-demand mode or provisioned mode with auto-scaling
- Encryption, backup, Point-In-Time Recovery (PITR) up to 35 days

- Use cases: store IoT devices info, time-series data, ...

# Amazon QLDB
- QLDB stands for "Quantum Ledger Database"
- Ledger(원장)은 금융 트랜잭션을 기록하는 장부. 따라서 QLDB는 금융 트랜잭션 원장을 갖게 되는 것
- Fully Managed, Serverless, High available, Replication across 3 AZ
- Used to review history of all the changes made to your application data voer time
- Immutable system: 데이터베이스에 무언가를 쓰면, 삭제하거나 수정할 수 없음
- 일반 원장 블록체인 프레임워크보다 2~3배 더 나은 성능을 얻을 수 있음
- Amazon 관리형 블록체인이라는 또다른 데이터베이스 기술도 있음
- QLDB와 관리형 블록체인의 차이점은 QLDB에는 탈중앙화 개념이 없다는 것
- 즉, Amazon 소유의 중앙 데이터베이스에서만 저널을 작성할 수 있음 - 많은 금융 규제 규칙들을 따르는 것
- 즉, QLDB와 관리형 블록체인의 차이점은, QLDB에는 중앙 권한 구성 요소가 있다는 것 = 원장

# Amazon Timestream
- Fully managed, fast, scalable, serverless time series database
- Automatically scales up/down to adjust capacity
- 시계열은 시간 정보를 포함하는 포인트의 모음을 말함
- Timestream를 사용하면 데이터베이스의 용량을 자동으로 확장, 축소할 수 있고 매일 수조 건의 이벤트를 저장 및 분석할 수 있음
- 시계열 데이터가 있을 때는 관계형 데이터베이스보다 시계열 데이터베이스를 사용하는 것이 훨씬 빠르고 저렴함
- 쿼리를 예약하고 다중 척도 레코드도 얻을 수 있음
- SQL과도 완벽 호환
- 최신 데이터는 메모리에 저장되고 과거 데이터는 비용 효율적인 스토리지 계층에 저장됨
- 시계열 본석 기능이 있어 거의 실시간으로 데이터를 분석하고 패턴을 찾을 수 있음
- AWS의 다른 데이터베이스들처럼 전송 중 데이터와 저장 데이터의 암호화를 지원함
- Timestream의 사용 사례로는 IoT 애플리케이션, 운영 애플리케이션, 실시간 분석 등 시계열 데이터베이스와 관련된 모든 곳에서 사용할 수 있음