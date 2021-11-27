# 6장. 프락시

- 프락시 서버 : 중개자
- 클라이언트 - 프락시 - 서버
  - HTTP메시지를 정리하는 중개인 처럼 동작한다

#### 이 장에서는 ...

- 프락시 기능에 대한 특별한 지원
- 프락시 서버를 사용할 때 보게 될 몇 가지 교묘한 동작
- HTTP 프락시 서버의 모든 것

#### 이 장에서 다룰 내용

- HTTP 프락시와 웹 게이트웨이를 비교하고 HTTP 프락시가 어떻게 배치되는지 그림으로 보여주면서 설명
- 몇 가지 유용한 활용방법을 보여준다
- 프락시가 실제 네트워크에 어떻게 배치되어 있는지 그리고 트래픽이 어떻게 프락시 서버로 가게 되는지 설명한다
- 브라우저에서 프락시를 사용하려면 어떻게 설정해야 하는지 보여준다
- HTTP 프락시 요청이 서버 요청과 어떻게 다른지,
  - 그리고 프락시가 어떻게 브라우저의 동작을 미묘하게 바꾸는지 보여준다
- 일련의 프락시 서버들을 통과하는 메시지 경로를, Via 헤더와 TRACE 메서드를 이용해 기록하는 방법을 설명한다
  - 로깅?
- 프락시에 기반한 HTTP 접근 제어를 설명한다
- 어떻게 프락시가 클라이언트와 서버 사이에서 각각의 다른 기능과 버전들을 지원하면서 상호작용 할 수 있는지 설명한다

### 갑자기 드는 궁금증

- [ ] 프락시 서버가 해결해주는 문제들은 무엇일까?
- [ ] Forward proxy, Reverse proxy, Load balancing

## 6.1 웹 중개자

- 웹 프락시 서버 : (클라이언트의 입장에서) 트랜잭션을 수행하는 중개인
- 웹 프락시가 없다면 => 클라이언트는 HTTP 서버와 직접 이야기 한다
- 웹 프락시가 있다면 => 클라이언트는 HTTP 서버와 이야기 하는 대신, 자신의 입장에서 서버와 대화해주는 프락시와 이야기 한다
- 프락시 서버가 제공하는 좋은 서비스?
- 프락시 서버는 웹 서버이기도 하고 웹 클라이언트 이기도 하다

![](https://images.velog.io/images/gth1123/post/d33fba7a-b7a1-4cd4-a66e-8a6341f70ee7/image.png)

- 프락시는 웹 클라이언트에서 볼 때 서버처럼 동작하면서 요청 메세지를 받고 응답 메세지를 돌려준다
- 프락시는 웹 서버에서 볼 때 클라이언트 처럼 동작하면서 웹 요청 메세지를 보내고 웹 응답 메세지를 받는다

### 6.1.1 개인 프락시와 공유 프락시

#### 공용 프락시

- 여러클라이언트가 공유
- 대부분의 프락시가 공용이며 공유된 프락시
- 중앙 집중형 프락시를 관리하는게 더 비용효율이 높고 쉽다
- 프락시를 이용하는 사용자가 많을 수록 공통된 요청에서 이득을 취할 수 있음

#### 개인프락시

- 하나의 클라이언트만을 위한 프락시

<details>
<summary>ISP(Internet service provider)</summary>
  
**인터넷 서비스 제공자(Internet service provider; ISP)는 인터넷에 접속하는 수단을 제공하는 주체를 가리키는 말이다.**
 그 주체는 영리를 목적으로 하는 사기업인 경우가 대다수이나 비영리 공동체가 주체인 경우도 있다.

ISP는 일반적으로 인터넷에서 이용 가능한 모든 것에 대한 접근권을 사용자에게 부여하는 액세스 포인트나 게이트웨이 역할을 한다.

- KT, SKT, LG유플러스
- 출처 : [위키백과](https://ko.wikipedia.org/wiki/%EC%9D%B8%ED%84%B0%EB%84%B7_%EC%84%9C%EB%B9%84%EC%8A%A4_%EC%A0%9C%EA%B3%B5%EC%9E%90)

</details>

### 6.1.2 프락시 vs. 게이트웨이

![그림 6-2](https://images.velog.io/images/gth1123/post/bed02756-2d97-4729-98c2-06757651bd68/image.png)

- 그림 6-2

- 프락시 : 같은 프로토콜을 사용하는 둘 이상의 애플리케이션을 연결
- 게이트웨이 : 서로 다른 프로토콜을 사용하는 둘 이상을 연결
- 그림 6-2b는 HTTP 프론트엔드와 POP 이메일 백엔드를 연결
  - Naver메일, Daum메일과 같은 웹 기반 이메일 프로그램은 HTTP 이메일 게이트웨이다

<details>
<summary>메일 관련 프로토콜 : POP, SMTP</summary>
  
- https://help.naver.com/support/contents/contents.help?serviceNo=2342&categoryNo=2284
- https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=sung_mk1919&logNo=221402277615
- https://www.hanbiro.com/temp/print_view.php?bid=spammail&no=1

</details>

- 실질적으로 프락시와 게이트웨이의 차이점은 모호하다
- 상용 프락시 서버는 SSL 보안 프로토콜, SOCKS 방화벽, FTP 접근 그리고 웹 기반 애플리케이션을 지원하기 위해 게이트웨이 기능을 구현한다.
  - 게이트웨이 : 8장

<details>
<summary>프락시 vs. 게이트웨이</summary>

- 둘 다 내부 네트워크를 인터넷으로 라우팅
- 게이트웨이 : 프록시 = '문' : '벽'
- 프록시 서버는 허용된 커넥션만 걸러주지만, 게이트웨이는 어떠한 필터링도 해주지 않는다.
- 출처 : https://coding-start.tistory.com/342

</details>

## 6.2 왜 프락시를 사용하는가?

- 프락시 서버 : 실질적이고 유용한 것이라면 무슨 일이든 함
  - 보안 개선
  - 성능 향상
  - 비용 절약
  - 트래픽 감시

#### 어린이 필터

![](https://images.velog.io/images/gth1123/post/b8852026-0c65-4273-b9c7-e204ab1c0a7e/image.png)

- 부적절한 사이트의 접근을 강제로 거부

#### 문서 접근 제어자

![](https://images.velog.io/images/gth1123/post/a7f969e8-7459-492f-ab75-d8a1710c949f/image.png)

- 접근 제어 전략을 구현하고 감사 추적(audit trail)
- 필터랑 비슷한데 비밀번호를 묻는 차이가 있는 것 같다

#### 보안 방화벽

![](https://images.velog.io/images/gth1123/post/e36da637-ca9c-465b-88c0-54d120b590f4/image.png)

- 조직 안에 들어오는 프로토콜의 흐름을 통제

#### 웹 캐시

![](https://images.velog.io/images/gth1123/post/d65272f5-4caf-4771-940b-657f5477181c/image.png)

- 인기 있는 문서의 로컬 사본을 관리
  - 프락시에 있는 데이터는 언제 갱신될까?
    - 각 서비스마다 다를 것 같다
    - https://docs.oracle.com/cd/E19438-01/819-3161/agcache.html

#### 대리 프락시

![](https://images.velog.io/images/gth1123/post/78530049-34e3-44e1-a1fe-091b2462b5d5/image.png)

- 웹 서버인 것 처럼 위장
  - 리버스프락시 == 서버 가속기
- 리버스 프락시로 불리는 이들은 진짜 웹 서버 요청을 받지만 웹 서버와는 달리 요청 받은 콘텐츠의 위치를 찾아내기 위해 다른 서버와 커뮤니케이션을 함
- 공용 콘텐츠에 대한 느린 웹 서버의 성능을 개선하기 위해 사용될 수 있다
- 콘텐츠 라우팅기능과 결합되어 주문형 복제 콘텐츠의 분산 네트워크를 만들기 위해 사용될 수 있다

#### 콘텐츠 라우터

![](https://images.velog.io/images/gth1123/post/adf5923f-1c72-4fe5-849c-44cb0f261587/image.png)

- CDN - 넷플릭스, 라이브러리 등
  - [CDN 위키백과](https://en.wikipedia.org/wiki/Content_delivery_network)
  - [CDN 이란](https://www.akamai.com/ko/our-thinking/cdn/what-is-a-cdn)
- 인터넷 트래픽 조건과 콘텐츠의 종류에 따라 요청을 특정 웹 서버로 유도
- 사용자나 콘텐츠 제공자가 더 높은 성능을 위해 돈을 지불했다면 콘텐츠 라우터는 요청을 가까운 복제 캐시로 전달할 수 있음

#### 트랜스코더

![](https://images.velog.io/images/gth1123/post/e25f70cd-9d65-425e-b3aa-b0c324f1e2af/image.png)

- 콘텐츠를 클라이언트에게 전달하기 전에 본문 포맷을 수정할 수 있다
  - 트랜스코딩
    - e.g.
      - 크기를 줄이기 위해 GIF를 JPG로 변환
      - 텍스트 파일 압축
      - 문서를 외국어로 변환(유튜브 : 외국비디오 제목을 한글로 변환되어 표시 - 유튜브 코리아에서 변환)

#### 익명화(Anonymizer) 프락시

![](https://images.velog.io/images/gth1123/post/6616f138-33e9-4ccf-b354-0bd9d9b22539/image.png)

- 신원을 식별할 수 있는 특성들을 제거함으로써 개인 정보 보호와 익명성 보장
  - e.g., IP주소, From 헤더, Referer헤더, 쿠키, URI 세션 아이디, 사용자의 OS 종류

### 사진 출처

- HTTP 완벽가이드
- https://velog.io/@code_newb/%ED%94%84%EB%9D%BD%EC%8B%9C
- https://www.slideshare.net/HyeonSeokChoi/http-6
