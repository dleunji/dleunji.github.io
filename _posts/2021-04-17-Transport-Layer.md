---
Title : "Transport Layer"
layout : post
date : 2021-04-17
category : Network
blog : true
author : dleunji
description : Transport Layer
---

### Summary

- Transport-layer services
- Multiplexing and Demultiplexing
- Connectionless Transport : UDP
- Principles of reliable data transfer
- Connection-Oriented Transport : TCP
- Principles of congestion control
- TCP congestion control

---

### Transport services and Protocols

- 서로 다른 호스트의 app간에 logical communication
- Application Layer보다 아래, Network Layer보다 위
  - Sending : app의 message를 **segment로 break**하여 **network layer에 전송**
  - Receiving : 세그먼트를 받아서 모아 **message로 reassemble**하고 **app layer로 pass**

- 1개 이상의 transport protocol 가능
  - 인터넷은 TCP and UDP



### Transport vs. Network layer

- Network Layer : 호스트 간의 logical communication
- Transport Layer : 프로세스 간의 logical communication
- Household analogy



### Multiplexing/Demultiplexing

- **Multiplexing** at sender
  - **Gathering**
  - 여러 application 프로세스를 통해 데이터를 모아서 transport header에 추가한다.
- **Demultiplexing** at receiver
  - **Delivering**
  - header를 이용해 received data를 올바른 socket에 전달한다.

### How demultiplexing works

- Host receives IP datagrams
  - 각 **datagram**은 **src / dest IP address**  가진다.
  - 각 datagram은 하나의 transport layer segment 가진다.
  - 각 **segment**에는 **src / dest port #** 가진다.
- Host는 올바른 socket으로 segment를 보내기 위해 IP address /port # 사용

![segment](https://user-images.githubusercontent.com/46207836/115106903-8207ce00-9fa2-11eb-859b-d0129394e791.png)

### Connectionless Demultiplexing(UDP)

- 생성된 socket은 host-local port #를 가진다.
- UDP socket으로 보낼 datagram을 만들 때, dest IP address / port # 명시
- Host가 UDP segment를 받으면 port #에 해당하는 socket으로 UDP segment 전달
- src port#는 다르지만 **같은 dest port #**를 가진 IP datagram은 **같은 socket**으로 보내진다.

### Connection-Oriented Demultiplexing(TCP)

- TCP socket은 4개의 tuple로 구분
  - **src IP address**
  - **src port #**
  - **dest IP address**
  - **dest port #**
- Receiver는 segment를 알맞은 socket으로 보내기 위해 4가지 정보 모두 활용
- Server Host는 동시다발적인 TCP socket 지원
  - 모든 socket이 4가지 조건으로 구분되기 때문
- Web server는 각 연결된 **Client마다** 다른 socket을 가질 수 있다.
  - Non persistent HTTP는 각 request마다 다른 socket을 가진다.



### UDP

- Transport layer라 **segment** 이야기할 줄 알았는데, 이름이 User **Datagram** Protol이라 혼란스러울 수도...
- no frills
- Bare bones
- can be lost
- Or delivered out of order to app
- `Connectionless`
  - no handshaking between UDP sender and receiver
  - each UDP segment handled independently of others
- 사용 예
  - **streaming multimedia apps(loss tolerant, rate sensitive)**
  - **DNS**
  - **SNMP**
- Reliable transfer over UDP
  - add reliability at application layer

- UDP 헤더에는 length와 checksum 존재
  - **Checksum : detect errors**

![udp](https://user-images.githubusercontent.com/46207836/115106927-b3809980-9fa2-11eb-92c4-313e9776dbfc.png)

- 왜 UDP를 선택하는가
  - No connection establishment
  - Simple : no connection state at sender, receiver
  - Small header size
  - No congestion control 
- Checksum 연산 
  - ✰✰ 계산 시험 출제 예상 ✰✰
  - Checksum에 쓰이는 헤더 및 데이터를 16비트 단위로 분할하여 비트 합을 구한 뒤, 이에 대한 1의 보수를 취해 계산된다
  - 만약 비트 합 중 carry 발생 시 wrap around 적용

### Principles of Reliable Data Transfer

- **✰✰ 계산 시험 출제 예상 ✰✰**
- TCP와 UDT가 구분되는 가장 큰 특징
- 하위계층에서 상위계층으로 데이터를 전송할 때 전송된 데이터가 손상되거나 손실되지 않게 보장하는 개념
- TCP의 안정적인 데이터 전송
  - Application Layer의 sender에서 데이터를 보내고, Transport Layer에서 TCP를 제공하고 Application Layer의 Receiver에 도착한다.

<img width="355" alt="스크린샷 2021-04-17 오후 5 48 21" src="https://user-images.githubusercontent.com/46207836/115107421-b29d3700-9fa5-11eb-9b5c-0b7afed4f9e1.png">

![rdt](https://user-images.githubusercontent.com/46207836/115107393-97322c00-9fa5-11eb-84d7-f329e7a192e0.png)

- Unidirectional 경우만 고려
- FSM으로 표현

### rdt 1.0 : reliable transfer over a reliable channel

- channel에 어떠한 에러도 없다고 가정
- 상태가 오직 하나

![rdt1 0](https://user-images.githubusercontent.com/46207836/115107814-145ea080-9fa8-11eb-8e9e-2ff15f14f639.png)

### rdt 2.0 : channel with bit errors

- channel이 플립을 일으킬 수 있다고 가정
- 이 때 에러 처리에 대해 생각하고 설계
- **ack** : 잘받았다! / **nak** : 못 받았다!
- 데이터 메세지는 아니고 정보에 관한 메세지를 하나 더 보낸다
- sender retransmits packet on receipt of NAK
- 새로운 기능
  - error detection
  - feedback : control msgs(ACK, NAK) from receiver to sender

![rdt2 0](https://user-images.githubusercontent.com/46207836/115110124-bf755700-9fb4-11eb-9c49-66c21bd4e422.png)

- 대신 문제점 발생!
  - 만약 ACK, NAK가 corrupted?
  - sender는 receiver가 이에 대해 아는지 모름
  - duplicate의 가능성으로 재전송 어려움
- 해결 방법
  - 만약 ACK, NAK가 발생하면 재전송하되, 각 패킷에 **sequence** 추가
  - 그러면 receiver은 duplicate 버리기 가능

### rdt 2.1 : sender/ receiver handles garbled ACK, NAKs

#### Sender

![image](https://user-images.githubusercontent.com/46207836/115111044-56441280-9fb9-11eb-8624-dba0805a3eb6.png)

- Garbled ack/naks가 오면 초기상태로 가고 0번 패킷이 내려오기를 기다린다.

- rdt_send(data) 이벤트가 일어나면 패킷을 만들고(make_pkt(0, data, checksum))

  시퀀스 0을 넣어서 전송하고 state를 전환하여 ack/nak를 기다린다.

- 이를 receiver에서 받고 corrupted거나 nak이 왔으면 이전에 보낸 데이터를 다시 보내고 받았는데 ack, nak이 corrupted가 아니며 ack이면 다음 순서인 1번 패킷을 기다리는 state가 된다.

- sender는 1번을 넣어서 보내고 nak이면 다시 보내고, ack이면 다시 0번 상태가 된다.

#### Receiver

![image](https://user-images.githubusercontent.com/46207836/115110974-fe0d1080-9fb8-11eb-9f1c-98fee72ae980.png)

- receiver도 초기상태가 먼저 되고, 패킷을 받는다
- corrupted도 아니고 받은 패킷의 시퀀스 == 내가 기다리는 시퀀스, ack를 보내고 다음 시퀀스를 기다리는 상태로 전환된다
- 받은 패킷의 시퀀스 != 내가 기다리는 시퀀스, ack는 보내고 상태는 안 바꾼다
- 또 문제점 발생!
  - 0,1로도 충분하지 않음
  - TCP는 시퀀스 넘버가 충분히 많다.
  - ack하고 nak을 받는 부분을 계속 확인해야하고
  - 방금 0을 받았는지 1을 받았는지도 계속 확인
  - receiver부분에서도 동일한 패킷인지를 체크하고 ack/nak 잘 보냈는지 확인할 수 없다는 단점

### rdt 2.2 : a NAK-free protocol

- nak 대신에 ack1, ack0으로 바꿔서 설계

#### Sender

![img](https://blog.kakaocdn.net/dn/8muOi/btqDogZNciy/SEdtG8nX1cCkH9mJjkNqE1/img.png)

- 초기 상태는 0을 기다리는 상태
- 데이터를 보내는 이벤트가 일어나고, 시퀀스 0을 넣어서 패킷 전송(내려 보냄)
- 그 다음 0이 있는 ack가 오기를 기다림
- 만약 1이 있는 ack가 오면 nak라고 간주

#### Receiver

![img](https://blog.kakaocdn.net/dn/bRYRAE/btqDsbQdR77/bj5EXIQlF9hUKXwIAsJzBK/img.png)

- 도착한 패킷의 시퀀스도 일치하고 에러도 없다면 application layer에 보내고 network layer에서 ack1을 보낸다
- 시퀀스가 0인 패킷을 기다리는데, 1인 패킷이 오면 ack1보냄

### rdt 3.0 : channels with errors and loss

New assumption

- Underlying channel can also lose packets(data, ACKs)
- 재전송을 위해 checksum과 sequence #가 필요하다

Approach

- Sender waits "reasonable" amount of time for ACK
- 즉, sender가 데이터를 전송하고 합리적인 시간만큼 기다린 뒤 ack가 없으면 다시 보냄
- if pkt (or ACK) just delayed(not lost):
  - Retransmission will be duplicate, but seq #'s already handles this
  - Receiver must specify seq # of packet being ACKed
- 결국 타이머가 필요

![image](https://user-images.githubusercontent.com/46207836/115112387-424fdf00-9fc0-11eb-9b9b-3973e15fba5a.png)

- 초기 상태에서 데이터를 보낼 때 seq 0을 넣어서 보내고
- Network layer에서 send하고 타이머를 시작한다
- 그 상태에서 ack0을 기다리는 상태로 전환
- receiver에서 잘 받았다고 왔는데 corrupted 되거나 ack0 대신 ack1이 오면 가만히 있다가
- 타임아웃되면 다시 보내고 타이머를 켠다
- 다시 받았는데 원하는 ack0이 오면 타이머 스탑하고 다음 상태로 넘어간다

- rdt 3.0 is correct, but performance stinks (<u>계산 문제</u>)

### rdt 3.0 : stop-and-wait operation

- Stop-and-wait : sender sends one packet, then waits for receiver response

![s-a-w](https://user-images.githubusercontent.com/46207836/115113025-24d04480-9fc3-11eb-9412-2d86bbe97366.png)

### Pipelined protocols

- 하나의 패킷만 보내면 ACK가 돌아올 때까지 sender는 아무것도 하지 않고 대기하는 시간이 길다.

- Pipelining : sender allows multiple, "in-flight", yet-to-be-acknowledged pkts
- Increased utilization
- Stop-and-wait와 차원이 다르게 seq # 증가, packet # 증가
- Two generic forms of pipelined protocols : ***Go-Back-N***, ***selective repeat***

![pipelining](/Users/ieunji/Downloads/pipelining.png)

### Go-Back-N

- Sender can have up to N unacked packets in pipeline
- **Window** : 한 번에 얼만큼의 packet을 보낼 것인가
- Window size만큼 feedback을 받지 않고 한 번에 packet을 보낸다
- Receiver only sends **cumulative ack**
  - doesn't ack packet if there's a gap
  - 만약 ack11이라고 받으면 11번 packet까지 완벽하게 받았다는 뜻
- Sender has timer for oldest unacked packet
  - When timer expires, retransmit all unacked packets
- Receiver는 자신이 받아야할 seq #의 packet만 계속 기다린다.
- 만약 받아야할 seq #의 packet이 안오면 무조건 버린다
  - e.g. receiver가 pck0을 받으면 ack0을 주고 1을 기다린다.
  - 만약 pck2가 먼저오면 버리고 "0번까지 제대로 받았다"는 의미로 ack0을 보낸다.
  - pck1을 받으면 1을 기다렸으니 ack1을 보내고, pck2가 아닌 다른 pck가 오면 모두 버리고 ack1을 보낸다.
- 만약 window size = 4
  - 0,1,2,3 packet을 보내고, receiver가 0번 packet을 받고, ack0을 주면
  - sender는 ack0이 왔으니 packet4번을 보낸다
  - receiver가 1번 packet을 받았으면 ack1을 보내고
  - sender는 ack1을 받았으니 packet 5를 보낸다
  - 이런 식으로 진행되다가, 6번 packet이 유실되어 receiver가 받지 못했다면
  - window는 6,7,8,9를 가리키고 있고, receiver가 7,8,9 packet을 받아도 **6번을 받지 못했기 때문에 ack5를 보낸다.**
  - 그리고 6번 packet에 대한 timeout이 발생하면서 sender는 6,7,8,9번 packet을 다시 보내고, receiver는 ack5에서 6,7,8,9를 잘 받았기 때문에 ack 6,7,8,9를 순차적으로 보낸다.

### Selective Repeat

- Go-Back-N 방식에서 하나의 packet만 문제가 일어나도 window size만큼의 모든 packet을 다 전송하는 것이 비효율적이어서 이를 보완한 방식
- **유실된 packet만 selective하게 재전송**하는 방식
- ACK n = N번 패킷만 잘 받았다 (N-1번은 잘받았는지랑 상관없이)
- 즉 순서에 맞게 들어오지 않은 packet이어도 receiver는 해당 packet을 버리는 것이 아니라 buffer에 저장해두고 있어야 한다.
- 그리고 window에 해당하는 packet을 모두 받으면 window를 그제서야 이동



### Dilemma..

만약 **sequence number를 {0,1,2,3}** 이렇게 4개 사용하고, **window size가 3**이라면

e.g. 

- sender가 packet 0 ,1,2를 보낸다.
- receiver는 packet을 받고나서 ack 0,1,2를 보낸다.
- 그런데 **모든 ack가 유실**된 경우 **sender는 timeout에 의해서 packet 0 번을 다시 보낸다.**
- 그런데 receiver는 내가 받을 3번 packet의 다음 packet인 0번 packet이라고 생각해서 이를 buffer에 저장할 수 있다.(사실 이미 받은 건데..)
- 해당 문제의 해결방법은 sequence number를 늘리는 것인데, 무작정 늘릴 수 없고 window size와 어떤 관계로 문제를 일으키지 않는 선에서 최소한으로 늘릴 수 있을까?
- 그래서 window size와 sequence number은 관련이 있다.
- `Window size * 2 + 1 = sequnce number`



### TCP : Overview

- **Point-to-point**:

  - 1 sender, 1 receiver (1:1)

- **Reliable, in-order byte stream**

  - no message boundaries

- **Pipelined**:

  - Window
  - TCP congestion and flow control set window size

- Full duplex data

  - 서로가 sender이자, receiver이기 때문에
  - **Bi-directional data flow in same connection**
  - **MSS** : maximum **segment** size(TCP가 한 번에 보낼 수 있는 최대의 데이터 양)
  - 즉 packet의 기본단위는 segment

- **Connection-oriented**:

  - **Handshaking (exchange of control msgs) inits sender**, receiver state before data exchange

- **Flow controlled**

  - Receiver buffer의 남은 size만큼 sender가 데이터를 보낸다.
  - Sender will not overwhelm receiver

  ![tcp](https://user-images.githubusercontent.com/46207836/115119417-3e818400-9fe3-11eb-9a9c-1667ffc0f27f.png)

- source port# : 16bit
- dest port# : 16bit
  - 그래서 port# 0 ~ 약 65000
- Seq # , ACK #

### TCP segment structure

- 외울 필요 없고 src port # / dest port #

- Packet header에서 가장 중요한 정보는 Sequence & ACK number

- **Sequence number**

  - 데이터의 첫번째 byte 번호
  - **Byte** stream number of **first** byte in segment's data
  - e.g. 100개의 연속된 byte를 보내는데, seq #가 42번이면 **42번째** 데이터를 보내는 것

- **Acknowledgements**

  - Sequence number of **next byte expected** from other side cumulative ACK
  - e.g. ACK79이면 78번까지 잘 받았고 79번 데이터를 기다린다는 뜻

- 예를 들어 5000byte 짜리 파일을 전송하는 경우를 살펴보자. 이 때의 MSS가 100byte라고 생각하면 TCP는 5000 byte짜리 파일을 전송하기 위해 50개의 세그먼트를 생성하게 될 것이다. (5000 / 100 = 50). 이런 경우 TCP는 각 세그먼트의 Sequence Number Field에 Byte 기준 번호를 붙인다. 즉, 첫 번째 세그먼트의 Sequence number가 0이라 가정하면 두 번째 세그먼트는 100번, 세번째 세그먼트는 200번의 식으로 Sequence number가 설정된다.

  그렇다면 Ack 번호는 어떠한가? Ack번호의 경우에는 Sequence number보다 약간 복잡한데, 그것은 말 그대로 받은 것에 대한 응답이기 때문이다. 기본적으로 Ack번호는 수신자의 입장에서 송신자로부터 앞으로 받아야할 다음 데이터의 Sequence number이다. 말이 복잡한데, 예를 들어보자. 위에서 든 예에서 만일 송신자가 첫 번째 세그먼트를 보냈다고 생각해보자. 수신자가 데이터를 받으면 그 데이터의 Sequence number는 0번일 것이고 MSS가 100 byte이기 때문에 0번째 byte부터 99번째 byte까지의 데이터가 도착했을 것이다. 그러면 수신자는 자신이 99번째 byte까지 잘 받았고, 앞으로 100번째 byte로 시작되는 세그먼트를 받기 원한다는 표시로 Ack번호를 100으로 붙여 응답하는 것이다.

   이것은 송신자에게 있어서 수신자의 상태에 대해 부가적인 정보를 주는데, 이 정보는 자신이 어떤 데이터를 보내야할지, 즉 새로운 데이터 스트림을 줘야할지 아니면 이미 보낸 데이터를 재전송해야할지를 결정할 수 있는 중요한 것이다.

  이 점을 이해하기 위해 또 다른 예를 들어보자. 위의 파일 전송에서 송신자가 순서대로 3개의 세그먼트를 보냈다고 생각해보자. 즉 Sequence number 0, 100, 200번의 3개의 세그먼트를 통해 300byte를 전송한 것이다. 그런데 첫번째 세그먼트와 세번째 세그먼트는 수신자에게 잘 도착했는데 두 번째 세그먼트가 네트웍에서 손실되었다고 생각했을 때 수신자는 송신자에게 어떻게 응답해야 하는가?

  수신자의 입장에서 생각해 보면 현재 두 개의 세그먼트를 받았다. Sequence number 0번의 0~99번째까지의 100byte와 Sequence number 200번의 200 ~ 299번째까지의 100byte이다. 이 때 수신자는 중간의 100 ~ 199번째의 데이터를 기대한다는 의미로 Ack number 100으로 응답할 수 있다. 그러면 송신자는 수신자가 두 번째 세그먼트를 못받았다는 것을 알고 다시 재전송할 수 있다.

- 만약 "computer"이라는 메시지를 보내면(8byte)

  - A : seq# = 42 / ACK # = 79 / data = "computer"(42번째 데이터를 보낼게)
  - B : seq# = 79 / ACK# = 50 / data = "computer"(42 + 8 50번째 데이터 보내줘)
  - computer → c(42), o(43), ..., r(49)
  - B가 보내는 seq#는 B가 만드는 것이고, 그 번호가 A가 보내는 ACK 번호를 결정한다.
  - 만약 B가 seq# = 40인 메시지를 1byte 보내면, A의 ACK # = 41(41번째 데이터 보내줘)



### TCP round trip time, timeout

Q. How to set TCP timeout value?

	- longer than RTT
	- RTT는 다 다르다

- 너무 적으면 premature timeout → 불필요한 retransmission
- 너무 길면 segment loss에 대해 slow reaction

Q. How to estimate RTT?

- SampleRTT로 측정
- 다 다르다
- `EstimatedRTT = (1- a) * EstimatedRTT + a * SampleRTT`(사실 알파)
- typically `a = 0.125`
- `Timeout interval = EstimatedRTT + safety margin`
  - EstimatedRTT의 편차가 클수록 safety margin이 커진다.
- Estimate SampleRTT의 분산 from Estimated RTT = safety margin
  - `DevRTT = (1-B) * DevRTT + B * |SampleRTT - EstimatedRTT|`
  - Typically `B = 0.25`
- `Time Interval = EstimatedRTT + 4 * DevRTT`



### TCP reliable data transfer

- TCP는 비신뢰적인 인터넷 네트워크 계층(IP)의 상위 계층에서 신뢰적인 데이터 전달(RDT) 서비스 제공
  - Pipelined segments
  - Cumulative acks
  - Single retransmission timer
- retransmissions triggered by
  - Timeout events
  - duplicate acks



### TCP sender events

- 상위 계층으로부터 수신된 데이터
  - 데이터를 받았으므로 상위계층으로부터 데이터 전송 요청이 온다
    - Sequence #를 가진 세그먼트 생성. 
    - 이 때 seq #는 첫 번째 바이트의 byte stream 숫자
    - 타이머가 실행되지 않고 있으면 타이머 시작
    - 타이머의 만료주기는 Timeout Interval로 계산
- Time out
  - 타임아웃에 의해 세그먼트가 재전송되고, 전송했으나 ACK 받지 않은 가장 오래된 세그먼트에 대해 타이머가 재시작된다.
- ACK 수신
  - 이전에 ACK 받지 않은 세그먼트의 ACK이면 해당 세그먼트를 ACK 응답된 세그먼트로 표시
  - 즉 윈도우 크기 조정
  - 아직 ACK 받지 못한 세그먼트들이 존재한다면 타이머 시작

```c
NextSeqNum = InitialSeqNum
SendBase = InitialSeqNum   //송신했으나 ACK되지 않은 가장 오래된 순서 번호
 
loop(forever) {
	switch(event) {
		event : data received from application above // 상위 계층으로부터 수신된 데이터
			create TCP segment with sequence number NextSeqNum // 다음 순서번호를 가진 TCP 세그먼트 생성
			if(timer currently not running) { // 아직 타이머가 시작하지 않았다면
    	start timer //타이머 시작
    }
    pass segment to IP // 생성한 세그먼트 목적지 IP주소로 전송
    NextSeqNum = NextSeqNum + length(data) // 전송할 세그먼트의 순서번호 변경

		event : timer timeout // 타임아웃
			retransmit not-yet-acknowledged segment with smallest sequence 					number
			start timer
 
		event : ACK received, with ACK field value of y //y번의 ACK를 수신한 경		우
			if (y > SendBase) { // 누적된 ACK가 도착한 경우(새 데이터가 온 경우)
				SendBase = y;
				if (there are currently not-yet-acknowledged segments) { //아직 				ACK받지 않은 데이터가 남은 경우
    			start timer //타이머 시작
			}
		}
	}
}
```

### TCP Receiver Events

![event at receiver](https://user-images.githubusercontent.com/46207836/115130275-6398e580-a029-11eb-82b7-2f4bed109446.png)



### TCP fast retransmit

- If sender receives 3 ACKs for same data(triple duplicate ACKs)
- Resend unacked segment with smallest seq #
  - Likely that unacked segment lost, so don't wait for timeout

![fastransmit](https://user-images.githubusercontent.com/46207836/115130313-cee2b780-a029-11eb-8e19-09b04ed139cc.png)

- 왜 굳이 3개일까
  - To detect delayed(not lost) packet & avoid unnecessary ret

### TCP flow control

- TCP 연결의 end host들은 연결에 대한 개별 recv 버퍼를 설정한다.

- Receiver buffer의 남은 size만큼 sender가 데이터를 보낸다. → 데이터 보내는 속도와 받는 속도를 같게 조절

- receiver controls sender, so sender won't overflow receiver's buffer by transmitting too much, too fast.

![flowcontrol](https://user-images.githubusercontent.com/46207836/115130342-1a956100-a02a-11eb-8b6e-cbd0b37052e5.png)

- Receiver "advertises" **free buffer space** by including ***rwnd* value in TCP header** of receiver-to-sender segments
  - ***RcvBuffer*** size set via socket options (typical default is 4096 bytes)
  - many operating systems autoadjsut ***RcvBuffer***
- Sender limits amount of unacked("in-flight") data to receiver's rend value
- 최종목표 : receive buffer의 overflow 방지

![buffer](https://user-images.githubusercontent.com/46207836/115130780-4403bc00-a02d-11eb-99e3-bc1204761328.png)

- `LastByteRead` : 호스트 B의 애플리케이션 프로세스에 의해 버퍼로부터 읽힌 데이터 스트림의 마지막 바이트 수
- `LastByteRcvd` : 호스트 B에서 네트워크로 도착하여 수신 버퍼에 저장된 데이터 스트림의 마지막 바이트 수
- `LastByteRcvd` - `LastByteRead` <= `RcvBuffer`
- `rwnd`는 버퍼의 여유 공간
- `rwnd` = `RcvBuffer` - (`LastByteRcvd` - `LastByteRead`)

### Connection Management

- Before exchanging data, sender/receiver *"**handshake**"*
- Agree on connection parameters

- 2-way handshake
  - variable delays
  - retransmitted messages
  - Message reordering
  - can't see other side
- TCP 3-way handshake

![3wayhandshake](https://user-images.githubusercontent.com/46207836/115130832-dd32d280-a02d-11eb-82ad-59385cbcb8f0.png)

### TCP : closing a connection

- Client, server each close their side of connection
- TCP는 full duplex 통신을 하므로 Client의 send/recv buffer, Server의 send/recv buffer을 각각 닫아줘야 한다.
  - send TCP segment with FIN bit = 1
- Respond to received FIN and ACK
  - On receiving FIN, ACK can be combined with own FIN
- Simultaneous FIN exchanges can be handled

![tcpclose](https://user-images.githubusercontent.com/46207836/115131005-68f92e80-a02f-11eb-9135-b1bd084714bb.png)

- 클라이언트의 send buffer 종료. 여전히 recv는 가능
- 서버가 이 세그먼트를 수신하면 서버는 클라이언트에게 확인 세그먼트를 보내고(여전히 data 주고받기 가능)
- 서버는 FIN=1로 설정된 자신의 종료 세그먼트를 보내고 send buffer 종료요청
- 마지막으로 클라이언트는 서버의 종료 세그먼트에 ACK
  - 클라이언트는 send buffer를 종료하되, ACK loss를 대비하여 retx대비하여 2분 정도 기다린다. 

### Principles of congestion control

*congestion:*

- `rwnd ` 무시하고 `cwnd`만을 고려해 window size W를 어떻게 조정하는지 

- informally "too many sources sending too much data too fast for network to handle"
- Manifestation:
  - lost packets (buffer overflow at routers)
  - long delays (queueing in router buffer)

### Causes and costs of congestion at Network Router

- Link capacity
- Finite Buffer at Router
  - 이상 : known loss
  - 현실 : duplicates → unneeded  retransmissions
- Multi-hop path
  - When packet dropped, any upstream transmission capacity used for that packet was wasted!
  - downstream에 가서 그런 waste가 발생하면 upstream의 라운=터 + 사용한 bandwidth가 모두 낭비
  - Src → dest로 여러 개의 router를 거쳐갈 때 upstream의 네트워크 자원 낭비

### TCP congestion control

- 접근 방식 
  - sender가 transmission rate를 증가하고(즉 window size up!!) 
  - loss가 발생할 때까지 Usable bandwidth 조사 
- How?
  - ACK 받았을 때 increase transmission rate(`cwnd` 높인다)
  - Loss 생겼다고 판단되면 (1. Timeout 2. 3 duplicate ACK)  backs off(`cwnd` 낮춘다)
  - 즉 TCP는 네트워크 상황이 좋다고 판단되면 sending rate를 높여 공격적으로 네트워크 자원을 활용한다(**cwnd is dynamic**)

- **연결로 트래픽을 보내는 sending rate를 제한하는 방법**

  - cwnd로 TCP sender가 네트워크로 트래픽을 전송할 수 있는 비율을 제한

  - ACK가 안된 데이터양은 `cwnd`와 `rwnd`의 최소값을 초과하지 않는다.
  - `LastByteSend - LastByteAcked <= min{cwnd, rend}`

- **TCP sender가 자신과 dest 사이의 경로에 혼잡이 존재하는지 감지하는 방법**

  - Timeout 또는 3 dup ACK 수신이 발생하면 TCP sender에 loss 발생한 것
  - congestion 감지

### TCP control algorithm : AIMD

![aimd](https://user-images.githubusercontent.com/46207836/115133203-4f60e280-a041-11eb-8da9-039ba04ca7ad.png)

- Additive increase : loss가 감지되기 전까지 매 RTT마다 `cwnd`를 1MSS씩 증가한다
- Multiplicative decrease : loss 감지되면 `cwnd` 절반으로 줄인다.



### TCP Slow Start

<img width="703" alt="image" src="https://user-images.githubusercontent.com/46207836/115133272-e5950880-a041-11eb-8963-d016193df1a5.png">

- intial rate = 1MSS(`cwnd`) 로 시작해(전송률 : 1MSS/RTT) 한 세그먼트가 한 ACK을 받을 때마다 ACK당 1MSS씩 증가한다. ssthresh에 도달할 때까지...
- loss가 생길 경우 ,ssthresh를 혼잡이 생긴 시점의 `cwnd`값의 절반으로 정한다.
- `cwnd`값이 `ssthresh`에 도달한다면 slow start를 종료하고 **CA(혼잡회피모드)** 로 전환한다.
- → cwnd이 각 RTT당 1MSS씩 linear하게 증가한다.

- threshold에 가까워지면 linearly increase



### TCP throughput

- avg. TCP thruput as function of window size, RTT?
  - Ignore slow start, assume always data to send
- W : window size(measured in bytes) where loss occurs
  - avg. window size (# in-flight bytes) is 3/4 W
  - Avg. thrust is 3/4 W per RTT

![tcpthroughput](https://user-images.githubusercontent.com/46207836/115135704-5f82bd00-a055-11eb-97bb-0c261604581a.png)

### TCP Futures : TCP over "long , fat pipes"

링크의 에러율을 포함해 throughput을 계산하는 것이 맞다

### TCP Fairness

TCP는 네트워크의 상황을 추측, throughput을 결정하는 bottleneck에서 bandwidth를 얼마나 공정하게 share하는가

<img width="699" alt="image" src="https://user-images.githubusercontent.com/46207836/115136440-d66e8480-a05a-11eb-8c21-aad5dbcec836.png">

- connection1이 connection을 더 먼저 시작했을 수도, cwnd가 훨씬 커 bandwidth를 더 많이 차지하고 있는 상황

- connection2의 cwnd가 점점 증가하면서 파란선을 넘고 loss가 생긴다.

- loss가 생기면 각 cwnd를 loss가 생긴 시점의 반으로 줄인다.
- 결국 두 connection은 R/2만큼의 bandwidth를 쓴다.
- 모두 TCP connection을 사용. 경쟁하는 bottleneck link와 host사이의 # of RTT (hops)가 같다고 가정하면 그 TCP connection들은 경쟁하는 R값을 공평하게 나눠갖는다.



### Explicit Congestion Notification(ECN)

Network-assisted congestion control

- two bits in IP header(ToS field) marked by network router to indicate congestion
- Congestion indication carried to receiving host
- Receiver (seeing congestion indication in IP datagram) sets ECE bit on receiver-to-sender ACK segment to notify sender fo congestion 



---

### 각 계층의 전송단위

- **Application** : message
- **TCP(transport)** : segment
  - header + data
  - application에서 보낸 message가 data에 저장
- **IP(Network)** : packet
  - packet도 header + data
  - data에는 segment의 header와 data가 packet의 data에 속한다.
- **Link** : Frame
  - 위와 같은 방식으로  header + data



- **TCP 요약**

TCP는 UDP와 다르게 네트워크 안의 라우터 버퍼가 오버플로우 되지 않게 하기 위해 congestion control을 한다. congestion control의 목적은 2가지.

1. NW에 가용한 bandwidth를 가능한 많이 써 utilization을 높여 응용의 처리율을 증가시킴
2. NW의 router 버퍼에서 심각한 **지연** 혹은 overflow로 인한 **loss**가 발생하지 않도록 함

congestion control의 방법 중 Tahoe와 Reno 모두 Slow Start 방식으로 시작. Slow Start는 1 MSS로 세그먼트를 보내기 시작해, 가용한 network의 bandwidth를 모두 사용하지 못 할 수 있음. congestion control의 목적 중 하나는 가용한 network의 bandwidth를 가능한 많이 쓰는 것.

ACK을 받을 때마다 하나의 ACK 당 하나의 MSS씩 증가(2배씩 double로 증가), ssthresh까지 증가시 혼잡 회피로 변경하여 linear하게 1 MSS씩 증가. 진행하면서 Timeout값을 점점 증가. 만약 loss가 발생했다고 판단되면, `cwnd` 값을 loss가 생긴 시점의 `cwnd` 값의 절반으로 `cwnd` 를 갱신. Tahoe는 SlowStart를 다시 시작 (1MSS 전송 부터), RENO의 경우 ACK을 한 번 더 받은 후 3 dup ack이 오면 현재 `cwnd` 의 절반으로 `cwnd`를 갱신한 후 혼잡 회피(CA)로 linear하게 1MSS씩 증가 (3 dup ACK은 한 번 ACK 받은 후 3번 더 오면 3 dup ACK이라고 판단

