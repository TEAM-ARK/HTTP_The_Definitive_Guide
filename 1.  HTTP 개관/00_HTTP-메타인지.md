HTTP 완벽가이드 책 스터디를 진행하기로 해서 읽고 있다.
그럼 HTTP란 무엇일까?

## 네트워크 데이터 전송 프로토콜

- 네트워크의 데이터를 전송하는 프로토콜 5종류

  - TCP, UDP, IP, ICMP, ARP
    - TCP, UDP, IP : 실제로 데이터를 전송하는 프로토콜
    - ICMP, ARP : 데이터 전송을 도와주고 보조해주는 프로토콜

- TCP(Transmission Control Protocol)
  - Layer 4계층 프로토콜이라고도 불림
  - 연결 지향성, 상대방과 통신 연결을 실시한 뒤 데이터 요청이나 응답을 함
  - 3-Way 핸드 쉐이킹으로 동작
    - 3-Way 핸드 쉐이킹
      - 클라이언트가 syn을 서버로 보냄
      - 서버가 받으면 syn, ack를 클라이언트에 보냄
      - 클라이언트는 ack를 서버에 보냄
    - [ ] preflight 에 대해 알아보기
      - http://wiki.gurubee.net/display/SWDEV/CORS+%28Cross-Origin+Resource+Sharing%29
  - TCP로 사용하는 것들 : HTTP(80), HTTPS(443), telnet(23), ssh(22), ftp(21), ftp-data(20), smtp(25), pop3(110)
- UDP(User Datagram Protocol)
  - Layer4 계층, 비연결 지향성
  - TCP에 비해 특별한 특징이나 기능이 전혀 없고 오류검사만 함
  - UDP에 사용되는 것들 : DNS(53), tftp(69), bootpc(dhcp client 68), bootps(dhcp server 67), snmp(161), ntp(123)
    - 서로 연결할 필요가 없는 서비스를 사용
- IP(Internet Protocol)
  - Layer3 계층, 비연결 지향성
  - 로컬환경에서 리모트 환경으로 데이터의 전송을 담당
  - TTL(Time to Live)로 거리측정과 패킷루프를 방지할 수 있음
    - Windows : 128, Linux 64, Cisco 255
    - 라우터 마다 TTL 1씩 감소 : 요청 TTL이 128이고 3개의 라우터를 거치면 125가 됨
- ICMP(Internet Control Message Protocol)

  - IP프로토콜을 이용한 데이터 전송 가능 여부를 테스트하는 프로토콜
  - 네트워크를 구축하고 정식 서비스 전에 테스트 용도로 사용
  - 명령어 : ping, tracert
    - tracer 명령어 : 처음 TTL을 1로 보내고 막히면 2로 보내면서 점점 숫자를 증가시켜서 목적지에 도착하면 끝냄
  - ICMP는 대형 포털사이트에선 막아놓음(디도스 공격 같은 것을 줄 수 있기 때문)
    - [ ] 디도스 공격에 대해 알아보기
  - 윈도우에서도 개인컴퓨터로 오는 ICMP를 막을 수 있음(제어판>방화벽>고급설정>인바운드규칙)

- ARP(Address Resolution Protocol)
  - 주소변환 프로토콜, 목적지 IP주소에 대한 MAC 주소를 설정하는 프로토콜

### 참고

- https://gsk121.tistory.com/90

## MAC Address

- 개념적 예시 : 주민등록번호
- 단 하나의 고유한 주소를 부여해서 통신을 할 수 있도록 만든 일종의 하드웨어 주소
- 통신을 하기 위한 모든 랜카드에 고유한 MAC 주소가 존재
  - 이 세상에 딱 하나만 존재하는 주소

### IP 주소 vs Mac 주소

#### IP 주소

- 인터넷을 하기 위해 사용되는 주소
  - [ ] IP 주소가 의미하는 것에 대해 알아보기
- IP 주소만으로 통신할 수 없음
  - TCP/IP와 OSI 7Layer 통신표준에서 IP주소는 레이어 3 네트워크에서 사용되고
  - Mac 주소는 레이어 2 데이터 링크에서 사용되는 주소
- 통신 표준에 맞게 단게별로 진행 될 때, IP주소에서 MAC주소로 변환되는 과정을 거침
  - ARP(Address Resolution Protocol)

#### 통신 절차

- Mac주소로는 통신하고 싶은 장비를 찾을 수 있음
- 그렇지만 처음 통신은 IP주소 부터 시작 됨
- e.g., 내 IP 주소가 192.168.0.100 이라면 나와 통신하는 상대방은 192.168.0.0 네트워크 대역까지 찾아옴
- 그 뒤 192.168.0.100을 찾고 내 컴퓨터 Mac주소로 목적지를 변환한 뒤 최종적으로 나와 통신할 수 있게 됨
- 인터넷 상 통신은 중간에 수많은 ip 주소와 Mac 주소가 변환되는 과정을 거침
- 목적지를 찾고 해당 IP에 연결된 목적지의 Mac주소를 찾고
- 다시 IP를 찾고 Mac주소를 찾고를 반복하는 과정

#### MAC : Media Access Control

- 랜카드에 하나의 고유한 Mac adress가 존재
- 총 48비트 길이의 주소
- 8자리 마다 하이픈(-), 콜론(:), 점(.)으로 구분.
  - e.g.,
    - 12:34:56:78:90:11
    - 00-00-00-00-00-00
    - 0000.0000.0000
- 48비트로 주소를 2진수로 표시하면 너무 길기 때문에 16진수로 나눠서 표시
- 보통 앞의 16 진수 6개는 OUI(Organiztional Unique Identifier)를 말함
  - 어느회사에서 만들었는지 식별할 수 있음
- 나머지 6개는 호스트 식별자, 각 회사에서 임의로 붙이는 일종의 시리얼 넘버
- (앞)12:34:56 => OUI 주소
- (뒤)78:90:11 => 호스트 주소(시리얼넘버 같은 것)
- CMD창에서 ipconfig /all 이라고 입력하면 Mac adrress를 확인할 수 있고 구글에 OUI vender라고 검색해서 나오는 결과로 내 랜카드가 어느회사에서 만들어졌는지 알 수 있음
  - 내 컴퓨터의 랜카드는 인텔에서 만들어졌다.
  - 7C:5C:F8 Intel Corporate

### 참고

- https://m.blog.naver.com/wood0513/222084400286

## 프로토콜

- 사람과 사람이 통신할때 서로 이해할 수 있는 언어, 공용된 언어를 사용해 전세계 모든 사람과 대화 할수 있다라고 하면,
  컴퓨터와 컴퓨터도 서로 이해 할 수 있는 언어, 공용된 언어를 사용 해야 한다는 것인데
  이 것이 바로 프로토콜(Protocol) 입니다.
- 프로토콜은 원래 외교상의 언어로써 의례나 국가간의 약속을 의미
- 통신에서 프로토콜은 어떤 시스템이 다른 시스템과 통신을 원활하게 수용하도록 해주는 통신 규약, 약속
- 프로토콜의 기능(프로토콜 마다 전체 또는 일부만 포함)
  - 세분화와 재합성
  - 캡슐화
  - 연결제어
  - 오류제어
  - 흐름제어
  - 동기화
  - 순서 결정
  - 주소 설정
  - 다중화
  - 전송 서비스
- 프로토콜의 특성에 따른 분류
  - 직접 / 간접 프로토콜
  - 단일체 / 구조적 프로토콜
  - 대칭 / 비대칭 프로토콜

### 참고

- [프로토콜이란](https://mindnet.tistory.com/entry/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-%EC%89%BD%EA%B2%8C-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-9%ED%8E%B8-%ED%94%84%EB%A1%9C%ED%86%A0%EC%BD%9C-%EC%9D%B4%EB%9E%80-Protocol-%EC%9D%B4%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80)

## TCP/IP

- TCP/IP는 1960년대 후반 부터 장비간 통신을 위해 미국방성(DoD:Department of Defence)에서 개발하여 만들어진 프로토콜
- 현재는 거의 모든 컴퓨터가 기본으로 제공하여 인터넷 표준 프로토콜이 됨
- 1974년 전송제어프로토콜(TCP: Transmission Control Protocol)이 나옴
- 1981년 IP(Internet Protocol)이 RFC791표준으로 1982년 TCP/IP가 표준 프로토콜로 지정 됨
- Open Protocol, 누구나 적용하고 보완할 수 있음
- 4개의 계층적 구조를 가짐
  - 각 계층마다 독립적인 기능을 가며 다른 계층에 영향을 미치지 않음
  - Application, Transport, Internet, Network interface
    - 이를 좀 더 세분화하여 계층화 한 것이 OSI 7 Layer
      ![](https://t1.daumcdn.net/cfile/tistory/21338533520473E005)
- OSI 7 Layer 중 3, 4
  - 3 Layer : Internet Layer
    - 인터넷 계층
    - IP, IGMP, ICMP, Routing Protocol, ARP
      - IP(Internet Protocol)로써 논리적 주소를 통해 최적 경로를 선택하여 데이터를 전송할 수 있게 함
        - 이 때 Routed Protocol(IP, IPX, Apple Talk)와 Routing Protocol(RIP, IGRP, EIGRP, OSPF, IS-IS, BGP)를 제공
      - ICMP(Internet Control Message Protocol)은 패킷 전송시 발생한 메세지와 에러 정보를 알려주게 됨
      - ARP(Address Resolution Protocol)은 논리적 주소(IP와 같은)을 통해 상대방의 MAC Address를 찾음
      - RARP(Reverse ARP)는 ARP의 반대 개념으로 물리적 주소를 통해 논리적 주소를 알려줌
  - 4 Layer : Transport Layer
    - 전송 계층, 데이터 전송방식을 결정
    - TCP, UDP
      - TCP(Transmission Control Protocol) : Connection-Oriented 패킷 전송을 제어하며 데이터 전송 시 에러를 복구할 수 있으며 신뢰성을 가짐
      - UDP(User datagram Protocol)
        - Connectionless 패킷 전송 제어.
        - 데이터 전송 시 발생한 에러를 복구할 수는 없으나 오버헤드가 적어 속도가 빠른편

### 참고

- [TCP/IP Model Layer, OSI 7 Layer](https://mindnet.tistory.com/entry/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-%EC%89%BD%EA%B2%8C-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-10%ED%8E%B8-OSI-7-Layer-TCPIP-Model-Layer?category=702276)

## OSI 7계층 (Layer)모델

- Open Systems Interconnection Reference Model
- 국제 표준화 기구(ISO)에서 개발한 모델, 컴퓨터 네트워크 프로토콜 디자인과 통신을 계층별로 나누어 설명한 것
- OSI 7 Layer 같이 복잡한 계층으로 이루어진 이유에 대한 간단한 비유
  - 편지를 보내는 과정
    - 편지를 써서 봉투에 담는다
    - 편지봉투에 주소를 적는다
    - 우체국에 가서 일반인지 등기인지 결정하여 우표를 붙인다
    - 우체국에선 편지봉투위에 적힌 주소를 보고 배달을 한다.
    - 편지를 받은 사람은 자신한테 온 편지가 맞다면 그 내용을 확인한다.
- OSI 7 Layer를 공부하는 이유
  - 데이터 흐름을 알 수 있다.
  - 문제해결(Trouble shooting)이 쉬워진다.

#### 인터넷

- 서버와 클라이언트의 질의응답 과정

##### 인터넷에서 통신 방식

- 각 기기들이 유선 또는 무선으로 통신을 함

##### WWW(World Wide Web)

- 인터넷을 통한 상호 연결된 웹페이지 시스템
- 구성 요소
  - HTTP protocol
  - URI
  - HTML
- www은 인터넷위에 구축된 많은 응용 프로그램 중 하나
- Tim Berners-Lee가 WWW으로 알려진 아키텍처를 제안

#### 네트워크

- ARPANET(1970년대)의 NCP(Network control Program)이 개정된 것이 TCP/IP

##### TCP/IP

- 인터넷을 사용할 때 따라야 할 규칙(프로토콜)

#### OSI 7 Layer

- 네트워크를 구상할 때 사용하는 참조 모델
- divide and conquer
- 두 객체간 통신할 때 과정을 7단계로 나누어 놓은 것
  - 각 계층에 따라 역할이 나누어져 있음

##### 7계층. Application layer

- 편지를 보낼 때 편지지에 해당하는 부분
- 전달하고자 하는 글
- 사용자가 특정 어플리케이션을 통해 데이터를 입력하고 가공하는 곳
- HTTP, FTP, SMTP, Telnet 같은 프로토콜들이 속한 계층

##### 6계층. Presentation layer(표현계층)

- 표현하는 계층
- 인코딩(Encoding), 암호화(Encryption), 압축(Compress)
- 서로 다른 시스템이 통신을 할 때, 공통된 표준형식에 맞춰 변형해서 보내게 되는 것

##### 5계층. Session layer(세션 계층)

- 연결에 대해 이야기 함
- 전화처럼 쌍방으로 주고 받을 것인지 / 무전기 처럼 한쪽 씩 번갈아가면서 받을지 / 일방적으로 받기만 할 것인지
- 이러한 연결 회선에 대한 생성, 관리
- 세션복구도 지원함
  - 세션 도커 : 체크포인트라는 것을 통해 동기화
  - e.g., A -> B : 5MB 마다 체크포인를 설정 했다면, 48MB의 데이터를 연결하던 도중 연결이 끊기더라도 체크포인트 덕분에 45MB부터 다시 세션을 재개할 수 있게 됨

##### 4계층. Transport layer(전송 계층)

- 서로 다른 두 네트워크간의 전송을 담당
- 편지를 써서 상대방한테 보낼 때 편지봉투에 발신자와 수신자를 적는 것
- 세그멘테이션, 흐름제어, 오류제어 등을 제공
  - 세그멘테이션 : 상위 계층 데이터를 받아서 세그멘트라는 데이터의 단위로 나누는 것
    - e.g., A(서버) -> B(사용자) : 세그멘테이션을 하지 않는다면 100MB의 비디오가 모두 로딩되고 나서야 비디오를 볼 수 있을 것임
    - 세그먼트 작은 단위로 나누게 되면 비디오 일부분을 볼 수 있게 됨
    - 연결이 중간에 끊기게 되었을 때 세그멘테이션을 하지 않게 되면 큰 데이터가 날라감
  - 흐름제어 : 서로다른 데이터 전송량이 다른 기기에서
    - A(50Mbps - 초당 50Mb 처리) -> B(10Mbps)
      - A가 초당 10Mbps를 처리하는 B에 50Mbps를 보낼 경우, B가 A에게 전송량을 낮춰달라고 요구하면 전송량을 A가 10Mbps로 낮추는 방식
      - 반대로 높이는 요청도 가능
    - Stop and Wait 또는 Sliding Window같은 방식을 사용
  - 오류제어 : 보낸 데이터가 데이터 손실이 없는지 확인하고 만약 오류가 있다면 다시 해당 데이터를 보내주는 것
    - e.g., 3번데이터가 안왔다면 다시 3번 데이터를 보내줌
    - FEC, BEC, ARQ 같은 방식이 있음
- 포트번호
- TCP, UDP에 대한 정의
  - 데이터를 보내는 쪽에서 수신측이 온전히 데이터를 받는지 확인 여부
  - 일반 우편 vs 등기 우편
    - 등기우편 : 잘받았다는 것을 알 수 있음

##### 3계층. Network layer

- IP나 라우터 장비가 속한 계층
- 데이터의 전송을 담당
- 라우팅 : 호스트에 IP번호를 부여, 해당 IP까지 최적의 경로를 찾아주는 기능을 제공
- 경로 설정, 도착지 주소 표현(IP)
  - IP주소 : 192.168.0.1 (논리적 주소)

##### 2계층. Data link layer

- 네트워크 계층과 비슷
  - 차이점
    - 네트워크 계층 : 서로다른 두 네트워크간의 전송을 담당
    - 데이터 링크 계층 : 동일한 네트워크 내 전송을 담당
- 오류제어와 흐름제어를 제공
  - 트랜스포트 계층에도 있지만 여기에도 있음
  - 데이터 계층의 데이터 단위 : Frame
    - e.g., 10개의 프레임이 있다고 가정, 그 중 두개의 프레임이 오류가 났을 때 데이터 링크 계층에서 이 데이터 조각들을 그냥 버림
    - 반면 트랜스포트의 오류제어는 해당 데이터가 없으면 다시 보내줌 - 오류 복구까지 지원
- 몇 동 몇호에 해당하는 것
- MAC주소
- 충돌 방지 시스템

##### 1계층. Physical layer

- 디지털 신호를 전기신호로 전송
- 회선, 부호화, 전기신호 방식 등
- 동기식, 비동기식

##### Encapsulation

- 각 계층을 거치면서 데이터를 합치는 과정(보내는 쪽) : 7->1 단계

##### Decapsulation

- 수신측에서 이 데이터를 열어보기 까지 과정 1->7 단계

#### TCP/IP suite

- OSI 7 Layer와 비슷하지만 약간 다르다.
- 4단계로 압축됨 (7~5 => 4)
  - Application, Presentation, Session => Application

## TCP/IP Model

![](https://cdn.guru99.com/images/1/102219_1135_TCPIPvsOSIM1.png)

- 사실 우리가 사용하고 있는 네트워크 모델은 OSI 모델이 아닌 TCP/IP 모델을 사용
- OSI 모델의 5, 6계층(세션계층, 프레젠테이션계층)이 어플리케이션 계층으로 통합되어 있음

![](https://images.velog.io/images/gth1123/post/f95e8a56-c7e8-4091-920d-adb0c202d2aa/image.png)

- A에서 B로 데이터를 보냄
- 스위치 : 데이터링크 계층의 대표적인 하드웨어
  - 스위치의 캠테이블 : 스위치와 연결되어 있는 라우터의 Mac주소를 갖고 있음
  - 데이터가 오면 해당기기(캠테이블이 가리키는 라우터)로 보내줌
  - 디캡슐레이션을 해서 헤더에 대한 정보를 살펴보고 라우터에 대한 정보가 있으면 그쪽으로 보내줌
- 라우터 : 네트워크 계층의 하드웨어
  - 디캡슐레이션을 두번 해서 도착지에 대한 IP주소를 확인 후 라우팅 테이블을 통해 라우팅(어떤 네트워크 안에서 통신 데이터를 보낼 때 최적의 경로를 선택하는 과정)을 시킨 후 B의 맥주소를 파악한 다음 Layer 2에 해당하는 헤더를 업데이트 시킴
    - 라우팅 후 B에 대한 맥주소로 업데이트 시킴
- B쪽의 스위치에 도착하면 B의 스위치가 캠테이블을 통해 B로 전달

### Application 계층

- 평소 데이터를 보낼때 HTTPS를 사용(이미 어플리케이션 Layer를 사용한 것)
- e.g., A에서 B로 데이터를 보낼 때
  - A : 상위 계층에서 하위 계층으로 내려보내면서(5 -> 1) 계층별 **헤더**를 붙임(encapsulation)
  - B : 캡슐화된 데이터들을 다시 디캡슐레이션을 하면서 데이터를 얻는 방식

=> data

### Transport 계층

- TCP를 사용할 것인지 UDP를 사용할 것인지 정함
  - TCP : 데이터 손실되었는지 확인, 데이터 순서 보장
  - UDP : 데이터를 보내고 나면 그에 대한 책임을 지지 않음
    - 신뢰도는 떨어지지만 빠르고 연속적이기 때문에 스트리밍 같은 서비스에서 많이 사용 됨
  - TCP인지 UDP인지에 대한 정보 헤더를 붙임
- 출발지와 도착지에 대한 포트정보도 헤더에 넣어서 캡슐화 => 세그먼트

=> data + L4 Header : 세그먼트

### Network 계층

- 출발지와 도착지에 대한 IP 정보를 헤더를 만들어서 세그먼트에 붙이고 캡슐화 => 패킷

=> data + L4 Header + L3 Header : 패킷

### Data link 계층

- 출발지인 Mac주소와 가장 가까운 라우터의 Mac주소를 넣음
  - B의 Mac주소를 넣지 않는 이유 : A는 처음에 B에 대한 Mac주소를 알지 못하기 때문
- A는 DHCP와 ARP를 통해서 라우터의 IP를 바꿔 IP를 맥주소로 변환한 후에 라우터에 대한 도착지 Mac 주소를 만든 후 헤더에 넣어줌
- Trailer라는 정보도 붙음
  - 오류제어를 위한 정보

=> L2 Trailer + data + L4 Header + L3 Header + L2 Header : 프레임(Frame)

### 물리 계층(Physical layer)

- 전기신호로 바꾼 다음 데이터를 전송

### 참고

- [그림으로 배우는 네트워크 이야기 OSI 7 Layer](https://youtu.be/aTPy201F0AA)
- [WWW란?](https://medium.com/dream-youngs/www-%EB%9E%80-b2c069c730a4)
- [파즈의 OSI 7 Layer](https://youtu.be/Fl_PSiIwtEo)
