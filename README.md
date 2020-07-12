# AWS 웹 3티어 구축 실습

## 목차.
- 실습1. AWS VPC 구축 및 서브넷 구성 (35분)
- 실습2. WebServer, Bastion 서버 생성 및 접속 (50분)
- 실습3. 오토스케일링 구성, RDS 생성 및 접속 (50분)
- Terminate Resources (10분)

* * *

## 실습1. AWS VPC 구축 및 웹 서버 시작 

* * *

### 1-1. 개요 
- 본 실습 섹션에서는 `[Amazon Virtual Private Cloud(VPC)]`를 사용하여 자체 VPC를 생성합니다.
- AWS VPC에 구성 요소를 추가하여 사용자 정의된 네트워크를 구성합니다.
- AWS EC2 인스턴스에 대한 보안 그룹을 생성합니다.
- 웹 서버를 실행하고 이를 VPC에서 시작하도록 EC2 인스턴스를 구성 및 사용자 정의합니다.

![image](/images/Day1-Description-VPC.png)

`[Amazon Virtual Private Cloud(VPC)]`는 Amazon Web 서비스(AWS) 리소스를 고객이 정의한 가상 네트워크에서 실행할 수 있는 서비스입니다. 이 가상 네트워크는 고객의 자체 데이터 센터에서 운영하는 기존 네트워크와 매우 유사하며 AWS의 확장 가능한 인프라를 사용한다는 이점이 있습니다. 

`[인터넷게이트웨이(IGW)]`는 AWS VPC에 있는 인스턴스와 인터넷 간 통신을 허용하는 VPC 구성요소입니다. 라우팅 테이블(Route Table)은 네트워크 트래픽이 향하는 방향을 결정하는 데 사용되는 경로라고 부르는 규칙 세트를 포함합니다. VPC에 있는 각 서브넷은 라우팅 테이블에 연결되어 있어야합니다. 라우팅 테이블이 서브넷에 대한 라우팅을 제어합니다.

AWS VPC를 생성하면 각 가용 영역에 하나 이상의 서브넷(Subnet)을 추가할 수 있습니다. 서브넷 트래픽이 인터넷게이트웨이(IGW)로 라우팅되는 경우 해당 서브넷을 퍼블릭 서브넷(Public Subnet)이라고 합니다. 서브넷이 인터넷게이트웨이(IGW)로 라우팅되지 않으면 해당 서브넷을 프라이빗 서브넷(Private Subnet)이라고 합니다.

<br>

### 1-2. 목표
본 실습을 마친 후 다음을 할 수 있게 됩니다.
- AWS VPC를 생성
- 서브넷(Subnet)을 생성
- 보안 그룹(Security Group)을 생성
- AWS VPC에서 EC2 인스턴스 시작

<br>

### 1-3. 사전 조건
본 실습을 위해서는 다음이 필요합니다.
- Microsoft Windows, Mac OS X 또는 Linux(Ubuntu,SuSE, Red Hat)가 실행되며 Wi-Fi가 되는 컴퓨터 사용
- Chrome, Firefox 또는 IE9 이상 버전과 같은 인터넷 브라우저 (IE9 이전 버전은 지원 안됨)

<br>

### 1-4. 기간 
본 실습에는 약 35분 정도가 소요

<br>

### Task 1 : VPC 생성 

- 개요
    - 이 섹션에서는 VPC를 생성합니다.

- 시나리오

본 실습에서는 다음과 같은 인프라를 구축하게 됩니다.

![image](/images/Day1-Task-WEB.png)

<br>

### Task 1.1 : VPC 생성 
이 작업에서는 EC2 인스턴스용 Keypair 생성과 하나의 가용 영역에서 2개의 서브넷으로 구성된 VPC를 생성합니다.
- 1.1.1 `AWS Management Console`의 `서비스` 메뉴에서 `[EC2]`를 클릭
    - `[네트워크 및 보안]` 탭 아래 `[키페어]` 항목 클릭
    - `[키 페어 생성]`을 클릭하고 키 페어 이름에 **AWSLABS** 입력 후 생성
    - **AWSLABS.pem** 파일을 안전하게 보관 (EC2 인스턴스 접속 용도로 사용)
- 1.1.2 `AWS Management Console`의 `서비스` 메뉴에서 `[VPC]`를 클릭
- 1.1.3 좌측 상단에 위치한 `[VPC 대시보드]`를 클릭 후 `[VPC 마법사 시작]`을 클릭

    ![image](/images/Day1-Create-VPC.png)

- 1.1.4 탐색 창에서 `[퍼블릭 및 프라이빗 서브넷이 있는 VPC]`를 클릭
- 1.1.5 `[선택]`을 클릭하고 다음 정보를 입력
    - `[IPv4 CIDR 블록:*]` : **10.0.0.0/16**
    - `[VPC 이름]` : **My Lab VPC**
    - `[퍼블릭 서브넷의 IPv4 CIDR:*]` : **10.0.1.0/24**
    - `[가용 영역:*]` : **ap-northeast-2a** 
    - `[퍼블릭 서브넷 이름:]` : **Public_Subnet1** 
    - `[프라이빗 서브넷의 IPv4 CIDR:*]` : **10.0.3.0/24**
    - `[가용 영역:*]` : **ap-northeast-2a** 
    - `[프라이빗 서브넷 이름:]` : **Private_Subnet3**
- 1.1.6 `[NAT 게이트웨이의 세부 정보를 지정합니다.]`에서
    - 화면 오른쪽의 **대신 NAT 인스턴스 사용**를 클릭
- 1.1.7 `[인스턴스 유형:*]`에 나열된 첫 번재 인스턴스 유형을 선택 (**예 : t2.nano**)
- 1.1.8 `[키 페어 이름]`에서 **AWSLABS** 키 페어를 선택
- 1.1.9 `[VPC 생성]`를 클릭
- 1.1.10 VPC가 생성되면, VPC가 성공적으로 생성되었다는 메시지가 표시된 페이지 확인

<br>

### Task 1.2 : 추가 서브넷 생성
본 작업에서는 다른 가용 영역에 서브넷 2개를 추가로 생성하고 기존 라우팅 테이블을 통해 서브넷을 연결합니다.
- 1.2.1 VPC 탐색 창에서 `[서브넷]`를 클릭 
- 1.2.2 `[서브넷 생성]`을 클릭
- 1.2.3 `[서브넷 생성]` 대화 상자에서 다음 세부 정보를 입력
    - `[이름 태그]` : **Public_Subnet2**
    - `[VPC]` : **My Lab VPC** 클릭
    - `[가용 영역]` : **ap-northeast-2c** 선택
    - `[IPv4 CIDR 블록*]` : **10.0.2.0/24** 입력
- 1.2.4 `[생성]` 클릭 후 `[닫기]` 클릭
- 1.2.5 `[서브넷 생성]`을 클릭
- 1.2.6 `[서브넷 생성]` 대화 상자에서 다음 세부 정보를 입력
    - `[이름 태그]` : **Private_Subnet4**
    - `[VPC]` : **My Lab VPC** 클릭
    - `[가용 영역]` : **ap-northeast-2c** 선택
    - `[IPv4 CIDR 블록*]` : **10.0.4.0/24** 입력
- 1.2.7 `[생성]` 클릭 후 `[닫기]` 클릭
- 1.2.8 `[Public_Subnet2]`를 선택하고, 모든 다른 서브넷이 선택 해제되었는지 확인
    - 아래쪽 창에 있는 `[라우팅 테이블]`을 클릭
    - `[대상]` **0.0.0.0/0**의 `[대상]`이 접두사 **IGW**가 포함되어 있는지 확인
    - 포함되어 있지 않은 경우, `[라우팅 테이블 연결 편집]`를 클릭하고 `[라우팅 테이블 ID*]` 목록에 있는 다른 라우팅 테이블을 클릭
    - `[대상]` **0.0.0.0/0**의 `[대상]`이 접두사 **IGW**가 포함되어 있다면 `[저장]` 클릭
- 1.2.9 `[Private_Subnet4]`를 선택하고, 모든 다른 서브넷이 선택 해제되었는지 확인한 다음
    - 아래쪽 창에 있는 `[라우팅 테이블]`을 클릭
    - `[대상]` **0.0.0.0/0**의 `[대상]`이 접두사 **eni**가 포함되어 있는지 확인 
    - 포함되어 있지 않은경우, `[라우팅 테이블 연결 편집]`를 클릭하고 `[라우팅 테이블 ID*]` 목록에 있는 다른 라우팅 테이블을 클릭
    - `[대상]` **0.0.0.0/0**의 `[대상]`이 접두사 **eni**를 포함되어 있다면 `[저장]`을 클릭

<br>

### Task 1.3 : VPC 보안 그룹 생성
웹 및 SSH 트래픽에 대한 액세스 권한을 부여하는 VPC 보안 그룹을 생성합니다.
- 1.3.1 EC2 탐색 창에서 `[네트워크 및 보안]`탭에서 `[보안 그룹]`를 클릭
- 1.3.2 `[보안 그룹 생성]`을 클릭
- 1.3.3 `[보안 그룹 생성]` 대화 상자에서 다음 정보를 입력
    - `[보안 그룹 이름]` : **WebSecurityGroup**
    - `[설명]` : **Enable HTTP access**
    - `[VPC]` : **작업 1.1에서 생성한 VPC를 클릭(My Lab VPC)**
- 1.3.4 `[보안 그룹 생성]` 클릭
- 1.3.5 EC2 탐색 창에서 `[네트워크 및 보안]`탭에서 `[보안 그룹]`를 클릭
- 1.3.6 `[WebSecurityGroup]`을 선택
- 1.3.7 `[인바운드 규칙]` 탭을 클릭
- 1.3.8 `[규칙 추가]`을 클릭
- 1.3.9 `[유형]`에서 스크롤을 내려 `HTTP`를 클릭
- 1.3.10 `[소스]` 입력란에 **0.0.0.0/0**를 입력
- 1.3.11 `[규칙 추가]`를 클릭
- 1.3.12 `[유형]`에서 스크롤을 내려 `SSH`를 클릭
- 1.3.13 `[소스]` 입력란에  **내 IP**를 클릭
- 1.3.14 `[규칙 저장]` 클릭

<br>

### Task 2 : Web Server 시작

- 개요
    - VPC를 생성한 후, 생성한 VPC에서 EC2 인스턴스를 시작합니다.
    - EC2 웹 서버의 역할을 하도록 부트스트랩합니다.

<br>

### Task 2.1 : 첫 번째 웹 서버 인스턴스 시작
본 작업에서는 VPC에서 EC2 인스턴스를 시작하는 과정을 확인합니다. 이 인스턴스는 웹 서버의 역할을 합니다.
- 2.1.1 `[서비스]` 메뉴에서 `[EC2]`를 클릭
- 2.1.2 화면 좌측 중앙에 `[인스턴스 시작]`을 클릭
- 2.1.3 `[Step1 : AMI 선택]` 페이지에서 **Amazon Linux AMI 2018.03.0 (HVM), SSD Volume Type** 옆에 `[선택]`을 클릭
- 2.1.4 `[Step2 : 인스턴스 유형 선택]` 페이지에서 **t2.micro**가 선택되었는지 확인
    - `[다음 : 인스턴스 세부 정보 구성]`을 클릭
- 2.1.5 `[Step3 : 인스턴스 구성]` 페이지에서 다음 정보를 입력, 나머지 값은 모두 기본값 적용
    - `[네트워크]` : 작업 1.1에서 생성한 VPC를 클릭(**My Lab VPC**)
    - `[서브넷]` : 작업 1.2에서 생성한 `Public_Subnet2 | ap-northeast-2`를 클릭
    - `[퍼블릭 IP 자동 할당]` : **활성화**로 변경
    - `[종료 방식]` : **종료**로 변경
- 2.1.6 제일 하단으로 스크롤하여 `[고급 세부 정보]` 섹션을 확장
- 2.1.7 명령 참조 파일에서 다음 사용자 데이터를 복사하여 `[사용자 데이터]` 상자에 붙여 넣고, `[텍스트로]`가 선택되었는지 확인

```sql
#!/bin/bash -ex
yum -y update
yum -y install httpd php mysql php-mysql
chkconfig httpd on
/etc/init.d/httpd start
if [ ! -f /var/www/html/lab2-app.tar.gz ]; then
cd /var/www/html
wget https://us-west-2-aws-staging.s3.amazonaws.com/awsu-ilt/AWS-100-ESS/v4.0/lab-2-configure-website-datastore/scripts/lab2-app.tar.gz
tar xvfz lab2-app.tar.gz
chown apache:root /var/www/html/lab2-app/rds.conf.php
fi
```
- 2.1.8 `[다음: 스토리지 추가]`를 클릭
- 2.1.9 `[다음: 태그 추가]`를 클릭
- 2.1.10 `[Step 5 : 태그추가]` 페이지에서 다음 정보를 입력
    - `[태그 추가]` 클릭
    - `[키]` : **Name**
    - `[값]` : **Web Server 1**
- 2.1.11 `[다음: 보안 그룹 구성]`을 클릭
- 2.1.12 `[Step 6 : 보안 그룹 구성]` 페이지에서 **기존 보안 그룹 선택**을 클릭
    - 작업 1.3에서 생성한 보안 그룹을 선택 (**WebSecurityGroup**)
- 2.1.13 `[검토 및 시작]`을 클릭
- 2.1.14 인스턴스 정보를 확인한 후 `[시작하기]`를 클릭
- 2.1.15 `[Choose an existing key pair]`를 클릭
    - **AWSLABS** 키 페어를 클릭하고, 아래 승인 확인란 체크
    - `[인스턴스 시작]`을 클릭
- 2.1.16 아래로 스크롤하여 `[인스턴스 보기]`를 클릭
- 2.1.17 2개의 인스턴스(**Web Server1**과 VPC 마법사에서 시작한 **NAT 인스턴스**) 확인
    - t2.nano 타입의 EC2 인스턴스 선택 후 `[Name]` 탭 연필 표시 클릭 후  **NAT Instance**로 변경
- 2.1.18 `[Web Server 1]`의 `[Status Checks]` 열에 **2/2 검사 통과**가 표시될 때까지 대기
    - 3~5분 정도 소요. 오른쪽 위에 있는 새로 고침 아이콘을 사용하여 업데이트를 확인
- 2.1.19 `[Web Server 1]`을 선택하고 **퍼블릭 DNS(IPv4)** 값을 복사
    - 다음과 같은 주소(예시) : **ec2-54-180-82-138.ap-northeast-2.compute.amazonaws.com**
- 2.1.20 새 웹 브라우저 창이나 탭에 **퍼블릭 DNS(IPv4)** 값을 붙여 넣고 `[Enter]` 클릭
    -  아래와 같은 `[Amazon Linux AMI Test Page]`를 확인


![image](/images/Day1-Create-EC2.png)

<br>

실습1을 완료하셨습니다!

축하드립니다. 성공적으로 VPC를 생성하고 생성한 VPC에서 EC2 인스턴스를 시작했습니다. ^^

<br>

* * *

## 실습2. AWS DB 서버 구축 및 웹서버를 사용하여 DB와 상호 작용

* * *

### 2-1. 개요
- 이 실습은 이전 실습을 기반으로 합니다.
- `[Amazon Relational Database Service(RDS)` DB 인스턴스를 시작하는 과정을 살펴봅니다.
- 관계형 데이터베이스 관리 시스템(RDBMS) 요구사항에 맞게 Amazon RDS를 사용하도록 앞에서 생성한 웹 서버를 구성합니다.
- 본 실습은 AWS 관리형 데이터베이스 인스턴스를 활용하여 관계형 데이터베이스 요구 사항을 해결하는 개념을 강화하도록 설계되었습니다.

![image](/images/Day1-Description-RDS.png)

`[Amazon Relational Database Service(RDS)]`를 사용하면 클라우드에서 관계형 데이터베이스를 더욱 간편하게 설정, 관리 및 확장할 수 있습니다. 시간 소모적인 데이터베이스 관리 작업을 관리하는 한편, 효율적인 비용으로 크기 조정이 가능한 용량을 제공하므로 고객은 애플리케이션과 비즈니스에 좀 더 집중할 수 있습니다. Amazon RDS를 사용하면 6개의 익숙한 데이터베이스 엔진 Amazon Aurora, Oracle, Microsoft SQL Server, PostgresSQL, MySQL 및 MariaDB 중에서 원하는 것을 선택할 수 있습니다.

Amazon RDS 다중 AZ 배포는 데이터베이스(DB) 인스턴스의 가용성 및 내구성을 높여주므로 프로덕션 데이터베이스 워크로드에 적합합니다. 다중 AZ DB 인스턴스를 프로비저닝하면, Amazon RDS는 기본 DB 인스턴스를 자동으로 생성하고. 다른 가용 영역(AZ)에 있는 대기 인스턴스에 데이터를 동기식으로 복제합니다.

<br>

### 2-2. 목표
본 실습을 마친 후 다음을 할 수 있게 됩니다.
- 고가용성을 갖춘 Amazon RDS DB 인스턴스를 시작합니다.
- 웹 서버로부터의 연결을 허용하도록 DB 인스턴스를 구성합니다.
- 웹 애플리케이션을 열고 데이터베이스와 상호 작용합니다.

<br>

### 2-3. 기간
본 실습에는 약 35분 정도가 소요

<br>

### Task 1 : Amazon RDS DB 인스턴스 시작

- 개요
    - 이 작업에서는 MySQL이 지원되는 Amazon RDS DB 인스턴스를 시작합니다.

- 시나리오

다음 인프라에서 시작합니다.

![image](/images/Day1-Task-WEB.png)

다음 인프라를 구축합니다.

![image](/images/Day1-Task-RDS.png)

<br>

### Task 1.1 : RDS DB 인스턴스에 대한 VPC 보안 그룹 생성
이 작업에서는 웹 서버가 RDS DB 인스턴스에 액세스하도록 허용하는 VPC 보안 그룹을 생성합니다.
- 1.1.1 `[AWS Management Console의 서비스]` 메뉴에서 `[VPC]`를 클릭
- 1.1.2 VPC 탐색 창에서 `[보안]`탭에 속한 `[보안 그룹]`을 클릭
- 1.1.3 `[보안 그룹 생성]`을 클릭
- 1.1.4 `[보안 그룹 생성]` 대화 상자에서 다음 세부 정보를 입력
    - `[보안 그룹 이름]` : **DBSecurityGroup**
    - `[설명]` : **DB Instance Security Group**
    - `[VPC]` : **My LAB VPC** 클릭
- 1.1.5 `[보안 그룹 생성]`을 클릭 후 `[닫기]`
- 1.1.6 VPC 탐색 창에서 `[보안]`탭에 속한 `[보안 그룹]`을 클릭
- 1.1.7 방금 생성한 `[DBSecurityGroup]`을 선택
- 1.1.8	아래 `[인바운드 규칙]` 탭을 선택하고 `[규칙 추가]`을 클릭
- 1.1.9	다음 세부 정보로 인바운드 규칙을 생성
    - `[규칙 추가]` 클릭
    - `[유형]` : 스크롤을 내려 **MySQUAurora** 선택
    - `[소스]` : 입력 창에 **sg** 입력 후  **WebSecurityGroup** 항목을 클릭
- 1.1.10	`[규칙 저장]` 클릭 후 `[닫기]`

<br>

### Task 1.2 : Amazon RDS 인스턴스용 프라이빗 서브넷 생성
이 작업에서는 Amazon RDS 인스턴스용 프라이빗 서브넷을 2개 생성합니다.
- 1.2.1	VPC 탐색 창에서 `[서브넷]`을 클릭
- 1.2.2	`[서브넷 생성]`을 클릭
- 1.2.3	`[서브넷 생성]` 대화 상자에서 다음 세부 정보를 입력
    - `[이름 태그]` : **Private_Subnet5**
    - `[VPC]` : **My Lab VPC** 선택
    - `[가용 영역]` : 앞에서 **Public_Subnet1**용으로 적어둔 **ap-northeast-2a**를 클릭
    - `[CIDR block]` : **10.0.5.0/24**
- 1.2.4	`[생성]`후 `[닫기]`클릭
- 1.2.5	`[서브넷 생성]`을 클릭
- 1.2.6	`[서브넷 생성]` 대화 상자에서 다음 세부 정보를 입력
    - `[이름 태그]` : **Private_Subnet6**
    - `[VPC]` : **My Lab VPC** 선택
    - `[가용 영역]` : 앞에서 **Public_Subnet2**용으로 적어둔 **ap-northeast-2c**를 클릭
    - `[CIDR block]` : **10.0.6.0/24**
- 1.2.7	`[생성]`후 `[닫기]`클릭
- 1.2.8 `[Private_Subnet5]`를 선택하고, 모든 다른 서브넷이 선택 해제되었는지 확인
    - 아래쪽 창에 있는 `[라우팅 테이블]`을 클릭
    - `[대상]` **0.0.0.0/0**의 `[대상]`이 접두사 **eni**가 포함되어 있는지 확인 
    - 포함되어 있지 않은경우, `[라우팅 테이블 연결 편집]`를 클릭하고 `[라우팅 테이블 ID*]` 목록에 있는 다른 라우팅 테이블을 클릭
    - `[대상]` **0.0.0.0/0**의 `[대상]`이 접두사 **eni**를 포함되어 있다면 `[저장]`을 클릭
- 1.2.9 `[Private_Subnet6]`에서 앞에서 수행한 단계를 반복

<br>

### Task 1.3 : DB 서브넷 그룹 생성 
이 작업에서는 DB 서브넷 그룹을 생성. 각 DB 서브넷 그룹은 지정된 리전에서 두 개 이상의 가용 영역에 서브넷이 있어야 합니다.
- 1.3.1 `[서비스]` 메뉴에서 `[RDS]`를 클릭
- 1.3.2 RDS 탐색 창에서 `[서브넷 그룹]`을 클릭
- 1.3.3 `[DB 서브넷그룹 생성]`을 클릭
- 1.3.4 `[DB 서브넷그룹 생성]` 페이지에서 다음 세부 정보를 입력
    - `[이름]` : **dbsubnetgroup**
    - `[설명]` : **Lab DB Subnet Group**
    - `[VPC ID]` : **My Lab VPC** 클릭
- 1.3.5 `[가용 영역]`의 경우, `[Private_Subnet5]`용으로 선택한 가용 영역을 클릭 **(ap-northeast-2a)**
- 1.3.6 `[서브넷]`의 경우, `10.0.5.0/24`를 클릭한 다음 `[서브넷 추가]`를 클릭
- 1.3.7 `[가용 영역]`을 추가로 `[Private_Subnet6]`용으로 선택한 가용 영역을 클릭 **(ap-northeast-2c)**
- 1.3.8 `[서브넷]`의 경우, `10.0.6.0/24`를 클릭한 다음 `[서브넷 추가]`를 클릭
- 1.3.9 `[생성]`을 클릭
- 1.3.10 새 서브넷 그룹이 보이지 않으면, 콘솔의 오른쪽 위 모서리에 있는 새로고침 아이콘을 클릭

<br>

### Task 1.4 : RDS DB 인스턴스 생성
이 작업에서는 MySQL 지원 Amazon RDS DB 인스턴스를 구성 및 시작합니다.
- 1.4.1	`[서비스]` 메뉴에서 `[RDS]`를 클릭
- 1.4.2	Dashboard에서 `[데이터베이스 생성]` 버튼을 클릭
- 1.4.3	엔진 옵션으로`[MySQL]`을 클릭
- 1.4.4	버전(**MySQL 5.7.22**) 및 템플릿(**프리티어)**을 기본 값으로 지정 
- 1.4.5	`[설정]`에서 다음 세부 정보를 입력
    - `[DB 인스턴스 식별자]` : **labdbinstance**
    - `[Master username]` : **labuser**
    - `[마스터 암호]` : **labpassword**
    - `[암호 확인]` : **labpassword**
- 1.4.6	`[DB 인스턴스크기]`에서 스탠다드 클래스 선택 **(db.t2.micro)**, `[Storage]`는 기본값 적용 
- 1.4.7	`[가용성 및 내구성]`에서 다중 AZ 배포의 경우 **대기 인스턴스 생성(생산 사용량에 권장)** 을 선택해야 하지만 프리티어로 진행되기에 실습 구성에서는 생략
- 1.4.8	`[연결]`에서 다음 정보를 입력하고 나머지 값은 모두 기본값 적용
    - `[Virtual Private Cloud(VPC)]` : **My Lab VPC**
    - `[추가 연결 구성]` 확장
    - `[서브넷 그룹]` : **dbsubnetgroup**
    - `[퍼블릭 액세스 가능]` : **아니요** 클릭
    - `[VPC 보안 그룹]` : **DBSecurityGroup (VPC)** 클릭 추가, default 보안 그룹 **X** 클릭 삭제
- 1.4.8	`[추가 구성]`에서 다음 정보를 입력하고 나머지 값은 모두 기본값 적용
    - `[초기 데이터베이스 이름]` : **sampledb** 입력
    - `[자동 백업 활성화]` : **Check** 해제
- 1.4.9	`[데이터베이스 생성]`를 클릭
- 1.4.10 `[labdbinstance]`를 선택
    - `[정보]`가 **사용가능**으로 변할 때까지 대기 (최대 10분 정도 소요)
- 1.4.11 연결&보안 아래에 있는 `[엔드포인트]`를 복사하여 저장
    - `[엔드포인트]`는 `"labdbinstance.cdl8ki3b1gku.ap-northeast-2.rds.amazonaws.com"`과 같은 형태

<br>

### Task 2 : 데이터베이스와 상호 작용
- 개요
    - 이 작업에서는 이전 실습에서 생성한 웹 서버에 배포된 PHP 웹 애플리케이션을 통해 데이터베이스와 상호 작용합니다.

<br>

### Task 2.1 : 데이터베이스 웹 애플리케이션에 액세스
웹 서버에서 실행되는 웹 애플리케이션을 확인합니다.
- 2.1.1	`[서비스]` 메뉴에서 `[EC2]`를 클릭
- 2.1.2	EC2 탐색 창에서 `[인스턴스]`를 클릭
- 2.1.3	`[Web Server1]`을 선택하고, 모든 다른 인스턴스가 선택 해제되었는지 확인
    - 아래쪽 창에 있는 `[설명]` 탭까지 아래로 스크롤
- 2.1.4	`[Web Server1]`의 `[퍼블릭 DNS(IPv4)]`주소를 복사
- 2.1.5	`[퍼블릭 DNS(IPv4)]`주소를 새 브라우저 탭 또는 창에 복사
    - 웹 애플리케이션이 웹 서버의 인스턴스 메타데이터와 함께 표시
- 2.1.6	AWS 로고 오른쪽 옆에 있는 `[RDS]` 링크를 클릭
- 2.1.7	다음 정보를 입력
    - `[Endpoint]` : 앞에서 복사한 엔드포인트를 붙여 넣습니다. 3306이 생략되어 있는지 확인
    - `[Database]` : **sampledb**
    - `[Username]` : **labuser**
    - `[Password]` : **labpassword**

![image](/images/Day1-Connect-RDS1.png)

- 2.1.8	`[Submit]`을 클릭. 연결 문자열이 표시된 후
    - 10 초 후 페이지 이동 (2개의 새로운 레코드가 주소 테이블에 추가되어 아래와 같이 표시)

![image](/images/Day1-Connect-RDS2.png)

- 2.1.9	또 다른 담당자를 추가하려면 `[Add Contact]`을 클릭
    - `[Name]`, `[Phone]` 및 `[Email]`을 입력하고 `[제출]`을 클릭
- 2.1.10 담당자를 수정하려면, `[Edit]`를 클릭하고 필드를 수정한 다음, `[제출]`을 클릭
- 2.1.11 레코드를 삭제하려면, `[Remove]`를 클릭

<br>

실습2를 완료하셨습니다. 축하드립니다!

웹 사이트를 위한 관계형 데이터 스토어 구성을 성공적으로 완료했습니다. ^^<br>

<br>

* * *

## 실습3. AWS 아키텍처를 확장 및 로드 밸런싱

* * *

### 3-1. 개요
- 본 실습은 이전 실습을 기반으로 합니다.
- `[Elastic Load Balancing(ELB)]`과 `[Auto Scaling]` 서비스를 사용합니다.
- 인프라를 로드 밸런싱하고 자동 조정하는 과정을 살펴봅니다.

![image](/images/Day1-Description-ELB.png)

`[Elastic Load Balancing(ELB)]`은 수신되는 애플리케이션 트래픽을 여러 Amazon EC2 인스턴스에 자동으로 분산합니다. 애플리케이션 트래픽을 라우팅하는 데 필요한 로드 밸런싱 용량을 원활하게 제공하여 애플리케이션의 내결함성을 달성하도록 해줍니다. Elastic Load Balancing은 고가용성, 자동 조정 세 가지 유형의 로드 밸런서를 제공합니다. 실습3.에서는 Classic Load Balancer를 생성하여 여러 EC2 인스턴스에 걸쳐 간단하게 트래픽을 로드 밸런싱을 실습해보겠습니다.

![image](/images/Day1-Description-AutoScaling.png)

`[Auto Scaling]`을 사용하면 애플리케이션 가용성을 보다 간편하게 관리할 수 있으며, Amazon EC2 용량이 사용자가 정의한 조건에 따라 자동으로 확장 또는 축소됩니다. Auto Scaling을 활용하면 실행 중인 Amazon Ec2 인스턴스의 수를 원하는 수준으로 유지할 수 있습니다. 또한, Auto Scaling은 수요가 급증할 경우 Amazon EC2 인스턴스의 수를 자동으로 증가시키기 때문에 성능을 그대로 유지할 수 있으며, 수요가 적을 경우 자동으로 용량을 감소시켜 비용 낭비를 막아줍니다.


### 3-2. 목표 
본 실습을 마친 후 다음을 할 수 있게 됩니다.
- 실행중인 EC2 인스턴스에서 Amazon 머신 이미지(AMI)를 생성
- AWS ELB 로드 밸런서 추가
- EC2 Auto Scaling 시작 구성 생성
- EC2 Auto Scaling 그룹 생성
- 프라이빗 서브넷 내에서 새 EC2 인턴스를 자동 조정
- AWS CloudWatch 경보 생성
- AWS 인프라의 성능을 모니터링


### 3-3. 기간
본 실습에는 약 50분 정도가 소요


### Task 1 : Auto Scaling

- 개요
    - 이 작업에서는 인프라를 생성하고 자동 조정합니다.

- 시나리오

다음 인프라에서 시작합니다.

![image](/images/Day1-Task-RDS.png)

다음 인프라를 구축합니다.

![image](/images/Day1-Task-ELB.png)

<br>

### Task 1.1 : Auto Scaling 용 AMI 생성
이 작업에서는 Auto Scaling에서 사용할 새로운 인스턴스를 시작하는 출발점으로 AMI를 생성합니다.
- 1.1.1	`[AWS Management Console]`의 `[서비스]` 메뉴에서 `[EC2]`를 클릭
- 1.1.2	탐색 창에서 `[인스턴스]`를 클릭
- 1.1.3	`[Web Server 1]`의 `[상태 검사]탭` 2/2 checks passed로 표시되어 있는지 확인
    - 아닌 경우, 변경될 때까지 기다렸다가 다음 단계로 진행
    - 오른쪽 위에 있는 새로고침 버튼을 사용하여 업데이트를 확인
- 1.1.4	`[Web Server 1]`을 오른쪽 마우스 클릭, `[이미지]`를 클릭, `[이미지 생성]`를 클릭
- 1.1.5	다음 정보를 입력, 나머지 값은 그대로 기본값 적용
    - `[이미지 이름]` : **Web Server AMI**
    - `[이미지 설명]` : **Lab 3 AMI for Web Server**
    - `[재부팅 안 함]` : **Check v**
- 1.1.6	`[이미지 생성]`를 클릭
- 1.1.7	보류중인 이미지 보기 클릭하여 생성된 `[AMI ID]`가 정보를 확인 

<br>

### Task 1.2 : 로드 밸런서 추가
이 작업에서는 로드 밸런서를 생성하여 2개의 가용 영역에서 여러 EC2 인스턴스에 걸쳐 트래픽을 로드 밸런싱합니다.
- 1.2.1	EC2 탐색 창에서 `[로드밸런서]`를 클릭
- 1.2.2 `[로드 밸런서 생성]`를 클릭, 다음 화면에서 Classic Load Balancer를 선택
- 1.2.3 다음 정보를 입력, 나머지 값은 그대로 기본값 적용
    - `[로드 밸런서 이름]` : **Lab3ELB**
    - `[LB 생성할 VPC]` : **My Lab VPC** 선택
    - `[리스너 구성]` : **HTTP 80포트** 기본 값 유지
    - `[서브넷 선택]` : [+]를 클릭하여 **Public_Subnet1** 및 **Public_Subnet2**를 선택
- 1.2.4 `[다음: 보안 그룹 할당]`를 클릭
    - **default** 보안 그룹을 Check 해제
    - 이름에 **WebSecurityGroup**이 포함된 보안 그룹을 선택
- 1.2.5 `[다음: 보안 설정 구성]`을 클릭
- 1.2.6 본 실습에서는 보안 리스너는 구성하지 않음, `[다음 상태 검사 구성]`을 클릭
- 1.2.7 다음 정보를 입력, 나머지 값은 그대로 기본값 적용
    - `[Ping 경로]` : **/index.php** (기본값과 다른 값)
    - `[응답 시간 초과]` : **6**입력
    - `[정상 임계 값]` : **2**를 선택
- 1.2.8 `[다음: EC2 인스턴스 추가]` 클릭
- 1.2.9 다음 작업에서 로드 밸런서에서 EC2 인스턴스(Web Server1)를 추가, `[다음: 태그 추가]`를 클릭
- 1.2.10 `[검토 및 생성]`를 클릭
- 1.2.11 로드 밸런서의 구성을 확인하고 `[생성]`를 클릭
- 1.2.12 `[닫기]`를 클릭
- 1.2.13 `[Lab3ELB]`를 선택, `[인스턴스]`탭에서 `[상태]`의 서비스 중인 인스턴스0/1에서 1/1이 되는지 확인
- 1.2.14 `[설명]`탭으로 이동 후 로드 밸런서 *DNS 이름인 **Lab3ELB-1234897470.ap-northeast-2.elb.amazonaws.com**`을 복사 (이때 (A 레코드)는 복사에서 생략)
    - 주소창에 로드 밸런서의 DNS Name을 붙여 넣기
    - 로드 밸런서를 통한 EC2 웹 서버 확인

<br>

### Task 1.3 : 시작 구성 및 Auto Scaling 그룹 생성
이 작업에서는 Auto Scaling 그룹에 대한 시작 구성을 생성합니다.
- 1.3.1	EC2 탐색 창에서 AUTO SCALING `[시작 구성]`를 클릭
- 1.3.2	`[시작 구성 생성]`을 클릭
- 1.3.3	탐색 창에서 `[내 AMI]`를 클릭
- 1.3.4	앞에서 생성한 `[Web Server AMI]`에서 `[선택]`를 클릭
- 1.3.5	`[t2.micro]` 클릭하고 `[다음: 세부 정보 구성]`을 클릭
- 1.3.6	다음 정보를 입력하고, 나머지 값은 그대로 기본값 적용
    - `[이름]` : **Lab3Config**
    - `[모니터링]` : **CloudWatch 세부 모니터링 활성화** 선택
- 1.3.7	`[다음: 스토리지 추가]`를 클릭
- 1.3.8	`[다음: 보안 그룹 구성]`을 클릭
- 1.3.9	`[기존 보안 그룹 선택]`을 클릭, 이름이 `[WebSecurityGroup]`인 보안 그룹을 선택
- 1.3.10 `[검토]`를 클릭
- 1.3.11 시작 구성의 세부 정보를 확인, `[시작 구성 생성]`을 클릭
- 1.3.12 `[기존 키 페어 선택`]을 클릭, **AWSLABS key pair**를 선택
    - 승인 확인란을 선택한 후 `[시작 구성 생성`]을 클릭 후 닫기 버튼을 클릭
- 1.3.13 **Lab3Config**을 Check 후 `[Auto Scaling 그룹 생성]`을 클릭
    -  다음 정보를 입력, 나머지 값은 그대로 기본값 적용
    - `[그룹 이름]` : **Lab3ASGroup**
    - `[그룹 크기]` : **2**개의 인스턴스로 시작
    - `[네트워크]` : **My Lab VPC** 선택
    - `[서브넷]` : **Private_Subnet_3 (10.0.3.0/24)** 및 **Private_Subnet_4 (10.0.4.0/24)** 선택
- 1.3.14 아래로 스크롤하여 `[고급 세부 정보]`를 확장 후 `[하나 이상의 로드 밸런서에서 트래픽 수신]`을 선택
- 1.3.15 `[클래식 로드 밸런서]` 텍스트 상자를 클릭, `[Lab3ELB]`를 클릭
- 1.3.16 다음 정보를 입력, 나머지 값은 그대로 기본값 적용
    - `[상태 검사 유형]` : **ELB** 선택
    - `[모니터링]` : **CloudWatch 세부 모니터링 활성화** 선택
- 1.3.17 `[다음: 조정 정책 구성]`을 클릭
- 1.3.18 `[조정 정책을 사용하여 이 그룹의 용량 조정]`을 선택
- 1.3.19 **2**개와 **6**개의 인스턴스 범위 내에서 조정할 수 있도록 `[조정 범위]` 텍스트 상자를 수정
    - **단계 또는 단순 조정 정책을 사용하여 Auto Scaling 그룹 조정**을 클릭 
- 1.3.20 `[그룹 크기 증가]`에서 `[정책 실행 요건]`에 대해 `[새 경보 추가]`을 클릭
- 1.3.21 `[알림 보낼 대상:]`이 선택되어 있는지 확인한 후, `[주제 생성]`을 클릭
    - 이메일 알람을 생성하는 것은 선택 사항이며, 이 단계를 생략 가능
    - 이메일 알림을 받지 않으려면, `[알림 보낼 대상:]`을 선택 해제
- 1.3.22 다음 정보를 입력하고, 나머지 값은 그대로 기본값 적용 
    - `[알림 보낼 대상]` : **주제 이름 직접 입력**
    - `[수신자]` : **이메일 주소를 입력**
    - `[다음경우 항상]`: **Average** / **CPU 사용률**
    - `[이(가)]` : >= **65%**
    - `[최소 다음의 경우]` : **1** 연속 기간 **1분**
    - `[경보 이름]` : **HighCPUUtilization**
- 1.3.23 `[경보 생성]`을 클릭
- 1.3.24 `[그룹 크기 증가]`의 나머지 부분에 다음 정보를 입력
    - `[작업 수행]` : 추가 선택. **1**입력, 인스턴스 선택. 조건 **65**입력
    - `[인스턴스 필요 시간]` : 각 단계 후 워밍업 시간에 **60**초
- 1.3.25 `[그룹 크기 감소]`에서 `[정책 실행 요건]`에 대해 `[새 경보 추가]`을 클릭
- 1.3.26 `[알림 보낼 대상:]`이 선택되어 있는지 확인한 후, `[주제 생성]`을 클릭
    - 이메일 알람을 생성하는 것은 선택 사항이며, 이 단계를 생략 가능
    - 이메일 알림을 받지 않으려면, `[알림 보낼 대상:]`을 선택 해제
- 1.3.27 다음 정보를 입력하고, 나머지 값은 그대로 기본값 적용
    - `[다음경우 항상]`: **Average** / **CPU 사용률**
    - `[이(가)]` : <= **20%**
    - `[최소 다음의 경우]` : **1** 연속 기간 **1분**
    - `[경보 이름]` : **LowCPUUtilization**
- 1.3.28 `[경보 생성]`을 클릭
- 1.3.29 `[그룹 크기 감소]`의 나머지 부분에 다음 정보를 입력
    - `[작업 수행]` : 제거 선택. **1**입력, 인스턴스 선택. 조건 **20**입력
- 1.3.30 `[다음: 알림구성]`을 클릭
- 1.3.31 `[다음: 태그구성]]`을 클릭
- 1.3.32 다음 정보를 입력하고, 나머지 값은 그대로 기본값 적용
    - `[키]` : **Name**
    - `[값]` : **Lab3Weblnstance**
- 1.3.33 `[검토]`를 클릭
- 1.3.34 Auto Scaling 그룹의 세부 정보를 확인한 다음, `[Auto Scaling 그룹 생성]`을 클릭
- 1.3.35 Auto Scaling 그룹이 생성되면 `[닫기]`를 클릭
- 1.3.36 * Auto Scaling 그룹에 대한 알림 구독을 확인하는 이메일을 수신 (알림 보낼 대상을 설정했을 경우에만)
    - 이메일을 열고 `[Confirm subscription link]`를 클릭

<br>

### Task 1.4 : Auto Scaling이 작동하는지 확인하고 인스턴스를 로드 밸런서에 추가 
이 작업에서는 Auto Scaling이 올바르게 작동하고 있는지 확인합니다.
- 1.4.1	EC2 탐색 창에서 `[인스턴스]`를 클릭
- 1.4.2	4대의 인스턴스 확인 
    - `[[Web Server 1]` 1대
    - `[NAT Server]` 1대
    - `[Lab3WebInstance]`라는 이름의 새로운 인스턴스 2대
- 1.4.3	EC2 탐색 창에서 `[로드 밸런서]`를 클릭
- 1.4.4	`[Lab3ELB]`를 선택하고, 아래로 스크롤하여 `[인스턴스]` 탭을 클릭
    - 이 로드 밸런서 목록에 `[Lab3WebInstance]` 2대가 새롭게 추가된 것을 확인
- 1.4.5	`[Lab3ELB]`의 `[인스턴스]` 탭을 클릭
    - 이 인스턴스의 `[상태]`가 `[InService]`로 바뀔 때까지 대기
    - 오른쪽 위에 있는 `[새로고침]` 버튼을 사용하여 업데이트를 확인
- 1.4.6	로드 밸런서가 인스턴스의 `[상태]` 필드 아래에 `[InService]`를 표시
- 1.4.7 퍼블릭 액세스 가능한 최초 웹 서버는 더 이상 필요없기에 아래와 같이 삭제를 진행
    - `[서비스]` 메뉴에서 `[EC2]`를 클릭
    - EC2 탐색 창에서 `[인스턴스]`를 클릭
    - `[Web Server 1]`을 마우스 오른쪽 버튼으로 클릭
    - `[인스턴스 상태]`에서 `[종료]`를 클릭

<br>

### Task 2 : 인프라 모니터링

- 개요
    - 인스턴스가 최소 2개, 최대 6개인 Auto Scaling 그룹을 생성합니다.
    - Auto Scaling 그룹을 한번에 인스턴스 하나씩 증가하거나 축소하는 AutoScaling 정책을 생성합니다.
    - 그룹의 전체 평균 CPU 사용률이 >= 65% 일 때 해당 정책을 트리거하는 CloudWatch 경보를 생성합니다.
    - 그룹의 전체 평균 CPU 사용률이 <= 20% 일 때 해당 정책을 트리거하는 CloudWatch 경보를 생성합니다.
    - 최소 크기는 2이고 부하가 전혀 없으므로 현재 2개의 인스턴스가 실행중입니다.
    - 생성한 CloudWatch 경보를 사용하여 이제 인프라를 모니터링할 수 있습니다.

<br>

### Task 2.1 : Auto Scaling 테스트
본 작업에서는 앞에서 구현한 Auto Scaling 구성을 테스트합니다.
- 2.1.1	`[서비스]` 메뉴에서 `[CloudWatch]`를 클릭
- 2.1.2	탐색 창에서 `[경보]`탭에 경보를 를 클릭
- 2.1.3	`[HighCPUUtilization]`과 `[LowCPUUtilization]` 이라는 2개의 경보 확인
    - `[LowCPUUtilization]`은 `[상태]`가 `[경보상태]`임을 확인
    - `[HighCPUUtilization]`은 `[상태]`가 `[정상]`임을 확인
    - 이는 현재 그룹 CPU 사용률이 20% 미만이기 때문
    - 그룹 크기가 현재 최소 크기인 (2)이므로, Auto Scaling은 어떠한 인스턴스도 제거하지 않음
- 2.1.4 단계 1.2.13에서 복사한 로드 밸런서의 DNS 이름을 새 브라우저 창 또는 탭에 복사 
- 2.1.5	AWS 로고 오른쪽 옆에 있는 `[LOAD TEST]`를 클릭
    - 애플리케이션이 인스턴스에 대한 부하 테스트를 수행하고 5초 간격으로 자동 새로 고침
    - `[Current CPU load]`가 **100%** 상승한 것을 볼 수 있음
    - `[Load Test]` 링크는 간단한 백그라운드 프로세스를 트리거
- 2.1.6	`[서비스]` 메뉴에서 `[CloudWatch]`를 클릭
    - 5분 이내에 `[LowCPUUtilization]` 경보 상태가 `[정상]`으로 변경
    - 5분 이내에 `[HighCPUUtilization]` 경보 상태가 `[경보 상태]`로 변경되는 것을 확인
- 2.1.7	`[서비스]` 메뉴에서 `[EC2]`를 클릭
- 2.1.8	EC2 탐색 창에서 `[인스턴스]`를 클릭
- 2.1.9	이제 `[Lab3Weblnstance]`라는 이름의 인스턴스가 2개 이상 실행되는 것을 볼 수 있음
- 2.1.10 단계 2.1.4에서 연 브라우저 탭 또는 창을 닫음

<br>

실습3을 완료하셨습니다. 축하드립니다!

본 작업에서는 퍼블릭 서브넷2의 웹 서버 1을 종료하였고, Auto Scaling으로 EC2 웹 서버를 프라이빗 서브넷에 추가하고 Elastic Load Balancer을 사용하여 인프라의 부하분산을 하는 작업을 성공적으로 완료했습니다.

<br>

## Terminate Resources

10분

1. Delete Your Amazon RDS
RDS부터 지워봅시다

- `[AWS Management Console]` ▷ `[RDS]`
    - **labdbinstance**을 클릭하고 상단 `[작업]` ▷ `[삭제]` 클릭
        - 최종 스냅샷 생성 여부 **Check** 해제
        - 인스턴스 삭제 시 시스템 스냅샷 및 특정 시점으로 복구를 포함한 자동화된 백업을 더 이상 사용할 수 없다는 항목에 **Check**
        - 삭제를 확인하려면 아래 필드에 **delete me**라는 문구를 입력 후 `[삭제]` 클릭

<br>

2. Delete Your EC2 (AutoScaling Instance)
AutoScaling으로 생성된 EC2를 지워봅시다
- `[AWS Management Console]` ▷ `[EC2]` ▷ `[Auto Scaling]` 그룹 클릭
    - **Lab3ASGroup**을 클릭하고 상단 `[작업]` ▷ `[삭제]` 클릭

- `[AWS Management Console]` ▷ `[EC2]` ▷ `[시작 구성]` 클릭
    - **Lab3Config**을 클릭하고 상단 `[작업]` ▷ `[시작 구성 삭제]` 클릭

- `[AWS Management Console]` ▷ `[EC2]` ▷ `[AMI]` 클릭
    - **Web Server AMI**의 AMI ID를 기록 (예: ami-0ea06fc7ff2beb3ae)
    - **Web Server AMI**를 클릭하고 상단 `[작업]` ▷ `[등록 취소]`▷ `[계속]` 클릭

- `[AWS Management Console]` ▷ `[EC2]` ▷ `[스냅샷]` 클릭
    - **Web Server AMI**의 AMI ID로 검색
    - 검색된 스냅샷을 클릭하고 상단 `[작업]` ▷ `[삭제]` ▷ `[예, 삭제]`를 클릭

<br>

3. Delete Your EC2 (NAT Instance)
VPC 마법사로 생성된 EC2 Instance를 지워 봅시다
- `[AWS Management Console]` ▷ `[EC2]` 클릭
    - **NAT Instance**을 클릭하고 상단 `[작업]` ▷ `[인스턴스 상태]` ▷ `[종료]` 클릭
- `[AWS Management Console]` ▷ `[EC2]` ▷ `[탄력적 IP]`클릭
    - **NAT Instance**가 사용했떤 탄력적 IP를 클릭하고 `[작업]` ▷ `[주소 릴리스]` ▷ `[릴리스]` 클릭

<br>

4. Delete Your ELB
ELB도 지워봅시다
- `[AWS Management Console]` ▷ `[EC2]` ▷ `[로드 밸런서]` 클릭
    - **Lab3ELB**을 클릭하고 상단 [작업] ▷ `[삭제]` 클릭 ▷ `[예, 삭제]`


수고하셨습니다.



