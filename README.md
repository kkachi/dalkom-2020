# AWS 웹 3티어 구축 실습

## 목차.
- 실습1. AWS VPC 구축 및 서브넷 구성 (40분)
- 실습2. EC2, RDS 생성 및 접 (40분)
- 실습3. 로드밸런서, 오토스케일링 구성 (40분)
- Terminate Resources (10분)

* * *

## 실습1. AWS VPC 구축 및 서브넷 구성

* * *

### 1-1. 개요 
- 본 실습 섹션에서는 `[Amazon Virtual Private Cloud(VPC)]`를 사용하여 자체 VPC를 생성합니다.
- AWS VPC에 구성 요소를 추가하여 사용자 정의된 네트워크를 구성합니다.
- AWS EC2 인스턴스에 대한 보안 그룹을 생성합니다.
- 웹 서버를 실행하고 이를 VPC에서 시작하도록 EC2 인스턴스를 구성 및 사용자 정의합니다.

<br>

### 1-2. 목표
본 실습을 마친 후 다음을 할 수 있게 됩니다.
- AWS VPC를 생성
- 서브넷(Subnet)을 생성
- 보안그룹을 생성

![image](/img/1.png)

<br>

### 1-3. 사전 조건
본 실습을 위해서는 다음이 필요합니다.
- Microsoft Windows, Mac OS X 또는 Linux(Ubuntu,SuSE, Red Hat)가 실행되며 Wi-Fi가 되는 컴퓨터 사용
- Chrome, Firefox 또는 IE9 이상 버전과 같은 인터넷 브라우저 (IE9 이전 버전은 지원 안됨)

<br>

### 1-4. 기간 
본 실습에는 약 40분 정도가 소요

<br> 

### Task 1 : VPC 생성 

- 개요
    - 이 섹션에서는 VPC를 생성합니다.

<br>

### Task 1.1 : VPC, 서브넷 생성 및 인터넷 게이트웨이 생성
- 1.1.1 `AWS Management Console`의 `서비스` 메뉴에서 `[VPC]`를 클릭
- 1.1.2 좌측 상단에 위치한 `[VPC]`를 클릭 후 `[VPC 생성]`을 클릭하고 다음 정보를 입력
    - `[이름 태그]` : **testVPC**
    - `[IPv4 CIDR 블록:*]` : **10.0.0.0/16**
- 1.1.3 좌측 상단에 위치한 `[서브넷]`를 클릭 후 `[서브넷 생성]`을 클릭하고 다음 정보를 입력 (총 6개의 서브넷 생성)
    - `[이름 태그:]` : **testVPC-public-subnet-2a** 
    - `[VPC:]` : **testVPC** 
    - `[가용 영역:*]` : **ap-northeast-2a** 
    - `[IPv4 CIDR:*]` : **10.0.0.0/24**
    - `[이름 태그:]` : **testVPC-public-subnet-2c** 
    - `[VPC:]` : **testVPC** 
    - `[가용 영역:*]` : **ap-northeast-2c** 
    - `[IPv4 CIDR:*]` : **10.0.1.0/24**
    - `[이름 태그:]` : **testVPC-private-subnet-2a** 
    - `[VPC:]` : **testVPC** 
    - `[가용 영역:*]` : **ap-northeast-2a** 
    - `[IPv4 CIDR:*]` : **10.0.2.0/24**
    - `[이름 태그:]` : **testVPC-private-subnet-2c** 
    - `[VPC:]` : **testVPC** 
    - `[가용 영역:*]` : **ap-northeast-2c** 
    - `[IPv4 CIDR:*]` : **10.0.3.0/24**
    - `[이름 태그:]` : **testVPC-rds-subnet-2a** 
    - `[VPC:]` : **testVPC** 
    - `[가용 영역:*]` : **ap-northeast-2a** 
    - `[IPv4 CIDR:*]` : **10.0.4.0/24**
    - `[이름 태그:]` : **testVPC-rds-subnet-2c** 
    - `[VPC:]` : **testVPC** 
    - `[가용 영역:*]` : **ap-northeast-2c** 
    - `[IPv4 CIDR:*]` : **10.0.5.0/24**
- 1.1.4 좌측 상단에 위치한 `[인터넷 게이트웨이]`를 클릭 후 `[인터넷 게이트웨이 생성]`을 클릭하고 다음 정보를 입력
    - `[이름 태그:]` : **testVPC-igw** 
- 1.1.5 생성된 **testVPC-igw** 를 선택 후 우측의 `[작업]`을 클릭하고 `[VPC에 연결]`을 클릭
    - `[사용 가능한 VPC:]` : **testVPC** 
    
<br>

### Task 1.2 : 라우팅 테이블 구성 및 서브넷 
- 1.2.1 좌측 상단에 위치한 `[라우팅 테이블]`를 클릭 후 `[라우팅 테이블 생성]`을 클릭하고 다음 정보를 입력 (총 3개의 라우팅 테이블 생성)
    - `[이름 태그]` : **testVPC-public-rtb**
    - `[VPC]` : **testVPC**
    - `[이름 태그]` : **testVPC-private-rtb**
    - `[VPC]` : **testVPC**
    - `[이름 태그]` : **testVPC-rds-rtb**
    - `[VPC]` : **testVPC**
- 1.2.2 생성된 **testVPC-public-rtb** 를 선택하여, 아래의 `[라우팅]` 클릭 후, `[라우팅 편집]` 클릭하고 아래의 라우팅을 추가
    - `[대상]` **0.0.0.0/0**의 `[대상]` Internet Gateway에서 **testVPC-igw** 를 선택하고 `[라우팅 저장]` 클릭
- 1.2.3 생성된 **testVPC-public-rtb** 를 선택하여, 아래의 `[서브넷 연결]` 클릭 후, `[서브넷 연결 편집]` 클릭하고 아래의 서브넷을 추가
    - `[IPv4 CIDR]` : **10.0.0.0/24**
    - `[IPv4 CIDR]` : **10.0.1.0/24**
- 1.2.5 생성된 **testVPC-private-rtb** 를 선택하여, 아래의 `[서브넷 연결]` 클릭 후, `[서브넷 연결 편집]` 클릭하고 아래의 서브넷을 추가
    - `[IPv4 CIDR]` : **10.0.2.0/24**
    - `[IPv4 CIDR]` : **10.0.3.0/24**
- 1.2.7 생성된 **testVPC-rds-rtb** 를 선택하여, 아래의 `[서브넷 연결]` 클릭 후, `[서브넷 연결 편집]` 클릭하고 아래의 서브넷을 추가
    - `[IPv4 CIDR]` : **10.0.4.0/24**
    - `[IPv4 CIDR]` : **10.0.5.0/24**
    
<br>

### Task 1.3 : 보안 그룹 생성
- 1.3.1 좌측 중간에 위치한 `[보안 그룹]`을 클릭 후 `[보안 그룹 생성]`을 클릭하고 다음 정보를 입력 (로드밸런서 보안 그룹 생성)
    - `[보안 그룹 이름]` : **ext ELB**
    - `[설명]` : **ext ELB**
    - `[VPC]` : **testVPC**
- 1.3.2 인바운드 규칙 추가
- 1.3.3 `[유형]`에서 스크롤을 내려 `HTTP`를 클릭
- 1.3.4 `[소스]`에서 스크롤을 내려 `위치 무관`을 클릭
- 1.3.5 좌측 중간에 위치한 `[보안 그룹]`을 클릭 후 `[보안 그룹 생성]`을 클릭하고 다음 정보를 입력 (Bastion 서버 보안 그룹 생성)
    - `[보안 그룹 이름]` : **Bastion Host**
    - `[설명]` :  **Bastion Host**
    - `[VPC]` : **testVPC**
- 1.3.6 인바운드 규칙 추가
- 1.3.7 `[유형]`에서 스크롤을 내려 `SSH`를 클릭
- 1.3.8 `[소스]`에서 스크롤을 내려 `내 IP`를 클릭
- 1.3.9 좌측 중간에 위치한 `[보안 그룹]`을 클릭 후 `[보안 그룹 생성]`을 클릭하고 다음 정보를 입력 (WEB 서버 보안 그룹 생성)
    - `[보안 그룹 이름]` : **WEB**
    - `[설명]` :  **WEB**
    - `[VPC]` : **testVPC**
- 1.3.10 인바운드 규칙 추가
- 1.3.11 `[유형]`에서 스크롤을 내려 `HTTP`를 클릭
- 1.3.12 `[소스]`에서 스크롤을 내려 `위치 무관`을 클릭
- 1.3.13 `[유형]`에서 스크롤을 내려 `SSH`를 클릭
- 1.3.14 `[소스]`에서 스크롤을 내려 `위치 무관`을 클릭
- 1.3.15 좌측 중간에 위치한 `[보안 그룹]`을 클릭 후 `[보안 그룹 생성]`을 클릭하고 다음 정보를 입력 (RDS 서버 보안 그룹 생성)
    - `[보안 그룹 이름]` : **RDS**
    - `[VPC]` : **testVPC**
- 1.3.16 인바운드 규칙 추가
- 1.3.17 `[유형]`에서 스크롤을 내려 `MYSQL/Aurora`를 클릭
- 1.3.18 `[소스]`옆 검색에서 보안 그룹 중 **WEB** 을 선택

<br>

실습 1은 여기까지 입니다.

<br>

* * *

## 실습2. EC2, RDS 생성 및 접속

* * *

### 2-1. 개요
- 이 실습은 이전 실습을 기반으로 합니다.
- `[Amazon Relational Database Service(RDS)` DB 인스턴스를 시작하는 과정을 살펴봅니다.
- 관계형 데이터베이스 관리 시스템(RDBMS) 요구사항에 맞게 Amazon RDS를 사용하도록 앞에서 생성한 웹 서버를 구성합니다.
- 본 실습은 AWS 관리형 데이터베이스 인스턴스를 활용하여 관계형 데이터베이스 요구 사항을 해결하는 개념을 강화하도록 설계되었습니다.

<br>

### 2-2. 목표
본 실습을 마친 후 다음을 할 수 있게 됩니다.
- 웹 서버를 생성하고 브라우저를 통해 접속
- Bastion 서버를 생성하고 SSH 접
- 고가용성을 갖춘 Amazon RDS DB 인스턴스를 생성
- AWS 내에서 Amazon RDS DB 인스턴스로 접속

![image](/img/2.png)

<br>

### 2-3. 기간
본 실습에는 약 40분 정도가 소요

<br>

### Task 1 : Web Server, Bastion Server 시작

- 개요
    - VPC를 생성한 후, 생성한 VPC에서 EC2 인스턴스를 시작합니다.

<br>

### Task 1.1 : 웹 서버 인스턴스 생성 및 접속
- 1.1.1 `[서비스]` 메뉴에서 `[EC2]`를 클릭
- 1.1.2 화면 좌측 중앙에 `[인스턴스 시작]`을 클릭
- 1.1.3 `[Step1 : AMI 선택]` 페이지에서 **Amazon Linux 2 AMI (HVM), SSD Volume Type** 옆에 `[선택]`을 클릭
- 1.1.4 `[Step2 : 인스턴스 유형 선택]` 페이지에서 **t2.micro**가 선택되었는지 확인
    - `[다음 : 인스턴스 세부 정보 구성]`을 클릭
- 1.1.5 `[Step3 : 인스턴스 구성]` 페이지에서 다음 정보를 입력, 나머지 값은 모두 기본값 적용
    - `[네트워크]` : **testVPC**
    - `[서브넷]` : **testVPC-public-subnet-2c**
    - `[퍼블릭 IP 자동 할당]` : **활성화**로 변경
    - `[종료 방식]` : **종료**로 변경
- 1.1.6 제일 하단으로 스크롤하여 `[고급 세부 정보]` 섹션을 확장
- 1.1.7 명령 참조 파일에서 다음 사용자 데이터를 복사하여 `[사용자 데이터]` 상자에 붙여 넣고, `[텍스트로]`가 선택되었는지 확인

```sql
#!/bin/bash 
yum -y update
yum -y install httpd php mysql php-mysql
service httpd start
chkconfig httpd on
```
- 1.1.8 `[다음: 스토리지 추가]`를 클릭
- 1.1.9 `[다음: 태그 추가]`를 클릭
- 1.1.10 `[Step 5 : 태그추가]` 페이지에서 다음 정보를 입력
    - `[태그 추가]` 클릭
    - `[키]` : **Name**
    - `[값]` : **WebServer**
- 1.1.11 `[다음: 보안 그룹 구성]`을 클릭
- 1.1.12 `[Step 6 : 보안 그룹 구성]` 페이지에서 **기존 보안 그룹 선택**을 클릭
    - 이전에 생성한 보안 그룹을 선택 (**Web**)
- 1.1.13 `[검토 및 시작]`을 클릭
- 1.1.14 인스턴스 정보를 확인한 후 `[시작하기]`를 클릭
- 1.1.15 `[새 키 페어 생성]`을 클릭
    - 키 페어 이름은 **AWSLABS** 라고 지정 후, 키 페어 다운로드
    - `[인스턴스 시작]`을 클릭
- 1.1.16 생성이 완료되면 해당 인스턴스(**WebServer**)를 클릭하고 **퍼블릭 DNS(IPv4)** 값을 복사
- 1.1.17 새 웹 브라우저 창이나 탭에 **퍼블릭 DNS(IPv4)** 값을 붙여 넣고 `[Enter]` 클릭

<br>

### Task 1.2 : Bastion 서버 인스턴스 생성 및 접속
- 1.2.1 `[서비스]` 메뉴에서 `[EC2]`를 클릭
- 1.2.2 화면 좌측 중앙에 `[인스턴스 시작]`을 클릭
- 1.2.3 `[Step1 : AMI 선택]` 페이지에서 **Amazon Linux 2 AMI (HVM), SSD Volume Type** 옆에 `[선택]`을 클릭
- 1.2.4 `[Step2 : 인스턴스 유형 선택]` 페이지에서 **t2.micro**가 선택되었는지 확인
    - `[다음 : 인스턴스 세부 정보 구성]`을 클릭
- 1.2.5 `[Step3 : 인스턴스 구성]` 페이지에서 다음 정보를 입력, 나머지 값은 모두 기본값 적용
    - `[네트워크]` : **testVPC**
    - `[서브넷]` : **testVPC-public-subnet-2a**
    - `[퍼블릭 IP 자동 할당]` : **활성화**로 변경
    - `[종료 방식]` : **종료**로 변경
- 1.2.8 `[다음: 스토리지 추가]`를 클릭
- 1.2.9 `[다음: 태그 추가]`를 클릭
- 1.2.10 `[Step 5 : 태그추가]` 페이지에서 다음 정보를 입력
    - `[태그 추가]` 클릭
    - `[키]` : **Name**
    - `[값]` : **Bastion**
- 1.2.11 `[다음: 보안 그룹 구성]`을 클릭
- 1.2.12 `[Step 6 : 보안 그룹 구성]` 페이지에서 **기존 보안 그룹 선택**을 클릭
    - 이전에 생성한 보안 그룹을 선택 (**Bastion Host**)
- 1.2.13 `[검토 및 시작]`을 클릭
- 1.2.14 인스턴스 정보를 확인한 후 `[시작하기]`를 클릭
- 1.2.15 `[기존 키 페어 선택]`(**AWSLABS**)을 클릭하고, 아래 승인 확인 후 `[인스턴스 시작]`을 클릭

### Window

<br>

- 윈도우는 Putty를 통해 접속을 합니다.(https://www.putty.org/)
- 먼저 Puttygen을 실행 시켜 `Conversions` -> `import` -> `pem파일 선택` -> `save private key` 과정을 진행하여 .ppk 파일을 저장합니다.

<br>

![image](/img/4.png) 

<br>

- 변경한 .ppk 파일을 가지고 `SSH` -> `Auth` 항목에 저장합니다.

<br>

![image](/img/5.png)

<br>

- Bastion 서버의 public IP와 Port 22를 입력하고 `Open`을 클릭합니다.

<br>

![image](/img/6.png)

<br>

- 초기 계정명인 `ec2-user`를 입력하여 접속합니다.

<br>

![image](/img/7.png)

<br>

### Mac

- mac은 ssh 명령어를 통해서 접속을 합니다.

```sql
sudo ssh -i "AWSLABS.pem" ec2-user@(public IP)
```

<br>

### Task 2 : Amazon RDS DB 인스턴스 시작

- 개요
    - 이 작업에서는 MySQL이 지원되는 Amazon RDS DB 인스턴스를 시작합니다.
    
<br>

### Task 2.1 : DB 서브넷 그룹 생성 
- 2.1.1 `[서비스]` 메뉴에서 `[RDS]`를 클릭
- 2.1.2 RDS 탐색 창에서 `[서브넷 그룹]`을 클릭
- 2.1.3 `[DB 서브넷그룹 생성]`을 클릭
- 2.1.4 `[DB 서브넷그룹 생성]` 페이지에서 다음 세부 정보를 입력
    - `[이름]` : **DBsubnetgroup**
    - `[설명]` : **DBsubnetgroup**
    - `[VPC ID]` : **testVPC** 클릭
- 2.1.5 `[가용 영역]`의 경우, `[Private_Subnet5]`용으로 선택한 가용 영역을 클릭 **(ap-northeast-2a)**
- 2.1.6 `[서브넷]`의 경우, `10.0.4.0/24`를 클릭한 다음 `[서브넷 추가]`를 클릭
- 2.1.7 `[가용 영역]`을 추가로 `[Private_Subnet6]`용으로 선택한 가용 영역을 클릭 **(ap-northeast-2c)**
- 2.1.8 `[서브넷]`의 경우, `10.0.5.0/24`를 클릭한 다음 `[서브넷 추가]`를 클릭
- 2.1.9 `[생성]`을 클릭
- 2.1.10 새 서브넷 그룹이 보이지 않으면, 콘솔의 오른쪽 위 모서리에 있는 새로고침 아이콘을 클릭

<br>

### Task 2.2 : RDS DB 인스턴스 생성
- 2.2.1	`[서비스]` 메뉴에서 `[RDS]`를 클릭
- 2.2.2	Dashboard에서 `[데이터베이스 생성]` 버튼을 클릭
- 2.2.3	엔진 옵션으로`[MySQL]`을 클릭
- 2.2.4	버전(**MySQL 5.7.22**) 및 템플릿(**개발/테스트)**를 기본 값으로 지정 
- 2.2.5	`[설정]`에서 다음 세부 정보를 입력
    - `[DB 인스턴스 식별자]` : **testDB**
    - `[Master username]` : **admin**
    - `[마스터 암호]` : **gsneotek**
    - `[암호 확인]` : **gsneotek**
- 2.2.6	`[DB 인스턴스크기]`에서 버스터블 클래스 선택 **(db.t2.micro)**, `[Storage]`는 기본값 적용 
- 2.2.7	`[가용성 및 내구성]`에서 **대기 인스턴스 생성(생산 사용량에 권장)** 을 선택
- 2.2.8	`[연결]`에서 다음 정보를 입력하고 나머지 값은 모두 기본값 적용
    - `[Virtual Private Cloud(VPC)]` : **testVPC**
    - `[추가 연결 구성]` 확장
    - `[서브넷 그룹]` : **RDS**
    - `[퍼블릭 액세스 가능]` : **아니요** 클릭
    - `[VPC 보안 그룹]` : **RDS** 클릭 추가, default 보안 그룹 **X** 클릭 삭제
- 2.2.8	`[추가 구성]`에서 다음 정보를 입력하고 나머지 값은 모두 기본값 적용
    - `[초기 데이터베이스 이름]` : **sampledb** 입력
    - `[자동 백업 활성화]` : **Check** 해제
- 2.2.9	`[데이터베이스 생성]`를 클릭
- 2.2.10 `[labdbinstance]`를 선택
    - `[정보]`가 **사용 가능**으로 변할 때까지 대기 (최대 10분 정도 소요)
- 2.2.11 연결&보안 아래에 있는 `[엔드포인트]`를 복사하여 저장
    - `[엔드포인트]`는 `"labdbinstance.cdl8ki3b1gku.ap-northeast-2.rds.amazonaws.com"`과 같은 형태

<br>

### Task 3 : 데이터베이스와 상호 작용

- 개요
    - 이 작업에서는 이전 실습에서 생성한 웹 서버로 데이터베이스에 접속을 합니다.

<br>

### Task 3.1 : 데이터베이스에 액세스
웹 서버로 접속하여 생성한 데이터베이스로 접속을 합니다.
- 3.1.1 위에서 Bastion 서버에 접속한 것 같이, WebServer도 동일하게 접속을 합니다.
- 3.1.2 아래의 명령어를 참고하여 MySQL 클라이언트를 설치하고 데이터베이스에 접속을 합니다.

```sql
sudo yum install mysql           # MySQL 클라이언트 설치
mysql -uadmin -h [DB엔드포인트] -p    # MySQL 접속 명령어
```

<br>

실습 2는 여기까지 입니다.

<br>

* * *

## 실습3. 로드밸런서, 오토스케일링 구성

* * *

### 3-1. 개요
- 본 실습은 이전 실습을 기반으로 합니다.
- `[Elastic Load Balancing(ELB)]`과 `[Auto Scaling]` 서비스를 사용합니다.
- 인프라를 로드 밸런싱하고 자동 조정하는 과정을 살펴봅니다.

<br>

### 3-2. 목표 
본 실습을 마친 후 다음을 할 수 있게 됩니다.
- 실행중인 EC2 인스턴스에서 Amazon 머신 이미지(AMI)를 생성
- AWS ELB 로드 밸런서 추가
- EC2 Auto Scaling 시작 구성 생성
- EC2 Auto Scaling 그룹 생성

![image](/img/3.png)

### 3-3. 기간
본 실습에는 약 40분 정도가 소요

<br>

### Task 1 : Auto Scaling

- 개요
    - 이 작업에서는 인프라를 생성하고 자동 조정합니다.

<br>

### Task 1.1 : Auto Scaling 용 AMI 생성
- 1.1.1	`[AWS Management Console]`의 `[서비스]` 메뉴에서 `[EC2]`를 클릭
- 1.1.2	탐색 창에서 `[인스턴스]`를 클릭
- 1.1.3	`[WebServer]`의 `[상태 검사]탭` 2/2 checks passed로 표시되어 있는지 확인
    - 아닌 경우, 변경될 때까지 기다렸다가 다음 단계로 진행
    - 오른쪽 위에 있는 새로고침 버튼을 사용하여 업데이트를 확인
- 1.1.4	`[WebServer]`을 오른쪽 마우스 클릭, `[이미지]`를 클릭, `[이미지 생성]`를 클릭
- 1.1.5	다음 정보를 입력, 나머지 값은 그대로 기본값 적용
    - `[이미지 이름]` : **WebServer AMI**
    - `[이미지 설명]` : **WebServer AMI**
    - `[재부팅 안 함]` : **Check v**
- 1.1.6	`[이미지 생성]`를 클릭

<br>

### Task 1.2 : 로드 밸런서 추가
- 1.2.1	EC2 탐색 창에서 `[로드밸런서]`를 클릭
- 1.2.2 `[로드 밸런서 생성]`를 클릭, 다음 화면에서 Applicaion Load Balancer를 선택
- 1.2.3 다음 정보를 입력, 나머지 값은 그대로 기본값 적용
    - `[로드 밸런서 이름]` : **testELB**
    - `[LB 생성할 VPC]` : **testVPC** 선택
    - `[리스너 구성]` : **HTTP 80포트** 기본 값 유지
    - `[서브넷 선택]` : [+]를 클릭하여 **testVPC-public-subnet-2a** 및 **testVPC-public-subnet-2c** 를 선택
- 1.2.4 `[다음: 보안 그룹 구성]`를 클릭
    - **default** 보안 그룹을 Check 해제
    - 이름에 **ext ELB**이 포함된 보안 그룹을 선택
- 1.2.5 `[다음: 라우팅 구성]`을 클릭
- 1.2.6 다음 정보를 입력, 나머지 값은 그대로 기본값 적용
    - `[이름]` : **testTarget**
- 1.2.7 `[다음: 대상 등록]`을 클릭하고, 오토스케일링과 연동 할 에정이므로 스킵
- 1.2.8 `[다음: 검토]`를 클릭하고 로드밸런서를 생성

<br>

### Task 1.3 : 시작 구성 및 Auto Scaling 그룹 생성
- 1.3.1	EC2 탐색 창에서 AUTO SCALING `[시작 구성]`를 클릭
- 1.3.2	`[시작 구성 생성]`을 클릭
- 1.3.3	탐색 창에서 `[내 AMI]`를 클릭
- 1.3.4	앞에서 생성한 `[WebServer AMI]`에서 `[선택]`를 클릭
- 1.3.5	`[t2.micro]` 클릭하고 `[다음: 세부 정보 구성]`을 클릭
- 1.3.6	다음 정보를 입력하고, 나머지 값은 그대로 기본값 적용
    - `[이름]` : **testConfig**
    - `[모니터링]` : **CloudWatch 세부 모니터링 활성화** 선택
- 1.3.7	`[다음: 스토리지 추가]`를 클릭
- 1.3.8	`[다음: 보안 그룹 구성]`을 클릭
- 1.3.9	`[기존 보안 그룹 선택]`을 클릭, 이름이 `[Web]`인 보안 그룹을 선택
- 1.3.10 `[검토]`를 클릭
- 1.3.11 시작 구성의 세부 정보를 확인, `[시작 구성 생성]`을 클릭
- 1.3.12 `[기존 키 페어 선택`]을 클릭, **AWSLABS**를 선택
    - 승인 확인란을 선택한 후 `[시작 구성 생성`]을 클릭 후 닫기 버튼을 클릭
- 1.3.13 **testConfig**을 Check 후 `[Auto Scaling 그룹 생성]`을 클릭
    -  다음 정보를 입력, 나머지 값은 그대로 기본값 적용
    - `[그룹 이름]` : **testASgroup**
    - `[그룹 크기]` : **2**개의 인스턴스로 시작
    - `[네트워크]` : **testVPC** 선택
    - `[서브넷]` : **testVPC-private-subnet-2a(10.0.2.0/24)** 및 **testVPC-private-subnet-2c((10.0.3.0/24)** 선택
- 1.3.14 아래로 스크롤하여 `[고급 세부 정보]`를 확장 후 `[하나 이상의 로드 밸런서에서 트래픽 수신]`을 선택
- 1.3.15 `[대상 그룹]` 텍스트 상자를 클릭, `[testTarget]`를 클릭
- 1.3.16 다음 정보를 입력, 나머지 값은 그대로 기본값 적용
    - `[상태 검사 유형]` : **ELB** 선택
    - `[모니터링]` : **CloudWatch 세부 모니터링 활성화** 선택
- 1.3.17 `[다음: 조정 정책 구성]`을 클릭
- 1.3.18 `[조정 정책을 사용하여 이 그룹의 용량 조정]`을 선택
- 1.3.19 **2**개와 **4**개의 인스턴스 범위 내에서 조정할 수 있도록 `[조정 범위]` 텍스트 상자를 수정
    - **단계 또는 단순 조정 정책을 사용하여 Auto Scaling 그룹 조정**을 클릭 
- 1.3.20 `[그룹 크기 증가]`에서 `[정책 실행 요건]`에 대해 `[새 경보 추가]`을 클릭
- 1.3.21 `[알림 보낼 대상:]`이 선택되어 있는지 확인한 후, `[주제 생성]`을 클릭
    - 이메일 알람을 생성하는 것은 선택 사항이며, 이 단계를 생략 가능
    - 이메일 알림을 받지 않으려면, `[알림 보낼 대상:]`을 선택 해제
- 1.3.22 다음 정보를 입력하고, 나머지 값은 그대로 기본값 적용 
    - `[다음경우 항상]`: **Average** / **CPU 사용률**
    - `[이(가)]` : >= **70%**
    - `[최소 다음의 경우]` : **1** 연속 기간 **1분**
    - `[경보 이름]` : **HighCPUUtilization**
- 1.3.23 `[경보 생성]`을 클릭
- 1.3.24 `[그룹 크기 증가]`의 나머지 부분에 다음 정보를 입력
    - `[작업 수행]` : 추가 선택. **1**입력, 인스턴스 선택. 조건 **70**입력
    - `[인스턴스 필요 시간]` : 각 단계 후 워밍업 시간에 **60**초
- 1.3.25 `[그룹 크기 감소]`에서 `[정책 실행 요건]`에 대해 `[새 경보 추가]`을 클릭
- 1.3.26 `[알림 보낼 대상:]`이 선택되어 있는지 확인한 후, `[주제 생성]`을 클릭
    - 이메일 알람을 생성하는 것은 선택 사항이며, 이 단계를 생략 가능
    - 이메일 알림을 받지 않으려면, `[알림 보낼 대상:]`을 선택 해제
- 1.3.27 다음 정보를 입력하고, 나머지 값은 그대로 기본값 적용
    - `[다음경우 항상]`: **Average** / **CPU 사용률**
    - `[이(가)]` : <= **10%**
    - `[최소 다음의 경우]` : **1** 연속 기간 **1분**
    - `[경보 이름]` : **LowCPUUtilization**
- 1.3.28 `[경보 생성]`을 클릭
- 1.3.29 `[그룹 크기 감소]`의 나머지 부분에 다음 정보를 입력
    - `[작업 수행]` : 제거 선택. **1**입력, 인스턴스 선택. 조건 **10**입력
- 1.3.30 `[다음: 알림구성]`을 클릭
- 1.3.31 `[다음: 태그구성]]`을 클릭
- 1.3.32 다음 정보를 입력하고, 나머지 값은 그대로 기본값 적용
    - `[키]` : **Name**
    - `[값]` : **ASWEBServer**
- 1.3.33 `[검토]`를 클릭
- 1.3.34 Auto Scaling 그룹의 세부 정보를 확인한 다음, `[Auto Scaling 그룹 생성]`을 클릭
- 1.3.35 Auto Scaling 그룹이 생성되면 `[닫기]`를 클릭
- 1.3.36 퍼블릭 액세스 가능한 최초 웹 서버는 더 이상 필요없기에 아래와 같이 삭제를 진행
    - `[서비스]` 메뉴에서 `[EC2]`를 클릭
    - EC2 탐색 창에서 `[인스턴스]`를 클릭
    - `[WebServer]`을 마우스 오른쪽 버튼으로 클릭
    - `[인스턴스 상태]`에서 `[종료]`를 클릭

<br>

### Task 1.4 : Test
- 1.4.1 로드밸런서로 접속하여 부하 분산이 되는지 확인
    - `[서비스]` 메뉴에서 `[EC2]`를 클릭
    - `[로드밸런서]`에서 `[대상 그룹]`을 클릭
    - **testTarget**을 클릭하여 아래의 `[대상]`을 클릭하여 현재 인스턴스의 상태를 확인
    - `[로드밸런서]`에서 **testELB**을 클릭 후, `[DNS 이름]`의 주소로 접속하여 웹페이지가 뜨는지 확인
- 1.4.2 오토스케일링 동작이 잘 되는지 확인을 위해 오토스케일링으로 생성된 서버를 삭제
    - `[서비스]` 메뉴에서 `[EC2]`를 클릭
    - EC2 탐색 창에서 `[인스턴스]`를 클릭
    - `[ASWebServer]`을 마우스 오른쪽 버튼으로 클릭
    - `[인스턴스 상태]`에서 `[종료]`를 클릭
- 1.4.3 잠시 기다렸다가 인스턴스가 자동으로 생성되는 것을 확인

<br>

실습3을 완료하셨습니다. 준비된 실습을 모두 완료하셨습니다.

<br>

## Terminate Resources

10분

1. Delete Your Amazon RDS
RDS부터 지워봅시다
- `[AWS Management Console]` ▷ `[RDS]`
    - **testDB**을 클릭하고 상단 `[작업]` ▷ `[삭제]` 클릭
        - 최종 스냅샷 생성 여부 **Check** 해제
        - 인스턴스 삭제 시 시스템 스냅샷 및 특정 시점으로 복구를 포함한 자동화된 백업을 더 이상 사용할 수 없다는 항목에 **Check**
        - 삭제를 확인하려면 아래 필드에 **delete me**라는 문구를 입력 후 `[삭제]` 클릭

<br>

2. Delete Your EC2 (AutoScaling Instance)
AutoScaling으로 생성된 EC2를 지워봅시다
- `[AWS Management Console]` ▷ `[EC2]` ▷ `[Auto Scaling]` 그룹 클릭
    - **testASgroup**을 클릭하고 상단 `[작업]` ▷ `[삭제]` 클릭

- `[AWS Management Console]` ▷ `[EC2]` ▷ `[시작 구성]` 클릭
    - **testConfig**을 클릭하고 상단 `[작업]` ▷ `[시작 구성 삭제]` 클릭

- `[AWS Management Console]` ▷ `[EC2]` ▷ `[AMI]` 클릭
    - **WebServer AMI**의 AMI ID를 기록 (예: ami-0ea06fc7ff2beb3ae)
    - **WebServer AMI**를 클릭하고 상단 `[작업]` ▷ `[등록 취소]`▷ `[계속]` 클릭

- `[AWS Management Console]` ▷ `[EC2]` ▷ `[스냅샷]` 클릭
    - **WebServer AMI**의 AMI ID로 검색
    - 검색된 스냅샷을 클릭하고 상단 `[작업]` ▷ `[삭제]` ▷ `[예, 삭제]`를 클릭

<br>

3. Delete Your EC2 (NAT Instance)
VPC 마법사로 생성된 EC2 Instance를 지워 봅시다
- `[AWS Management Console]` ▷ `[VPC]` 클릭
    - **NAT Gateway**을 클릭하고 상단 `[작업]` ▷ `[NAT 게이트웨이 삭제]` 클릭

<br>

4. Delete Your ELB
ELB도 지워봅시다
- `[AWS Management Console]` ▷ `[EC2]` ▷ `[로드 밸런서]` 클릭
    - **Lab3ELB**을 클릭하고 상단 [작업] ▷ `[삭제]` 클릭 ▷ `[예, 삭제]`


수고하셨습니다.



