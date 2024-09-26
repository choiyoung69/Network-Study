## 6.7 총정리:웹 페이지 요처엥 대한 처리
지금까지 배운 내용을 정리하여 학생 밥이 이더넷 스위치에 랩톱을 연결하고 웹 페이지를 다운로드 하는 것을 보여준다.

### 6.7.1 시작하기:DHCP, UDP, IP 그리고 이더넷
![](https://velog.velcdn.com/images/choiyoung6609/post/46395eb6-f24c-4560-b310-ea538bd53a25/image.png)

먼저 밥은 이더넷 스위치(라우터에 연결되어 있음)에 랩톱을 연결한다.(이더넷 케이블 이용) 이떄 학교 라우터는 ISP comcast.net에 연결되어 있고, comcast.net은 DNS를 제공하며 DNS 서버는 학교 네트워크가 아닌 컴캐스트 네트워크에 있다고 가정한다. + DHCP 서버는 라우터에서 실행되고 있다고 가정한다.(일반적)

먼저 밥이 처음 네트워크에 랩톱을 연결했을 때 IP 주소가 필요하기 떄문에 **로컬 DHCP 서버에게 DHCP 프로토콜을 실행한다.**

1. 먼저 밥의 OS 에서는 **DHCP 요청 메세지**를 만들어서 목적지 포트 67, 출발지 포트 68를 갖는 **UDP 세그먼트**를 보낸다. 이 세그먼트는 브로드캐스트 IP 목적지 주소와 0.0.0.0의 출발지 IP 주소를 갖는다.

2. DHCP 요청 메세지를 포함한 IP 데이터그램은 **이더넷 프레임**에 들어가서 스위치에 연결된 모든 장치에게 보내진다.(MAC 주소를 FF:FF:FF:FF:FF:FF로 설정된다. 출발지 주소는 밥의 MAC 주소가 된다.

3. 이 이더넷 프레임은 이더넷 스위치에 전송되어 스우치는 자신의 모든 출력 포트에 브로드캐스트한다.

4. 라우터는 브로드캐스트 이더넷 프레임을 수신하고, **역다중화**를 진행하여 DHCP 요청 메세지가 추추된다. 이제 DHCP 서버는 DHCP 요청 메세지를 갖는다.

5. 라우터에서 실행되고 있는 DHCP 서버가 **CIDR** 블록 68.58.2.0/24에 있는 IP 주소를 할당할 수 있다고 하자. 그러면 DHCP 서버가 밥의 랩톱에게 주소를 할당하고 **DHCP ACK 메세지**를 만들어서 보낸다.

6. DHCP ACK 메세지는 스위치로 전송되고,(유니캐스트) 스위치는 **자가학습**으로 밥의 랩톱으로 가는 포트로만 전달한다.

7. 밥의 랩톱이 DHCP ACK 메세지를 추출하면 밥의 DHCP 클라이언트는 자신의 IP 주소와 DNS 서버의 IP 주소를 기록한다. 또한, IP 포워딩 테이블에 디폴트 게이트웨이 주소를 저장한다. 


### 6.7.2 여전히 시작하기 : DNS와 ARP
밥의 웹 브라우저는 www.google.com으로 **HTTP 요청 메세지**를 보내기 위해 **TCP 소켓**을 생성하는 절차를 필요로 한다. 이를 위해서는 www.google.com의 IP 주소를 알아야한다. 이는 **DNS 프로토콜**에서 처리된다. 

1. 밥의 운영체제는 **DNS 질의 메세지**를 생성한다. 이때 DHCP ACK 메세지에 들어있던 DNS 서버의 IP 주소와 출발지 주소(자신의 주소)도 기입된다.

2. 이 질의 메세지는 게이트웨이 라우터로 보내지게 되는데, DHCP ACK 메세지를 통해 게이트웨이 라우터의 IP 주소는 알았지만 MAC 주소를 파악하지 못했기 때문에 **ARP 프로토콜**을 사용한다.

3. 밥의 랩톰은 목표 IP 주소를 포함한 **ARP 질의 메세지**를 생성하여 스위치로 전송된다.(브로드캐스트)

4. 게이트웨이 라우터는 ARP 질의 메세지를 받아서 자신의 MAC 주소를 포함한 **ARP 응답 메세지**를 전달하낟. 

5. 밥이 게이트웨이 MAC 주소를 추출하여 **DNS 질의 메세지**를 보낸다. 이때 목적지 MAC 주소는 게이트웨이 라우터이지만, IP 데이터그램의 IP 목적지 주소는 DNS 서버이다. 

### 6.7.3 여전히 시작하기: DNS 서버로의 인트라 도메인 라우팅
1. 게이트웨이 라우터가 데이터그램을 받으면 **목적지 IP 주소**를 포워딩하여 적합한 링크 계층 프레임에 IP 데이터그램을 넣은 후, 프레임을 해당 링크로 전송한다.

2. 컴캐스트의 라우터에서는 프레임을 추출하여 데이터그램의 목적지 주소와 포워딩 테이블로부터 데이터그램을 DNS 서버로 전달할 출력 인터페이스를 결정한다.(**BGP, RIP< OSPS 등 사용)

3. DNS 질의 가 포함된 IP 데이터그램이 DNS 서버에 도착하면 **IP 자원 레코드**를 찾아 **DNS 응답 메세지**로 ㅁ만들어서 전달된다.

4. 밥의 랩톱은 IP 주소를 추출한다.

### 6.7.4 웹 클라이언트-서버 상호작용: TCP와 HTTP
1. **TCP 소켓**을 사용하기 위해서는 **세 방향 핸드쉐이크**가 수행되어야 한다. 따라서 TCP SYN 세그먼트를 생성하고 목적지 주소와 게이트웨이 라우터의 MAC 주소를 넣은 프레임을 전송한다.

2. TCP SYN 세그먼트를 포함하고 있는 데이터그램은 라우터의 포워딩 테이블을 사용해서 전달된다. 이때 라우터의 포워딩 테이블 엔트리는 **BGP 프로토콜**이 사용되었다.

3. TCP SYN이 www.google.com에 도착하면 환영 소켓으로 역다중화되어 TCP SYNACK 세그먼트로 전달된다.

4. 밥의 이더넷 컨트롤러에 TCP SYNACK 세그먼트가 도착하면 다시 역다중화된다.

5. 밥은 HTTP GET 메세지를 만들어서 소켓으로 보낸다. 이 메세지는 1~3번에서처럼 www.google.com으로 전송된다. 

6. www.google.com에서는 TCP 소켓으로부터 HTTP GET 메세지를 읽고 **HTTP 응답 메세지**를 보낸다.

7. 이 메세지는 밥의 랩톱에 도착하여 마침내 웹 페이지를 출력한다.
