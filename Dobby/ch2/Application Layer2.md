## 네트워크 지연

access link ( bandWidth ) 사용률이 80% 이상이 되면 지연이 발생하기 시작한다.

- 케이블 확장 공사를 하여 bandWidth를 늘린다.
- web proxy를 둬서 캐싱한다.
- Conditional GET
  If-modified-since <date> 에 따라 서버로부터 데이터를 받을지 결정한다.
  변경되지 않았다면 데이터를 제외한 텍스트만 받고, 변경될 때만 데이터를 받는다.
- 계층화하여 트래픽 분산

<br />

## DNS

도메인 네임 시스템

호스트 네임과 매핑되는 IP가 담겨있는 테이블 시스템

<br />

### DNS가 제공하는 서비스

- 호스트 에일리어싱
  복잡한 호스트 이름을 가진 호스트는 하나 이상의 별명을 가질 수 있다.
  [ 별칭 호스트 이름, 정식 호스트 이름 ]
- 메일 서버 에일리어싱
  기억하기 쉬운 전자메일 별칭 호스트 이름을 사용한다.
- 부하 분산
  인기 있는 사이트는 여러 서버를 가지며 이는 하나의 호스트 이름에 여러 IP 주소를 가진다.
  DNS 데이터베이스는 이 IP 주소 집합을 가지며, 클라이언트가 호스트 이름에 대한 DNS 질의를 하면 서버는 IP 주소 집합 전체를 가지고 순환식으로 응답한다.

<br />

### 데이터를 하나의 DN 서버에서 관리할 경우의 문제

- 먼 거리의 중앙 집중 데이터베이스
  검색 시간이 늘어난다.
- 서버의 고장
  서버가 다운되면 전체가 무너지게 된다.
- 트래픽양
  단일 DNS 서버가 모든 DNS 질의를 처리해야 한다.
- 유지 관리
  단일 네임 서버는 모든 인터넷 호스트에 대한 레코드를 유지해야 한다.

→ **단일 DNS 서버에 있는 중앙 집중 데이터베이스는 확장성이 전혀 없다.**

<br />

### 문제 해결 방법

DN서버를 분산 및 계층화 시킨다.

Root DNS Server - TLD DNS Server - Authoritative DNS Server - Local Name Server

- 네트워크를 운영하는 모든 기관은 보유하고 있는 도메인이 속하는 Authoritative DNS Server를 운영해야 한다.

<br />

### DNS 계층 구조

- Local DN `local name server`

클라이언트와 맞닿아 있는 네임 서버로, 클라이언트가 도메인 네임을 통해 IP 주소를 알아내고자 할 때 가장 먼저 찾게 되는 네임 서버이다.

로컬 네임 서버는 일반적으로 ISP에서 할당해주지만, 공개 DNS 서버를 이용할 수도 있다.

호스트가 ISP에 연결될 때, 그 ISP는 로컬 DNS 서버로부터 IP 주소를 호스트에게 제공한다.

<br />

- Root DN `root name server`

루트 도메인을 관장하는 네임 서버로, 질의에 대해 TLD 네임 서버의 IP 주소를 반환할 수 있다.

<br />

- TLD DN `Top-Level Demain name server`

질의에 대해 TLD의 하위 도메인 네임을 관리하는 네임 서버 주소를 반환할 수 있다.

<br />

- Authoitative DN `authoritative name server`

특정 도메인 영역을 관리하는 네임 서버로, 자신이 관리하는 도메인 영역의 질의에 대해서는 다른 네임 서버에게 떠넘기지 않고 곧바로 답할 수 있는 네임 서버이다.

로컬 네임 서버가 마지막으로 질의하는 네임 서버이다.

<br />

> 호스트가 DNS 질의를 보내면, 이 질의는 먼저 프록시로 동작하는 로컬 DNS 서버에게 전달되고, 그 로컬 DNS는 캐싱된 데이터에 포함되는지 확인한다.
> 포함되면 IP 주소를 호스트에게 전달하고, 포함되지 않으면 질의를 DNS 서버 계층으로 전달한다.

<br />

### 프로토콜

**UDP를 사용한다.**

DNS에서 전송하는 메시지의 크기는 작다. 그래서 유실될 확률이 적으며, 빠르기 때문에 UDP를 사용한다.

<br />

### 로컬 네임 서버가 네임 서버들에게 질의하는 방법

- 재귀적 질의

  ![image](https://github.com/user-attachments/assets/6402886d-8b6a-4653-be5e-5c67700ea33a)

  클라이언트가 로컬 네임 서버에게 도메인 네임을 질의하면, 로컬 네임 서버가 루트 네임 서버에게 질의하고,

  루트 네임 서버가 TLD 네임 서버에게 질의하고, TLD 네임 서버가 책임 네임 서버에게 질의하는 과정을 반복하며

  **최종 응답 결과를 역순으로 전달받는 방식이다.**

<br />

- 반복적 질의

  ![image](https://github.com/user-attachments/assets/921d6706-8a44-4fb6-92f0-ecb89b62823a)

  클라이언트가 로컬 네임 서버에게 IP 주소를 알고 싶은 도메인 네임을 질의 하면,

  로컬 네임 서버는 루트 도메인 서버에게 질의해서 다음으로 질의할 네임 서버의 주소를 응답받고,

  다음으로 TLD 네임 서버에게 질의해서 책임 네임 서버의 주소를 응답받는 과정을 반복하다 최종 응답 결과를 클라이언트에게 알려주는 방식이다.

<br />

### DNS records

DNS 분산 데이터베이스를 구현한 DNS 서버들은 호스트 이름을 IP 주소로 매핑하기 위한 `자원 레코드`를 저장한다.

```
Name, Value, Type, TTL;
```

- TTL: 자원 레코드의 생존기간
- Type
  - A이면 Name - 호스트이름, Value - IP 주소
    즉, A레코드는 특정 도메인에 매핑하는 IP주소(IPv4)를 알려준다.
  - NS(Name Server)면 Name - 도메인, Value - 도메인에 대한 IP 주소를 얻을 수 있는 다음 네임 서버 이름

<br />

DNS는 타입이 `NS`와 `A`인 레코드를 쌍으로 가지고 있다.

질의를 했을 때 NS와 A 레코드를 받으면 질의를 계속하고, A 레코드 하나만 받으면 원하던 IP 주소를 얻은 것이므로 Local DNS에 저장한 후, 클라이언트에게 전달한다.

> Root → TLD → Authotitative

| Name    | Value   | Type | TTL |
| ------- | ------- | ---- | --- |
| .edu    | dns.edu | NS   |     |
| dns.edu | 2.2.2.2 | A    |     |

| Name            | Value           | Type | TTL |
| --------------- | --------------- | ---- | --- |
| hanyang.edu     | dns.hanyang.edu | NS   |     |
| dns.hanyang.edu | 3.3.3.3         | A    |     |

| Name            | Value   | Type | TTL |
| --------------- | ------- | ---- | --- |
| www.hanyang.edu | 4.4.4.4 | A    |     |