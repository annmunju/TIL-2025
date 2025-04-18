
### IAM
1. AWS Root Account
	- Super user
	- Email/PW, AKID/SAK + MFA(Multi-factor Authentication)
2. IAM
	1. User: 영구자격증명. ID/PW (콘솔 접속), AKID/SAK (CLI/SDK) + MFA
	2. Group: User 집합. 정책 상속. 그룹 하위에 그룹 X
	3. Role: 임시자격증명
		1. User
		2. Resource
	- Policy (정책)
		1. Identity
		2. Resource
		- 정책 유형은 Managed(1:N), Inline(1:1)으로 구성
	- 권한 경계
		- 관리를 위해 특정 집합에 경계를 만들어 놓음
		- 그 안에서만 컨트롤 할 수 있음
3. 글로벌 인프라
4. Organization
	- OU (조직 단위): IAM의 그룹과 유사함. 다른 점은 OU 아래 하위 OU가 가능
	- SCP (서비스 제어 정책) : IAM의 권한 경계와 비슷하게 조직의 최대 권한 제어
	- SCP + 사용자 권한 + 권한 경계를 순서대로 정책을 확인함


### 실습 1: AWS CLI 살펴보기
- 글로벌 서비스 / 리전 별 서비스 구분되어있음 (IAM은 글로벌이라 리전 무관하게 사용, EC2는 리전 별로 다르게 생성 가능)

## Network (1)
- VPC - Subnet
	- CIDR 블록 지정시 16~28 사이. 넉넉하게 배정하는 것이 좋음.
	- VPC: 논리적 서비스 격리. 단일 리전에 만들어짐
	- 퍼블릭 서브넷를 만들기 위해 필요한 서비스
		- 인터넷 게이트웨이 (안에 NAT 기능이 포함되어 있어서 퍼블릭 IP를 내부의 프라이빗 IP로 주소 변환을 해줌)
		- 퍼블릭 IP 주소
		- 라우팅 테이블
			- 퍼블릭 라우팅 테이블: VPC 범위를 local로 지정. 0.0.0.0/0(인터넷으로 가는 모든 ip는)은igw-... 지정
			- 프라이빗 라우팅 테이블: VPC 범위를 local로 지정.
	- 서브넷 별로 라우팅 테이블을 따로 설정
	- EC2를 기반으로 하는 서비스는 반드시 VPC가 필요
- NAT Gateway
	- 퍼블릭 서브넷 안에 NAT 지정해두면 프라이빗 서브넷 라우팅 테이블에 NAT를 추가해야함
	- 보안 측면과 IP 주소 관리 측면에서 공인 IP 사용보다 NAT 사용이 유리
	- eip (탄력적 IP 주소): 변하지 않는 Ip 주소로 NAT에는 고정으로 붙음. 비쌈.
	- eni (탄력적 네트워크 인터페이스): 프라이빗 Ip, MAC 주소를 변하지 않고 사용할 수 있음
- 보안

| **특징**         | **네트워크 ACL (Network ACL)**                               | **보안 그룹 (Security Group)**                                    |
| -------------- | -------------------------------------------------------- | ------------------------------------------------------------- |
| **적용 범위**      | 서브넷 단위에서 작동                                              | 인스턴스-리소스 단위에서 작동 (예: EC2 인스턴스, ALB 등) ENI에 연결됨                |
| **상태 관리**      | 상태 비저장 (Stateless): 인바운드와 아웃바운드 트래픽 규칙을 각각 명시적으로 정의해야 함. | 상태 저장 (Stateful): 인바운드 요청이 허용되면 해당 요청에 대한 아웃바운드 응답은 자동으로 허용됨. |
| **규칙 종류**      | 허용(Allow) 및 거부(Deny) 규칙 모두 지원. 기본이 허용                    | 허용(Allow) 규칙만 지원. 기본이 거부                                      |
| **규칙 우선순위**    | 규칙 번호 순서대로 평가, 먼저 일치하는 규칙이 적용됨                           | 모든 규칙을 평가한 후 일치하는 규칙 적용                                       |
| **적용 방식**      | 서브넷 내 모든 리소스에 자동으로 적용                                    | 특정 보안 그룹에 연결된 인스턴스에만 적용                                       |
| **사용 사례**      | 서브넷 전체의 트래픽을 제어하거나 기본적인 네트워크 보안을 설정할 때 사용                | 특정 인스턴스나 리소스의 세밀한 보안 설정이 필요한 경우                               |
| **설정 가능 규칙 수** | 기본적으로 최대 20개의 인바운드 및 20개의 아웃바운드 규칙 지원 (최대 40개까지 확장 가능)   | 최대 60개의 인바운드 및 60개의 아웃바운드 규칙 지원                               |
|                | -                                                        | 보안 그룹 간 인바운드 아웃바운드 트래픽 제어가 가능                                 |

## 컴퓨팅
- EC2
	- 기본은 공유 테넌시(하드웨어 공유)
	- 전용 인스턴스(격리는 되나 서버를 소유하진 X-다시 호스팅했을 때 서버 변경될 수 있음), 전용 호스트(격리와 서버 소유까지 보장)으로 변경 가능
	- AMI (Amazon Machine Image)
	- 인스턴스 스토어: EC2와 같은 공간에 생성되는 블록 스토리지(휘발성) - 빠른 속도
		- EBS : 고정된 블록 스토리지
	- AWS Compute Optimizer: 적절한 인스턴스 클래스를 선택하기 위한 도구. CloudWatch 지표 스캔해서 더 나은 리소스 파악하고 권장사항 제시. 
	- 배치 그룹
		- 클러스터: EC2 인스턴스를 인접하게 배치
		- 분산형: 내결함성
		- 파티션: 분산 처리를 포함하는 서비스(hadoop 등) - 상관 관계가 있는 하드웨어 장애를 방지


###  스토리지
- EBS
	- 볼륨 유형 별로 볼륨당 최대 처리량과 IOPS가 상이함.
	- 버스트: 처리 용량이 몰리는 때에 기존에 사용하지 않는 용량을 남겨둘 수 있음 
- 파일 스토리지 (네트워크 공유. NAS) - EFS, FSx for ...
- 객체 스토리지 - S3 : 메타데이터 + 데이터로 오브젝트 만들어 저장
	- S3 : 저장비용(보관하는 비용. 조금 비쌈), 검색비용(저렴), 데이터 송신 비용 모두 부과. 하나의 파일이 5TB가 넘으면 안됨. > Glacier : 저장비용(저렴), 검색비용(비쌈)
- S3
	- 폴더 별로 구분되어 계층적으로 보이나 com/ 뒤에 위치한 모든 접두사가 키로 사용되어 검색속도가 (EFS보다) 빠름
	- 버킷에 일부 사용자에게 권한 부여를 위해 인라인 정책 많이 적용
	- Access Point: 지정된 접근 포인트. 접속할 수 있는 IAM 역할에만 사용할 수 있도록 경로를 줄 수 있음
	- 암호화 키 유형: S3 관리형 키, KMS (키 + 이중 암호화: s3에 role 적용 필요), 고객 제공 키
	- Intelligent-Tiering: 사용량을 확인하고 자동 전환이 됨. 
		- 수명주기정책 : 30일보다 오래된 객체 IA 이동 -> 90일 이상이면 Glacier ... 등으로 규칙을 지정할 수 있음
	- 버전 관리 제공. 사용시 데이터 잠금 가능 
	- 동일 리전 혹은 다른 리전으로 복제 가능
- EFS or FSx
	- 여러 인스턴스가 동일한 스토리지를 사용해야 하는 경우
	- 가용영역 무관하게, 온프레미스에서도 사용 가능
	- general purpose 버스터 처리량 모드를 제공
- 스토리지 마이그레이션
	- 스토리지 게이트웨이:어플라이언스에 캐싱 기능. 인터넷 망 통해서 전달
	- 데이터 싱크: 빠른 속도로 전송. 암호화 및 무결성 지원.
	- 스노우볼 엣지

### DB
- RDB
	- EC2+EBS를 사용하는 것처럼 구성. DB 완전 관리
	- multi AZ로 고가용성 확보. 
	- standby
	- cluster: 3개의 AZ (두개의 복제본 - 읽기 분산) -> RDS proxy 같은것을 앞에 두고 분산으로 수행할 수 있도록 두는 것을 권장. 같은 리전에서밖에 못씀
	- read - replica : 읽기 전용을 최대 5개까지 만들 수 있음. 다른 리전에서도 사용 가능
	- 자동 백업
	- Aurora Serverless : 요구사항 기반으로 용량 자동 확장 - 축소하는 크기 조정 구성
	- **오로라 확인 필요**
- DynamoDB
	- 4kb -> 1rcu 읽기 용량 단위
	- 온디맨드 요금제 : 쓴만큼 지불
	- 프로비저닝됨 요금제 : 구간 단위로 미리 약정
	- 최종 일관성
- 캐시
	- 일반적 캐싱 전략: 캐시 요청시 응답 -> 없으면 데이터베이스로 다시 요청 -> 그다음 어플리케이션이 캐시에 저장
	- 라이트 스루: 애플리케이션이 데이터베이스와 캐시에 데이터 동시 기록
	- elasticache
	- DynamoDB Accelerator(DAX) : dynamodb와 캐시 연결. 한국은 서비스 X
	- 마이그레이션 schema conversion tool

## 모니터링
- CloudWatch
	- 리소스 사용량 등 확인 가능
	- 로그 : 소스코드에서 나오는 로그 내용을 저장하고 액세스
- CloudTrail : 사용자 활동 및 API 사용량 추적
- VPC flow log : 트래픽 보안 제어를 위해 확인할 수 있는 로그
### ELB 로드 밸런서 유형
- Application : 7계층에서 패킷 열어서 확인 후에 http 단에서 로드벨런싱
	- 소스 IP가 소실됨
	- 라운드로빈 방식
	- SSL 오프로드 (뒤로 보내기 전에 앞에서 미리서 확인)
- Network : 4계층에서 패킷 열어서 TCP UDP
	- 소스 IP가 유지됨
	- 계속 같은 곳으로 보내는 형식
	- SSL 오프로드 
- NLB + ALB를 동시에 사용해서 소스 IP 주소를 유지할 . 수있도록 함
- Gateway : 바로 로드벨런싱 하는게 아니라 어디 들렸다가 오는거. 지능형 공격 차단 등
- 타겟 그룹 안에서 알아서 분산으로 트래픽을 보내줌

## 자동화
- 코드형 인프라 IaC
	- 드리프트: 코드 변경 사항
- cloudformation
	- ami와 같은 서비스는 리전별로 아이디가 상이한 경우가 있음 -> 작동 yaml 파일 작성시에 리전과 id 적절하게 맵핑 해야함.
	- 탬플릿 작성시 스택을 나눠서 사용하는 것을 권장
- systems manager: 관리를 목적으로 함
	- 인프라 구성을 확인하고 관리할 때 사용
	- session manager
	- patch manager 온프렘 + 클라우드 에이전트 버전 확인 및 업데이트
	- feature store 다른 리소스에 붙는 파라미터를 가져와서 사용할 수 있게 함. DB id pw 소스코드에 올리는 대신 가져올 수 있게 설정
- Amazon Q: 자동 코드 완성


## 컨테이너
- 레지스트리 : ECR
- 오케스트레이션 도구 : ECS, EKS
- 컨테이너 호스팅 : Fargate(lambda와 유사한 역할) / EC2로 호스팅

### ECS
1. 테스크 정의 : 작업 하나에 들어갈 이미지를 지정. 작업 하나에 이미지 여러개 들어갈 수 있음. (k8s의 pod 개념과 유사)
2. 서비스 정의 : 서비스 용량, 실행에 필요한 것들을 정의. 배치 어떻게 할지, 분산 어떻게 할지, 죽으면 어떻게 띄울지...
- 이렇게 정의된 것 단위를 "클러스터"라고 부르고 사용

### EKS
- 클러스터 프로비저닝 -> Fargate 혹은 EC2 사용하여 노드 배포 -> EKS 연결 -> 어플리케이션 실행


## 서버리스 서비스
### Lambda
1. 코드 작성하고 올려주기
	- handler 함수 안에 넣어주기
	- 함수 바깥에 전역 변수 정의하면 반복 사용시 실행 시간 단축 가능
2. 함수 실행 트리거 정의 
	- 직접 호출 - API GW로 호출할 수 있음
	- 예약
	- 이벤트
3. 실행 요청 후 자동으로 런타임 실행 
- 할 일 및 고려할 점
	- 런타임 설정 
	- IAM 권한 설정
	- 메모리는 128MB ~ 10GB로 설정
	- timeout ~15분
	- cold start / warm start (미리 설정하면 프로비저닝 해둠)
	- 실행 후 몇 분 간은 환경을 내리지 않고 둠.

### API Gateway
- 외부에서 오는 api 요청에 대해서 aws 내부 서비스에 어느 위치로 호출할지 결정
- ddos 보호 및 제한(WAF, Shield), 백엔드 요청 인증 & 권한 부여(cognito), 서드파티 개발자에 의해 api 사용을 조절하고 측정 및 수익화(트레픽 수 제한 가능)
- 응답 캐싱 기능 포함됨

### SQS
- 발신자와 수신자 사이의 버퍼 역할. 느슨한 결합
- 배달 못한 편지 대기열 확인하여 에러 처리
- 대기열 유형
	- 표준 : 초당 무제한 API 호출. 순서 미지정 1회 이상
	- FIFO : 초당 제한된 API 호출 횟수. 순서 지정 1회만!
- 대기열 구성 최적화
	- 큐가 polling 하면 N초 가량 처리되고 queue가 삭제됨. 
	- 긴 polling (기다렸다 폴링하기 - 오래걸림, 기다렸다가 여러개 처리) , 짧은 polling (왔다갔다 - 비쌈, 하나만 처리)
- 메시지 대기열 사용 예
	- 서비스 간 통신
	- 비동기 작업 항목 등

### SNS
- 게시자와 구독자
	- 이벤트 발생하면 게시자가 주제를 만듦 -> 주제에 포함된 메시지를 구독자에게 보내줌
	- 구독자는 lambda, sqs, http, email이 포함됨
- 표준, FIFO
- 한번 메시지 보내면 끝이라서 뒤에 SQS를 붙이기도 함
- ? 이벤트 브릿지랑 어떤게 다른거지 ?

| 특성        | SNS       | SQS      |
| --------- | --------- | -------- |
| 메시지 지속성   | 아니요       | 예        |
| 전송 메커니즘   | 푸시 (수동적)  | 폴링 (능동적) |
| 생산자 및 소비자 | 게시자 및 구독자 | 발송 및 수신  |
| 배포 모델     | 일대다       | 일대일      |

### Kinesis
- 스트리밍 데이터 수집 및 분석 (실시간). aws 서비스에서 유일한 실시간 서비스

#### data streams
- 1MBps로 들어오고 2MBps로 나감. N개의 샤드(분할된 데이터 구간)로 데이터를 병렬 송/수신.
- 결정해야 할 사항
	- 샤드 개수
	- 어디로 보낼지 결정 (데이터 처리)
#### firehose
- 준 실시간 데이터를 저장

### Step Functions
- 복잡한 분산 워크플로의 오케스트레이션
- 람다를 순차적으로 연결할 수 있음. 분기점 설정해서 단계를 분할할 수 있음.
- 시각적 워크플로로 사용한 마이크로 서비스 조정

## Network (2)
- endpoint
	- gateway endpoint : s3, dynamoDB (현재는 지원하지 않음) <- 라우팅 테이블에 연결해줬어야함
	- interface endpoint
- vpc peering - 무료
	- vpc 간 연결 지원. cidr 주소 반드시 상이해야함
	- 라우팅 테이블에 추가해줘야함
	- vpc 통해서 가는 것(전이식 peering) 안됨 (무조건 직통 연결로만 통신 가능)
- On-premise 연결 방법
	- virtual private gateway(VGW)를 vpc에 붙이기 (IGW 붙이듯이) 
	- site to site vpn : 암호화로 접속하는 인터넷 vpn
		- 온프렘에 에이전트 설치해야함
		- 최대 연결 속도는 1.25Gbps
	- direct connect (DX): direct connect location 까지 광케이블 깔아서 사용. 보안은 X.
		- 온프렘에 에이전트 설치 + DX에 고객 케이지에 에이전트 설치
		- 연결 속도 1/10/100Gbps
	- Transit Gateway : 허브 역할. vpc, dcg, vpn, tgw 다수의 가상 네트워크 망과 연결해서 사용. 못가게도 설정할 수 있음.
	- out posts : 서버 랙을 자사에 데센에 탑재. AWS에서 제공하는 컴퓨팅/DB 서비스를 만들어줌. (로컬 존과 유사) 지연시간이 매우 예민한 기업(금융권 등)에서 주로 사용

## 엣지 서비스
- Route53 : DNS 서비스인데 라우팅 기능을 포함한...
	- 라우팅 기능- 프라이빗 호스팅 영역: 내부에서 VPC 리소스 라우팅. 온프라미스와도 통합하거나, 리전 간 라우팅 가능
		- 지리적 위치 라우팅 : 사용자의 지리적 위치로 보냄 (직접 설정 가능)
		- 지리 근접 라우팅 : 위도 경도 기반으로 지리적으로 근처에 있는 곳으로 보냄
		- 지연 시간 기반 라우팅 : 지연 시간 낮은 곳으로 보냄
		- 다중값 응답 라우팅 : 여러 선택지를 주고 사용자가 선택하도록
		- 가중치 기반 라우팅 : 새로운 프로덕션 환경을 했을 때 트래픽을 조절해서 라우팅 + 로드벨런서와 함께 사용
- CloudFront
	1. 오리진 선택: S3 버킷, ELB, 사용자 오리진(EC2, 온프렘 서버)
		- 동적 컨텐츠를 송수신 할 때 안정적인 대역폭 유지하기 위해서는 캐싱 해야함. 이럴 때 ELB 로드 밸런서 붙임.
	2. 캐시 동작 정의
		- 어떻게 얼마나 저장할지 선택
		- 캐시 키 설정 :트레픽 해더 등을 이용해서 오리진을 세분화 할 수 있음
		- (선택 사항) 람다 엣지 함수 연결, WAF 제어, 사용자 지정 도메인 이름 추가 등
	3. 배포 생성 : 전 세계에 위치한 edge location에 배포됨 -> 요청 오면 거기 저장

### DDos 보호 (보안)
- DDos 공격: 감염된 호스트가 공격에 참여하여 목표 대상에 대량 요청을 생성
	- 일반적으로 7계층 (application) 공격
	- 4계층 공격 :SYN floods (네트워크 계속 요청해서 다른 일들을 하지 못하게)
- Shield 
	- Standard : 3,4 계층 방어
	- Advanced : 3~7계층 방어
- WAF
	- 웹 어플리케이션 방화벽
	- CloudFront, ALB, API GW, AppSync, Cognito
	- ACL 규칙 서술 정의해서 해당 조건에 해당하는 경우 못들어오게 하기
- Firewall Manager - ( WAF, VPC SG, Shild, Network Firewall, ACL, IAM ) 중앙에서 기준 보안 설정 가능

## 백업 및 복구
- 가용성 개념 : 고가용성, 내결함성, 백업, 재해 복구 고려
- 복구 시점 목표(RPO)와 복구 시간 목표(RTO)
	- 복구 시점 목표때 까지 백업 데이터 복구하는 것을, 복구 시간 목표때 까지 완료하기
- 스토리지 복제
- 컴퓨팅 백업 : AMI, 컨테이너 이미지로 생성
- 장애 조치 네트워크 설계
- 데이터베이스 백업 및 복제
- backup 서비스
	- organization 사용시 함께 사용 가능
- DR 사례
	- 백업 및 복원 : Data만 옮겨둠. 시간 짧게 걸림
	- 파일럿 라이트 : DB 복제 하고 프로비저닝 하는데 실행은 X
	- 웜 스텐바이 : 저용량으로 계속 실행해두기