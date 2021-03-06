---
Title : "Introduction of Network"
layout : post
date : 2021-04-14
category : Network
blog : true
author : dleunji
description : 기초 네트워크 
---

### Summary : 

- Network Edge : hosts, access net, physical media

# Chapter 1. Introduction

### Network Edge

-  그래프를 상상하면 된다.

- Node - billions of connected computing devices
  - Hosts = end systems(클라이언트와 서버)
  - Running Network apps
- Link - communication
  - Fiber, copper, radio, satellite
  - <u>transmission rate : bandwidth</u> : *Bits per Second*
- Packet Switching : forward packets(chunks of data)
  - routers and switches

### Network Core

- Interconnected routers
- Network of Networks

### The difference Between Network Edge and Network Core

- **Network Edge** refers to endpoint.
- **Network Core** refers to the components that provide services to those at the edge.

### What is Internet?

- Network of Networks : 즉 네트워크들을 잇는다

### DSL vs Cable Network

- **DSL(Digital Subscriber Line)** 
  - 지역 전화망을 통해 디지털 데이터 전송을 제공하는 기술
  - Voice ,data transmitted at different frequencies over **dedicated** line to central office
  - Use **existing** telephone line to central office DSLAM
  - DSL은 높은 주파수를 사용하고, 일반 전화는 낮은 주파수를 사용
- **Cable Network** 
  - 사용자를 Network Core에 연결
  - Homes **share** access network to cabel headend
    - Unlike DSL, which has dedicated access to central office
  - 중간에 splitter 존재
    - 광섬유가 일직선이면 데이터 전송에 무리가 없으나, 선이 꺾이면 전송률에 차이가 발생하기에 이를 방지하는 역할
  - HFC (Hybrid Fiber Coax) 
    - 비디오, 데이터 및 음성과 같은 광대역 컨텐츠를 운송하기 위해 네트워크의 서로 다른 부분에서 광섬유 케이블과 동축 케이블이 사용되는 통신 기술
    - Asymmetric : up to 30Mbps downstream transmission rate, 2Mbps upstream transmission rate
- **Home Network**

- **Ethernet(Enterprise access networks)**
  - typically used in companies, universities, etc
  - Today, end systems typically connect into **Ethernet switch**
- Wireless Access Networks
  - Shared wireless access network connects end system to router
  - Wireless LANs
    - Within building
    - WiFi
  - Wide-area Wireless Access
    - Provided by telco operator
    - 3G, 4G, LTE



### Host

- Client and Server

- sends packets of data

- breaks into smaller chunks, known as packets, of length L bits

- Transmits packet into access network at transmission rate R 

  = link capacity = link bandwidth

  ```
  packet transmission delay 
  = time needed to transmit L-bit packet into link
  = L(bits) / R(bits/sec)
  ```

  ![IMG_F889FC3DBBBB-1](https://user-images.githubusercontent.com/46207836/114731660-e47f8500-9d7c-11eb-9872-a0c89bdb243f.jpeg)



### Physical Media

- Bit : propagates between transmitter/receiver pairs

- Physical link : what lies between transmitter/receiver 

- Guided media(유선)

  - Signals propane in solid media
    - e.g. Copper, fiber, coax

- Unguided media(무선)

  - Signals propage freely
    - e.g. Radio

- **Twisted Pair(TP)**  ✰✰시험 출제 예상✰✰

  - Two insulated copper wires
  - 신호를 인식할 때 두 선의 전압차이를 가지고 신호 구분을 한다. 이 경우 두 선에 동일한 잡음이 가해진다면 두 선 사이의 전압차이는 잡음이 가해지기 전후에 차이가 없다.
  - 하지만 선이 꼬여있지 않으면, 어떤 잡음원이 근처에 있어서 전자기파를 방출하고 있을 때(e.g. TV 옆에 케이블이 지나갈 때) 그 잡음원으로부터 거리가 조금 달라지고, 영향을 받는 정도가 달라진다.
  - 그럼 두 선 사이의 전압 차에 영향을 미쳐서 신호가 왜곡되고, 잡음이 발생한다.
  - 그러나 선을 꼬아놓을 경우, 평균적으로 잡음원과의 거리가 동일해져 잡음이 줄어든다.

  ![image](https://user-images.githubusercontent.com/46207836/114735171-e0a13200-9d7f-11eb-863a-464aedd055b7.png)

- **Coaxial Cable** ✰✰시험 출제 예상✰✰

  - Two concentric copper conductors
  - Bidirectional
  - Broadband
    - Multiple Channels on cable
    - HFC
  - 가운데 심선이 신호선이 되고, 그 주위를 감사는 선이 ground역할을 하며 잡음을 방지한다.

  ![image](https://user-images.githubusercontent.com/46207836/114735960-a2584280-9d80-11eb-978a-11305bcdb45e.png)

- **Fiber Optic Cable** 

  - Glass fiber carrying light pulses, each pulse a bit
  - High-speed operation
  - Low error rate

  

- **Radio** ✰✰시험 출제 예상✰✰

  - Signal carried in electromagnetic spectrum
  - No Physical "wire"
  - Bidirectional
  - Propagation environment effects
    - Reflection (반사)
    - Obstruction by objects (장애)
    - Interference (간섭)
  - Types
    - Terrestrial microwave
    - LAN (e.g. WiFi)
    - Wide-area (e.g. cellular)
    - Satellite

---

### 용어 정리

1. LANs : Local Area Networks

2. ISP : Internet Service Provider

3. NAT : Network Address Transmission

4. BPS 

   - 데이터 통신에서 컴퓨터 모뎀과 전송매체의 데이터 속도를 나타내는 일반적인 척도

   - 매 초당 송신되거나 수신된 비트 수와 같다

   - 한 개의 데이터 비트가 전송되는 데 걸리는 시간은 bps로 표현된 디지털 전송속도 s의 역에 비례한다.

     d = 1 / s

   - 1 kbps = 1,000 bp
