# HTTP 완벽가이드 10장 정리

#### 10장에서 학습할 내용

- HTTP/2.0을 만들기 시작하게 된 배경
- HTTP/1.1과의 주요 차이점
- 현재까지 알려진 보안이슈
- 책은 HTTP/2.0 9번째 초안, http2-8에 기반하여 작성되었으므로 현 시점의 명세와 다를 수 있음

## 10.1 HTTP/2.0의 등장 배경

HTTP/1.1의 메시지 포맷은 구현의 단순성과 접근성에 주안점을 두고 최적화 되었다.

- 성능은 어느정도 희생
  - 병렬 커넷션이나 파이트라인 커넥션 도입(책 4장 참고)
    - 근본적인 해결책은 되지 못 함

### SPDY

2009년 구글 - 더 빠른 웹을 위한 프로토콜

여러 기능추가

- 헤더를 압축하여 대역폭을 절약
- 하나의 TCP 커넥션에 여러 요청을 동시에 보내 회전 지연을 줄임
- 클라이언트가 요청을 보내지 않아도 서버가 능동적으로 리소스를 푸시하는 기능

이 모두는 회전 지연을 줄이기 위한 것

2012년 HTTP 작업 그룹은 SPDY를 기반으로 HTTP/2.0 프로토콜을 설계하기로 결정
2013년 8번째 초안까지 SPDY와 변경된 점은 헤더를 압축할 때 deflate 알고리즘을 사용하지 않게 되었다는 것 정도(자세한 내용은 10.3.3)

## 10.2 개요

HTTP/2.0은 서버와 클라이언트 사이의 TCP 커넥션 위에서 동작
하나의 커넥션은 여러개의 스트림이 동시에 들어갈 수 있고,
스트림에 요청, 응답이 압축된 헤더와 같이 담기므로,
하나의 커넥션에서 여러개의 요청, 응답을 동시에 처리하는 것 역시 가능

서버 푸시를 도입

## 10.3 HTTP/1.1과의 차이점

### 10.3.1 프레임

HTTP/2.0에서 모든 메세지는 프레임에 담겨 전송 됨

HTTP/2.0 의 10가지 프레임

- DATA, HEADERS, PRIORITY, RST_STREAM, SETTINGS, PUSH_PROMISE, PING, GOAWAY, WINDOW_UPDATE, CONTINUATION
- 페이로드 형식, 내용 <- 프레임의 종류에 따라 다름

### 10.3.2 스트림과 멀티플렉싱

스트림 : HTTP/2.0 커넥션을통해 클라이언트와 서버 사이에서 교환되는 프레임들의 독립된 양방향 시퀀스

- 통로(커넥션)에서 왕복하는 배(스트림)같은 느낌 그 배(스트림)를 통해 전달하는 것이 프레임

HTTP/1.1에서는 한 TCP 커넥션을 통해 요청을 보냈을 때, 그 응답이 온 뒤에 다시 요청을 보낼 수 있었음

- 그래서 여러개의 TCP 커넥션을 만들어서 동시에 여러 개의 요청을 보내는 방법을 사용
  - p. 103 최신 브라우저는 6~8개의 병렬 커넥션을 지원

그러나 HTTP/2.0에선 하나의 커넥션에 여러 개의 스트림이 동시에 열릴 수 있음

- 통로하나에 동시에 여러개의 배(스트림)가 각각 하나의 짐(프레임)을 싣고 이동
- HTTP 1.1에서 하나의 응답이 올때 까지 기다리거나 여러개의 커넥션을 만들어야 되는 문제를 해결 함

뿐만 아니라 스트림도 우선순위를 가질 수 있음

- 대역폭이 충분치 않다면 우선순위 설정 가능
  - e.g., 이미지보단 HTML요청에 더 높은 우선순위 부여
  - 그러나 우선순위에 따르는 것은 의무사항이 아니기 때문에 요청이 우선순위대로 처리된다는 보장은 없다.

스트림 식별자 특징

- 31비트 무부호 정수 - 고유한 식별자를 갖음
- 클라이언트에 초기화 : 홀수
- 서버에 의해 초기화 : 짝수
- 새로 만들어지는 스트림 직별자 : 이전에 만들어졌거나 예약된 스트림의 식별자보다 커야 함
  - 이 규칙을 어기는 식별자를 받았다면 에러코드가 PROTOCOL_ERROR인 커넥션 에러로 응답
- HTTP2.0커넥션에서 한번 사용한 스트림 식별자는 다시 사용할 수 없음
  - 한번 연결된 스트림은 요청 응답이 한번 주고 받을 때 마다 새로운 식별자를 할당해서 그 스트림을 재활용 하는 것 같다.
- 커넥션을 오래 맺어서 식별자가 고갈되면 커넥션을 다시 맺음

### 10.3.3 헤더 압축

HTTP/2.0에선 헤더가 압축 됨

- 회전 지연, 대역폭에 실질적인 영향(성능 향상)
- 보안문제 때문에 deflate 알고리즘이 아닌 HPACK 명세에 따라 헤더를 압축

### 10.3.4 서버 푸시

HTTP/2.0은 서버가 하나의 요청에 대해 응답으로 여러 개의 리소스를 보낼 수 있음(그 요청에서 요청한 것이 아니더라도)

- HTML 문서 요청 -> 응답 : HTML문서 + HTML문서가 링크하는 이미지, CSS파일, JS파일 등
  - 어차피 여러번 요청하게 될 것을 미리 보내줌으로써 트래픽과 회전지연을 줄여줌
  - [ ] 이건 1.1에선 그럼 HTML만 먼저 받고 그 뒤에 다시 요청을 보내서 CSS, JS 파일을 받나?
  - 하나의 요청에 미리 여러개의 응답을 보낼 경우엔 서버에서 PUSH_PROMISE 프레임을 보내서 서버가 미리 보내려고 하는 것을 클라이언트에서 또 요청하는 것을 피하기 위함

서버 푸시 주의 사항

- 프락시로 인해 추가 리소스가 전달이 되지 않을 수 있고 요청하지 않은 추가리소스가 전달될 수 있음
- 서버 푸시 가능한 것들 : 안전하고 캐시가능하고 본문을 포함하지 않은 요청에 대해
  - [ ] 본문을 포함하지 않은 요청이란?
- 푸시할 리소스는 클라이언트가 명시적으로 보낸 요청과 연관 된 것이어야 함
- 서버가 보내는 PUSH_PROMISE 프레임은 원 요청을 위해 만들어진 스트림을 통해 보내짐
  - [ ] 그러면 그 스트림은 유지되고 스트림 식별자만 새로 할당되어서 다시 서버 푸시를 위한 응답이 전달 되는 건가?
- 클라이언트는 반드시 서버가 푸시한 리소스를 동일 출처 정책(SOP, Same-origin policy)에 따라 검사 해야 한다.
- 서버 푸시를 끄고 싶다면 SETTING_ENABLE_PUSH을 0으로 설정하면 된다.
  - 헤더에서 설정하는 건가?

## 10.4 알려진 보안 이슈

### 10.4.1 중개자 캡슐화 공격(Intermediary Encapsulation Attacks)

HTTP/2.0 메세지를 중간의 프락시가 HTTP/1.1 메세지로 변환할 때 메세지의 의미가 변질될 가능성이 있다.

### 10.4.2 긴 커넥션 유지로 인한 개인정보 누출 우려

커넥션이 오래 유지되는 경우, 이전에 사용했던 사람의 커넥션이 브라우저에 남아서 개인정보의 유출에 악용될 가능성이 있다.

# HTTP/1.1 VS 2.0 VS 3.0

#### HTTP/2 에서 하나의 TCP에 여러개의 스트림이 동시에 열릴 수 있음

- 성능 향상

#### HTTP/2.0 부터는 body가 이진 데이터로 전송됨(1.1에선 문자열로 이루어져 있음)

![](https://images.velog.io/images/gth1123/post/45a4ba78-49ff-44c9-b8b0-cc98cf673d7d/image.png)

#### 1.1에서 pipelining 기술을 도입하였지만, 여전히 HOL(Head-of-line) Blocking 문제가 있음

- HOL Blocking : 패킷이 순서대로 도착해야 하므로, 패킷이 도착할 때 까지 그 이후의 패킷은 전송되지 못하는 것
- 1개의 TCP 연결당 1개의 스트림만 이용
- 하나의 TCP에 여러개의 스트림이 동시에 열게 함으로써 이 문제를 해결
- TCP 연결이 1개 이므로 매번 TCP 연결을 할때 마다 3-way-handshake로 인한 오버헤드가 없음
  - 오버헤드 : 처리시간 외의 추가적인 지연시간

## HTTP/3

HTTP/2를 도입하면서 많은 문제를 해결했지만 TCP 프로토콜 자체의 한계 때문에 더 이상 성능 개선에도 한계가 오게 됨

-> UDP를 사용

### QUIC : Quick UDP Internet Connections

- TCP/IP 기반의 애플리케이션 레이어 프로토콜인 HTTP를 QUIC 위에 얹었다
- HTTP over QUIC : HQ
- HTTP/2에 있는 프레임, 스트림, 메시지 구조와 기술들은 그대로 HTTP/3으로 승계되었고 명칭만 HQframe, QPACK 등으로 변경
  ![](https://images.velog.io/images/gth1123/post/ae80b64f-eaa9-4cd3-bd94-f823d6909657/image.png)
- HTTP/2, HTTP/3의 구조

#### HTTP/3 서버 - 클라이언트

HTTP/2와 마찬가지로 HTTP/3를 사용하려면, 클라이언트(주로 웹 브라우저)와 서버(웹 서버) 모두 HTTP/3 프로토콜을 지원해야 합니다.

### TCP vs UDP

|                  | TCP                  | UDP                     |
| ---------------- | -------------------- | ----------------------- |
| 연결 방식        | 연결 지향형 프로토콜 | 비 연결 지향형 프로토콜 |
| 전송 순서        | 보장                 | 보장하지 않음           |
| 신뢰성           | 높음                 | 낮음                    |
| 전송속도(상대적) | 느림                 | 빠름                    |
| 혼잡제어         | O                    | X                       |
| 헤더 크기        | 20바이트             | 8 바이트                |

### 1. 딜레이 감소

![](https://images.velog.io/images/gth1123/post/13d5b73d-eb8e-47de-a62d-d219609d411f/image.png)

### 2. 네트워크 스위칭 속도 개선

매 요청마다 클라이언트 - 서버의 아이피가 필요한 것이 아니고, QUIC는 Unique connection ID를 사용해서 모든 패킷이 잘 구별될 수 있도록 한다.
따라서 휴대폰으로 인터넷을 할 때, 중간에 와이파이에서 LTE로 변경해도 스트림이 계속 유지가 된다.
(TCP의 경우에는 처음부터 다시 데이터를 받아야 함)

## 고찰

- 프론트엔드 웹앱을 개발하는 입장에선 당장 HTTP/2 또는 3을 적용하기 위해 어떤 코드를 수정할 부분이 있는 것은 없는 것 같다.

### 참고

- https://velog.io/@zzzz465/HTTP1.1-2-3-%EC%9D%98-%EC%B0%A8%EC%9D%B4%EC%A0%90
- [오버헤드](https://velog.io/@mygomi/TIL-70-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D-%EC%9A%A9%EC%96%B4-%EC%98%A4%EB%B2%84%ED%97%A4%EB%93%9C-Overhead)
- https://en.wikipedia.org/wiki/HTTP/3
- [HTTP 3.0 : RFC 9000](https://datatracker.ietf.org/doc/html/rfc9000)
- https://bruce.pllip.com/HTTP-3-%EC%A0%81%EC%9A%A9
- https://m.blog.naver.com/sehyunfa/221680799006
