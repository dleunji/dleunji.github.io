---
Title : "Introduction of Network2"
layout : post
date : 2021-04-15
category : Network
blog : true
author : dleunji
description : 기초 네트워크
---

### Summary : 

- Network Core : packet/circuit switching, Internet structure
- Performance : loss, delay, throughput
- protocol layers, service models

---

### Network Core

- Mesh of interconnected **routers**(경로를 탐색하고 패킷을 저장한 뒤 다시 송신하는 역할)
- Packet-Switching : hosts break application-layer messages into packets
  - Forward packets from one router to the next, across links on path from source to destination
  - Each packet transmitted at full capacity

### Packet-Switching : Store-and-Forward

- 1 hop에 L/R sec 소요
- 그리고 모든 패킷은 반드시 목적지까지 송신되기 전에 라우터에 송신되어야 한다.
- 즉 2 hops 필요
- End-End delay = 2 L/R(assuming zero propagation delay)

![image](https://user-images.githubusercontent.com/46207836/114740689-0a108c80-9d85-11eb-8fda-e916a9764a7d.png)

### Packet-Switching : Queueing Delay, Loss 

✰✰ 시험 출제 예상 ✰✰

- Queueing and Loss
  - If arrival rate(in bits) to link exceeds transmission rate of link for a period of time:
    - packets will queue, wait to be transmitted on link
    - Packets can be dropped (lost) if memory (buffer) fills up

![image](https://user-images.githubusercontent.com/46207836/114741407-b5214600-9d85-11eb-87fc-d59f88ae480e.png)



### Two Key Network-core Functions

- **Routing**

  - determines source-destination route taken by packets
  - Routing Algorithms

  <img width="184" alt="image" src="https://user-images.githubusercontent.com/46207836/114743996-32e65100-9d88-11eb-8bfa-17387d1c4ae6.png">

- **Forwarding**

  - move packets from router's input to appropriate router output
  - Select의 문제 "Packet을 어느 파트로 내보낼까"

### Alternative Core - Circuit Switching

- End-End Resources

  - allocated to, reserved for "call" between source & destination:

    - In diagram, each link has four circuits.
    - Call gets 2nd circuit in top link and 1st circuit in right link

    ![Circuit switching](https://user-images.githubusercontent.com/46207836/114744208-675a0d00-9d88-11eb-96dc-27d43191cb69.png)

    - Dedicated resources : no sharing circuit-like performance
    - Circuit segment idle if not used by call (no sharing)
    - commonly used in traditional telephone networks
    - 경로가 딱 하나 (src → dest)이므로 Circuit만 잘 지정하면 된다.

### Circuit Switching: FDM vs TDM

- FDM
  - Frequency Division Multiplexing
  - used in analog system
  - **Bandwidth** is committed to the different sources
  - 한 전송로의 대역폭을 여러 개의 작은 채널로 분할하여 여러 단말기가 동시에 이용하는 방식
- TDM
  - Time Division Multiplexing
  - works with digital signals likewise analog signals
  - **share the timescale** for the various signals 
  - 링크의 높은 대역폭을 여러 연결이 공유할 수 있도록 하는 디지털과정
  - 한 전송로 대역폭을 시간 슬롯으로 나누어 채널에 할당함으로써 몇 개의 채널이 한 전송로의 시간을 분할하여 사용
- 즉 FDM에서는 대역의 일부를 공유한다면, TDM은 시간을 공유한다.

### Packet Switching vs Circuit Switching

✰✰ 시험 출제 예상 ✰✰

`Packet switching allows more users to use network`

참고 : https://gaia.cs.umass.edu/kurose_ross/interactive/ps_versus_cs.php

[장점]

Packet Switching is great for bursty data

	-  resource sharing
	-  Simpler, no call setup

[단점]

excessive congestion possible : packet delay and loss

- Protocols needed for reliable data transfer, congestion control

[해결방법]

Provide circuit-like behavior

- bandwidth guarantees need for audio/video apps
- still an unsolved problem

### Internet Structure

**ISP**가 각기 대응하면 비효율적이다.

**Global ISP**를 제공함으로써 효율적인 access를 가능케 한다.→ Clustering

**IXP**로 **ISP 간**에 인터넷 트래픽을 원활하게 소통할 수 있도록 한다.

**Peering link**는 2개의 네트워크를 연결한다.

**Regional network**는 ISP와 access net 간에 연결하도록 한다.

**Content Provider Network**는 그들만의 네트워크, 서비스를 end user에게 제공한다.

(e.g. Google, Microsoft), **Tier-1 commercial ISP** 능가



### Packet Loss and Delay

Packet Queue에 주목하세요!

- **Loss** : Packet arrival rate to link **exceeds** output link capacity
- **Delay** : Packets queue, **wait for turn**



### Four Sources of Packet Delay

✰✰ 시험 출제 예상 ✰✰

![image](https://user-images.githubusercontent.com/46207836/114877657-70a3b200-9e3a-11eb-92fc-68750ce15549.png)

- Total Nodal delay != Nodal Processing Delay
  - **(Total) Nodal delay** = The sum of all latency delay
  - **(Nodal) Processing Delay** = Every device along the communication path that actually processes / examines the packets 
  - **Queueing Delay**
    - Time waiting at output link for transmission
    - Depends on congestion level of router
    - **La / R** ( packet length * average packet arrival rate / link bandwidth )
    - 0에 가까울 수록 delay 적다는 뜻
  - **Transimission Delay**
    - Output link에서 **push**해서 packet을 밖으로 내보내는 데 걸리는 시간
    - **L / R** ( packet length [bits] / link bandwidth [bps]])
  - **Propagation Delay**
    - 나간 packet이 media **physical**을 타고 **next hop**에 도착할 때까지의 시간
    - **D / S** (length of physical link / propagation speed [m/sec])
- Caravan Analogy ✰✰ 시험 출제 예상 ✰✰
  - delay of nth car = transmission delay * # cars + propagation delay

### Packet Loss

- Queue(buffer) preceding link in buffer has finite capacity
- Packet arriving to full queue dropped (lost)
- lost packet may be retransmitted by previous node, by source end system, or not at all

### Throughput

✰✰ 시험 출제 예상 ✰✰

- Rate (bits/time unit) at which bits transferred **between sender / receiver**
  - Instantaneous : rate at given point in time
  - average : rate over longer period of time
- != Bandwidth
- Throughput = 처리량 = 통신에서 *네트워크* 상의 어떤 노드나 터미널로부터 또 다른 터미널로 전달되는 단위 시간당 디지털 데이터 전송
- Bandwidth = 대역폭
- Bandwidth가 이론상의 이상적인 전송량이고, Throuput은 실제로 전달된 전송량
- **Bottleneck Link** : link on end-end path that contains end-end throughput
  - In practice : Link bandwidth of receiver or sender is often bottleneck

### Protocol "Layers"

- Layers: each layer implements a service
  - via its own internal-layer actions
  - Relying on services provided by layer below

### Why layering?

- Dealing with complex systems:

  - explicit structure allows **identification**, **relationship** of complex system's pieces
  - Modularization eases **maintenance, updating of system**

  

### Internet Protocol Stack

✰✰ 시험 출제 예상 ✰✰

- **Application** : supporting network applications
  - FTP, SMTP, HTTP
- **Transport** : processs-process data transfer
  - TCP, UDP
- **Network** : routing of datagrams from source to destination
  - IP, routing protocols
- **Link** : data transfer between neighboring network elements
  - Ethernet, 802.111(WiFi), PPP 
- **Physical**
  - Bits "on the wire"

### ISO/OSI Reference Model (OSI 7계층)

✰✰ 시험 출제 예상 ✰✰

1. **application**
2. **Presentation** 
   - Allow applications to **interpret meaning of data**
   - e.g. encryption, compression, machine-specific conventions
3. **Session**
   - Synchronization, checkpointing, recovery of data exchange
4. **Transport**
5. **Network**
6. **Link**
7. **Physical**

- Internet stack에는 Presentation, Session 계층 없음

### Encapsulation

- OSI계층모델에서 사용자 데이터가 각 계층을 지난다.
- 하위계층은 상위 계층으로부터 온 정보를 데이터로 취급한다.
- 자신의 계층 특성을 담은 제어정보(주소, 에러제어 등)를 헤더화 시켜 붙이는 일련의 과정

![image](https://user-images.githubusercontent.com/46207836/114888465-393a0300-9e44-11eb-8aaa-b2e074f7e853.png)

