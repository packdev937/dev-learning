**Keywords**
- 여러 가용 영역에 배포된 Windows 인스턴스
- 공유 Windows 파일 시스템 
**Answer**
- Amazon FSx 구성 각 Windows 인스턴스에 Amazon FSx 파일 시스템 탑재 (SMB 프로토콜)
	- SMB (Server Message Block) 프로토콜은 네트워크에서 파일, 디렉토리, 프린터, 직렬 포트 및 기타 자원으로의 공유 엑세스에 사용되는 프로토콜

**Keywords**
 - 이벤트 데이터를 수신하는대로 처리
 - 특정 순서로 작성
 - 운영 오버헤드 최소화 
**Answer**
 - SQS FIFO 대기열을 생성하여야 "특정 순서" 대로 데이터를 처리
 - 이 외 Amazon SNS 및 Amazon SQS 표준 대기열은 데이터를 순서대로 처리하지 않음.

**Keywords**
- REST API에서 데이터 포인트 엑세스 
- 기존 분석 플랫폼 
**Answer**
- REST API를 지원하는 AWS 서비스는 Amazon API Gateway
- Lambda를 사용한 기존의 데이터 포인터에 엑세스 
**Wrong**
- Amazon Athena는 S3 데이터를 쿼리하기 위한 용도 
- QuickSight는 분석 서비스. 기존 분석 플랫폼이 있다면 무용
- Kinesis Data Analytic는 분석 서비스.  기존 분석 플랫폼이 있다면 무용

**Keywords**
- 고가용성 아키텍쳐
- 단일 AWS 리전 내에서 복원력
- 유지 관리에 최소한의 노력 
**Answer** 
- 여러 가용 영역의 인스턴스를 대상으로 하는 Auto Scaling 그룹에서 지원하는 Network Load Balancer 생성
- 파티션 배치 그룹은 같은 가용 영역의 서버 집합 

**Keywords**
- Amazon CloudFront 배포 
- AWS WAF
- 엑세스를 차단해야 하는 외부 악성 IP 
**Answer**
- 악성 IP 주소를 차단하는 IP 일치 조건을 추가하도록 AWS WAF 구성 수정 
**Wrong**
- CloudFront에는 NACL을 적용할 수 없음
- ALB 뒤의 EC2에서 인식하는 주소는 로드밸런서의 IP 주소
- 보안 그룹은 거부 규칙이 없음 

**Keywords**
- AWS 청구 비용 보고서
- 가장 효율적인 방법
**Answer**
- Cost Explorer에서 보고서 생성 및 보고서 다운로드

**Keywords**
- 3계층 어플리케이션 이미지 
- 확장성, 고가용성, 적은 양의 어플리케이션을 변경하는 솔루션
**Answer**
- 연결 프론트엔드 계층 및 애플리케이션 계층에 다중 AZ 로드 밸런싱 AWS **Elastic Beanstalk** 환경을 제공
	- Elastic Beanstalk은 개발자가 코드만 업로드하면 나머지를 다 처리해주는 서비스
	- 최소한의 구성에 많이 등장 
- 데이터베이스는 다중 AZ DB Instance
- Amazon S3를 사용하여 사용자 이미지를 저장 
**Wrong**
- RDS DB 읽기 전용 복제본은 성능 향상 솔루션 

Keywords
- 로그 파일 7년 보관
- 모든 파일 동시에 엑세스할 수 있는 보고 도구에 의해 분석
- 비용 효율적 
Answer
- S3가 가장 비용 효율적인 스토리지

