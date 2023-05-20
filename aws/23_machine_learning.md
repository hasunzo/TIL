# Amazon Rekognition
- Find objects, people, text, scenes in images and videos using ML
- 얼굴을 분석하고 비교하여 사용자 확인을 하며 이미지 내의 인물 수를 셀 수 있음
- 익숙한 얼굴을 저장해 자체 데이터베이스 생성
- 이미지 속 인물이 궁금할 때 유명인 얼굴의 데이터베이스와 비교
- Use cases:
  - Labeling
  - Content Moderation
  - Text Detection
  - Face Detection and Analysis (gender, age, range, emotions...)
  - Face Search and Verification
  - Celebrity Recognition
  - Pathing (ex: for sports game analysis)
## Amazon Rekognition - Content Moderation
- 이미지나 비디오에서 부적젏거나 원치 않거나 불쾌감을 주는 콘텐츠를 탐지하는 기능
- Amazon Rekognition가 이미지를 분석한 후 플래그를 띄우도록 항목의 최소 신뢰도 임곗값을 설정하면 됨

# Amazon Transcribe
- 자동으로 음성을 텍스트로 변환
- 오디오를 넣으면 자동으로 텍스트로 변환됨
- 자동 음석 인식 (ASR) 딥러닝 프로세스를 사용하여 음성을 텍스트로 매우 빠르고 정확하게 변환
- Redaction을 사용하여 개인 식별 정보 (PII)를 자동으로 제거
  - 나이, 이름, 사회보장번호가 있다면 자동으로 제거됨
- 다국어 오디오를 자동으로 언어 식별할 수 있음

# Amazon Polly
- 텍스트를 음성으로 변환하여 음성 작동 애플리케이션을 만들 수 있음
## Amazon Polly - Lexicon & SSML
- 발음 어휘 목록을 사용해 사용자 지정 발음을 생성할 수 있음
  - AWS와 같은 두문자어는 'AWS'를 그대로 읽는 대신
  - Amazon Web Services로 풀어서 읽도록 함
- SSML 기능은 음성 합성 마크업 언어
  - 보다 다양한 사용자 지정 음성을 만들 수 있는 기능
  - 특정 단어나 구절을 강조할 수 있고 음성학적 발음을 구현하거나 숨소리를 넣거나 속삭이듯 말할 수 있음
  - 뉴스 진행자 스타일로 말할 수도 있음
  - '\<break time="3s" />'

# Amazon Translate
- 언어 번역 기능을 제공
- Translate 를 통해 콘텐츠를 현지화할 수 있음
- 해외 사용자를 위한 웹사이트와 애플리케이션 등에 적용 가능
- 대량의 텍스트를 효율적으로 번역

# Amazon Lex & Connect
## Amazon Lex (same technology that powers Alexa)
- 'Alexa, 내일 날씨가 어때?' -> '날씨가 화창합니다'
- 자동 음석 인식
- 음성을 인식하는 ASR이라서 말을 텍스트로 바꿔줌
- Lex가 자연어 이해를 통해 말의 의도를 파악하고 문자을 이해함
- 챗복 구축이나 콜 센터 봇 구축에 도움을 줌
## Amazon Connect
- 가상 고객 센터
- 전화를 받고 고객 응대 흐름을 생성하는 클라우드 기반 서비스
- 다른 고객 관계 시스템 혹은 관리 시스템인 CRM 및 AWS 서비스와 통합할 수 있음
- Amazon Conenct의 장점
  - 기존 고객 센터 방식에 비해 초기 비용이 없고 비용이 약 80% 저렴함
- 전체 흐름
  - 일정을 예약하기 위해 전화를 검
  - Amazon Connect에서 지정한 번호로 통화
  - Amazon Connect로 전화가 연결되면, Lex는 이 통화의 모든 정보를 스트리밍하여 통화 목적을 이해함
  - Lex가 올바른 Lambda 함수를 호출
  - 이 Lambda 함수는 전화를 건 사람이 내일 오후 3시에 Tom과 미팅을 원함을 이해하고 CRM으로 이동하여 코드를 작성해서 미팅 일정을 잡음
- Lex는 ASR이고 Connect는 고객 센터

# Amazon Comprehend
- 자연어를 처리하는 NLP 
- 시험에서 NLP가 보이면 Amazon Comprehend를 생각하기
- 완전 관리형 서버리스 서비스
- 머신 러닝을 사용하여 텍스트에서 인사이트와 관계를 도출
- 텍스트 언어를 이해할 수 있고 텍스트에서 주요 문구, 장소 및 사람, 브랜드, 이벤트를 추출
- 분석 중인 텍스트가 긍정적인지 부정적인지 파악하는 감정 분석을 할 수 있음
- 토큰화 및 품사를 사용하여 텍스트를 분석할 수 있고 음성을 식별함

# Amazon Comprehend Medical
- 비정형 의료 텍스트에서 유용한 정보를 탐지해서 반환해 주는 서비스
- 의사 소견서나 퇴원 요약서, 검사 결과서 의료 사례 기록을 발견하면 자연어 처리를 사용해 텍스트를 탐지
- 문서와 문서 속의 보호된 개인 건강 정보(PHI)를 DetectPHI API로 탐지해냄
- 아키텍처 관점
  - Amazon S3에 문서를 저장
  - Comprehend Medical API를 실행
  - Kinesis Data Firehose로 실시간 데이터를 분석하거나
  - Amazon Transcribe를 사용해 음성을 텍스트로 변환한 후 
  - 텍스트 형식의 콘텐츠를 Amazon Comprehend Medical 서비스에 전달

# Amazon SageMaker
- 완전 관리형 서비스
- 머신 러닝 모델을 구축하는 개발자와 데이터 과학자를 위한 서비스
- 높은 수준의 머신 러닝 서비스로 조직의 실제 개발자와 데이터 과학자가 머신 러닝 모델을 만들고 구축하기 위해 사용
## 시험 점수를 예측할 모델 구축 예시
- 개발자이거나 데이터 과학자라면 학생들의 실제 시험 점수에 관한 모든 데이털르 수집
- 가능한 많은 데이터를 수집하고 해당 데이터를 라벨링
- 어떤 열이 무슨 데이터와 대응하는지 정해야 하고 점수도 필요
- 학생마다 특정한 점수를 얻게 됨
- 이렇게 수집한 데이터를 기반으로 점수가 어떨지 예상할 수 있음
- 라벨링 후 머신 러닝 모델을 구축
- 과거 데이터를 통해 점수를 예측하는 모델
- 머신 러닝 모델을 구축한 후에는 훈련 및 조정이 필요
  - 데이터와 출력이 더 들어맞도록
- SageMaker는 전 과정에서 도움을 줌
- 라벨링, 구축, 그리고 훈련 및 조정까지
- 새로운 데이터가 들어오면 기존에 가지고 있는 데이터에 근거하여 새로운 데이터를 예측함
- 이 모든 과정, 즉 라벨링과 구축, 훈련 및 조정, 적용 모두 SageMaker에서 가능

# Amazon Forecast
- 에측을 도와주는 기능
- 예를 들면 미래의 비옷 판매를 예측
- 예측이 필요한 것은 모두 사용 사례가 됨
- 제품 수요 계획과 재무 계획 자원 계획 등
- Amazon S3에 데이터를 업로드한 뒤에 Amazon Forecast 서비스를 시작
  - 그러면 예측 모델이 생성되고 이 모델을 사용하여 데이터를 예측함
  
# Amazon Kendra
- 머신 러닝으로 제공되는 완전 관리형 문서 검색 서비스
- 문서 내에서 답변을 추출할 수 있게 도와줌
- text, pdf, HTML PowerPoint, MS Word, FAQ
- Amazon Kendra는 이런 문서를 인덱싱하여 머신 러닝으로 작동되는 지식 인덱스를 내부적으로 구축
- 사용자의 상호 작용 및 피드백에서 학습하고 선호되는 검색 결과를 내놓은 증분식 학습
- 검색 결과를 조정할 수도 있음
- 데이터의 중요성 및 새로움 또는 사용자 정의 필터를 기반으로 조정 가능
- 시험에서 문서 검색 서비스를 보게 된다면 Amazon Kendra

# Amazon Personalize
- 실시간 맞춤화 추천으로 애플리케이션을 구축
- 맞춤화된 제품 추천 그리고 재순위화 또는 맞춤화된 직접 마케팅
- 검색한 내용이나 구매 내역 및 사용자 관심 등을 기반으로 나오는 결과
- Amazon S3로부터 입력 데이터를 읽음 (사용자 상호 작용 등의 데이터)
- Amazon Personalize API를 사용하여 Amazon Personalize 서비스에 실시간 데이터를 통합할 수 있음
- 웹사이트, 애플리케이션 및 모바일 앱 맞춤형 API 활용
- 맞춤화를 위해 SMS나 이메일을 보낼 수도 있음
- 이런 모든 걸 통합
- 추천 및 맞춤화된 추천을 위한 머신 러닝 서비스가 나오면 Amazon Personalize

# Amazon Textract
- 텍스트를 추출
- 텍스트, 손글씨, 또는 스캔을 한 문서의 데이터를 추출

# AWS Machine Learning - Summary
- Rekognition: face detection, labeling, celebrity recognition
- Transcribe: audio to text (ex: subtitles)
- Polly: text to audio
- Translate: translations
- Lex: build conversational bots - chatbots
- Connect: cloud contact center
- Comprehend: natural language processing
- SageMaker: machine learning for every developer and data scientist
- Forecast: build highly accurate forecasts
- Kendra: ML-powered search engine
- Personalize: real-time personalized recommendations
- Textract: detect text and data in documents
