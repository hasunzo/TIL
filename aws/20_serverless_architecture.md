# Micro Services architecture
- 서비스 개발 수명 주기를 줄이고자 사용
- 각 서비스가 독립적으로 확장하며 코드 리포지토리를 갖추길 원한다면 마이크로서비스 아키텍처를 사용
- You are free to design each micro-service the way you want
- Synchronous patterns: API Gateway, Load Balancers
- Asynchronous patterns: SQS, Kinesis, SNS, Lambda triggers (S3)
## Challenges with micro-services:
- 새로운 마이크로서비스를 생성할 때마다 오버헤드 발생
- 서버 밀도나 사용률을 최적화하는 데 어려움 (density/utilization)
- 여러 버전의 마이크로서비스를 동시에 가동하려면 복잡함
- 여러 서비스와 통합하려다 클라이언트 코드 요구사항이 급증하기도 함
## Some of the challenges are solved by Serverless patterns:
- API Gateway, Lambda scale automatically and you pay per usage
- You can easily clone API, reproduce environments
- Generated client SDK through Swagger integration for the API Gateway

# Software updates offloading
- EC2에서 작동하는 애플리케이션이 있고 종종 소프트웨어 업데이트를 배포함
- 컴퓨터에서 패치를 다운로드해 EC2에 설치하는 상황
- 소프트웨어가 새로 업데이트되면 요청을 많이 받게 됨 -> 콘텐츠는 네트워킹을 통해 다수에게 배포되는데 비용이 많이 든다.
- 애플리케이션을 변경하거나 아키텍팅을 다시 하는 일 없이 비용과 CPU를 최적화하고 싶을 때 어떻게 해야 할까?
### CloudFront를 상위에 두면 된다
- 엣지에서 소프트웨어 업데이트 파일은 캐시에 저장됨(업데이트 파일은 변하지 않기 때문)
- EC2 인스턴스는 서버리스가 아니지만 CloudFront는 서버리스라서 확장함
- ASG가 많이 확장하지 않아 EC2와 네트워크, EFS 비용을 크게 절감하고 가용성도 확보됨
- CloudFront가 기존 애플리케이션의 확장성을 높이고 비용을 절감하는 데 용이하다는 점을 기억하자
- 엣지에서 캐싱을 활용하는 정적 콘텐츠가 대부분인 경우에!