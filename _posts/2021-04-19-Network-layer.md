---
Title : "Network Layer"
layout : post
date : 2021-04-17
category : Network
blog : true
author : dleunji
description : Network Layer
---

#### Summary

- Overview of Network layer
  - Data plane
  - Control plane
- What's inside a router
- IP:Internet Protocol
  - datagram
  - fragmentation
  - IPv4 addressing
  - Network address translation
  - IPv6
- Generalized Forward and SDN
  - match
  - action

---

### Network layer

- Transport segment from sending to receiving **host**
- On Sending side
  - Encapsulates segments into datagrams
- On receiving side
  - Delivers segments to transport layer
- Network layer protocols in **every host, router**
- router examines header fields in all IP datagrams passing through it

### Two key network-layer functions

- 네트워크 레이어의 주요 2가지 기능
- **Forwarding**
  - **move packets** from router's input to appropriate router output
- **Routing**
  - **Determine route** taken by packets from source to destination

### Network layer : data plane, control plane

✰✰ 계산 시험 출제 예상 ✰✰

- **Data plane**
  - Local, **per-router function**
  - Determines how datagram arriving on router input port is forwarded to router output port
- **Control plane**
  - 여러 개의 router를 거쳐 메세지를 주고 받고, end-to-end path 설정
  - 2 control-plane approaches
    - Traditional routing algorithm
    - Software-defined networking(CDN)

### Per-router control plane

- 모든 router 안에 있는 routing algorithm components는 control plane에서 interacting
- Control plane과 data plane이 모두 같은 device에 있다.
- 모든 라우터 안에 control plane이 있다.
- 라우터 간에 메시지 주고 받으며 FIB로 내려보낸다
- **Destination-based forwarding**

### Logically centralized control plane

- A distinct(typically remote) controller(CAs)
- User의 data가 지나가는 data plane들은 physically 모여있음
- Control plane은 remote하게 멀리서 제어(routing)
- Control plane이 data plane을 제어하고, data plane은 forwarding만 한다.
- **Generalized Forwarding**

### Network service model

- What service model for "channel transporting" datagrams
- 즉 sender → receiver, datagram을 transporting하는 channel에 대해 어떤 service model?

### Router architecture overview

![image](https://user-images.githubusercontent.com/46207836/115162867-ec268d00-a0e0-11eb-897d-cd85b686e8dd.png)

- Routing, management control plane(software) 밀리세컨드(ms)
  - 알고리즘/프로토콜 수행
- Forwarding data plane(hardware) 나노세컨트(ns) → faster
  - 왜냐면 하드웨어적인 측면에서 데이터 패스만 하면 되기 때문
- 위층은 제어영역으로 routing processor 존재. 해당 프로세서는 특정 알고리즘을 통해 네트워크 상의 최적 경로를 설정한 후, 이를 라우터(Switching fabric)에게 알려준다.
- 아래 계층은 forwarding이 일어나는 데이터 영역 계층이다. 라우터를 기준으로 왼쪽에는 입력포트(Input ports)가 존재하고, 오른쪽에는 출력 포트(output ports)가 whswogksek.
- 입력포트와 출력 포트는 둘 다 물리 계층의 기능을 수행한다.(즉 단순히 node와 node끼리의 전송 수행)
- 입력 포트를 통해 들어온 데이터는 라우터 내에 있는 forwarding table에 들어간다.
- 라우터는 forwarding table을 보고 목적지를 알아낸다.
- 그 후 라우터는 해당 데이터를 목적지로 향할 수 있는 출력 포트로 내보낸다.

#### Input port functions

![image](https://user-images.githubusercontent.com/46207836/115162892-0f513c80-a0e1-11eb-993f-cc7c2861a14c.png)

- Physical layer
  - Bit-level reception
- Data link layer(추후)
- **Decentralized switching(분산 스위칭)**
  - Datagram header의 목적지 주소로 **입력 포트 메모리(input port)에 있는 Forwarding table**을 이용하여 **출력 포트(output port)**를 검색한다.
  - 목표 : Input port processing을 line speed로 처리하는 것
  - **Queueing** : 만약 datagram이 switch fabric에 forwarding하는 속도보다 빠르면
  - Forwarding
    - **Destination-based forwarding**
      - 오직 dest IP주소만 가지고 forward(traditional) → slow
    - **Generalized forwarding** 
      - Header의 어떠한 값을 기반으로 forward

- 즉 forwarding table을 기반으로 어떤 식으로 처리를 해서 출력 포트를 정하는 걸까

### Destination-based forwarding

![image](https://user-images.githubusercontent.com/46207836/115163312-67893e00-a0e3-11eb-9e3b-0f629b9fc08f.png)

- **Longest Prefix Matching** 
  - When looking for forwarding table entry for given destination address, use **longest address prefix that matches destination address**.
  - 즉 포워딩테이블에서 가장 길게 대응되는 값에 해당하는 값을 찾고 그게 지명하는 링크로 데이터를 내보내는 방식

### Switching fabrics

- transfer packet from input buffer to appropriate output buffer

- switching rate

- 3개의 타입

  - Memory

    - CPU가 직접 입/출력 포트 지정
    - 한 번에 하나의 작업만
    - 패킷을 시스템 메모리에 복사(즉, 모든 Input buffer로부터 들어오는 패킷들은 같은 메모리 공유)
    - 메모리 대역폭에 따라 속도 제한(datagram당 2번 시스템 버스에 접근 input→ memory/ memory → output)

  - Bus

    - 공유버스를 통해 입력 포트 메모리의 datagram을 출력 포트 메모리로 이동
    - **Bus contention** : switching speed **limited by bus bandwidth**
    - 여러 데이터가 들어와도 버스에 싣을 수 있는 데이터는 오직 하나, 다른 데이터는 대기

  - Crossbar

    - 버스의 대역폭 한계를 극복
      - Crossbar switch : n개의 입력포트와 n개의 출력포트로 연결된 2n의 버스로 구성

    - 각 수직 버스는 교차점에서 각 수평 버스와 교차하며 언제든지 열고 닫기 가능
    - 여러 데이터가 도착해도 병렬 처리 가능 → 지연 단축
    - 하지만 만약 서로 다른 메세지가 같은 출력 포트로 보내지는 경우, 하나의 데이터만 통과하고, 목적 링크가 같은 또다른 데이터 대기

### Input port queueing

- 스위치 구조가 모든 도착하는 패킷을 전달할 정도로 빠르지 않기 때문에 queueing 필요
- **HOL blocking**
  - 큐의 앞부분에 저장된 데이터그램이 큐의 다른 것들이 전달되는 것을 방해

### Output ports

- buffering required from fabirc faster rate

### Output port queueing

- **buffering**
  - arrival rate via switch > output line speed
- **queueing (delay) and loss**
  - output port buffer overflow!
- **How much buffering?**

![image](https://user-images.githubusercontent.com/46207836/115167962-5694f880-a0f4-11eb-9653-1ed222063160.png)

버퍼링 용량(B bit buffer) = link 용량(C bps) * 전형적인 RTT(250msec)

여기에 최근 대용량 TCP 흐름의 값에 따른 버퍼링 필요량으로 새롭게 계산

### Scheduling mechanisms

- FIFO
  - discard policy
    - Tail drop
    - Priority
    - Random
- Priority Scheduling
  - Send highest priority queued packet
  - Multiple classes, with different priorities
- Round Robin(RR)
  - Multiple classes
  - Cyclically scan classs : queues, sending one complete packet from each class(if available)
- Weighted Fair Queueing(WFQ)
  - generalized Round Robin
  - each class gets weighted amount of service in each cycle

### The Internet network layer

![layerfunc](https://user-images.githubusercontent.com/46207836/115182431-e9458f80-a114-11eb-9277-8194ea7f456b.png)

### IP datagram format

![ipdatagram](https://user-images.githubusercontent.com/46207836/115182580-2b6ed100-a115-11eb-81e0-30ff5e4aeff8.png)

### IP fragmentation, reassembly

- Network links have MTU(max. Transfer size) - largest possible link-level frame
- Large IP datagram divided ("fragmented") within net
  - One datagram becomes several datagrams
  - "reassembled" only at final destination
  - IP header  bits used to identify, order related fragments

### IP addressing

- IP address : 32 bit identifier for host, router interface
- interface : connection between host/router and physical link

### Subnets

- IP address
  - subnet part - high order bits
  - host part - low order bits
- 서브넷이란 무엇일까?
  - Device interfaces with same subnet part of IP address
  - can physically reach each other without intervening router
  - 외딴 섬

### IP addressing : CIDR

- **CIDR** : Classess InterDomain Routing
  - Subnet portion of address of arbitrary length
  - Address format : **a.b.c.d/x** where x is # bits in subnet portion of address
  - 클래스가 없다는 것은 네트워크 구분을 클래스로 하지 않는 것

### DHCP

### DHCP client-server scenario

### Hierarchical addressing : route aggregation























