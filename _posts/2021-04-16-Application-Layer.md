---
Title : "Application Layer"
layout : post
date : 2021-04-16
category : Network
blog : true
author : dleunji
description : Application Layer
---

## Summary:

- Conceptual implementation aspects of network application protocols
  - Transport-layer service models
  - Client-Server Paradigm
  - Peer-to-Peer Paradigm
  - Content Distribution Networks
- Network application-level protocols
  - HTTP
  - FTP
  - SMTP / POP3 / IMAP
  - DNS
- Creating network applications
  - socket API

---

### Network Apps

- No need to write software for network-core devices



### Application Architectures

- **Client-Server**

  - Server
    - **Always-on host**
    - Permanent IP address
    - Data centers for scaling
  - Client
    - Communicate with server
    - May be intermittently connected
    - May have dynamic IP addresses
    - **Do not communicate directly with each other**

- **P2P**

  - **No** always-on server
  - Arbitrary end systems **directly communicate**
  - Peers request service from other peers, provide server in return to other peers
    - Self Scalability
  - Peers are intermittently connected and change IP addresses
    - Complex management

  

### Processes Communicating

- `process : program running within a host`
  - **Within same host**, two processes communicate **using inter-process communication**(defined by OS)
  - processes in **different hosts** communicate **by exchanging messages**

- Clients, Servers
  - **Client process** : process that initiates communication
  - **Server process** : process that waits to be contacted

- Applications with P2P architectures have client processes & server processes

### Sockets

- process sends/receives messages to/from its socket
- Socket analogous to door(비유)
  - Sending process shoves message out door
  - Sending process relies on **transport infrastructure on other side of door** to deliver message to socket at receiving process

![socket](https://user-images.githubusercontent.com/46207836/114892970-32ad8a80-9e48-11eb-8fae-b90cd8f52659.png)

### Addressing Processes

- To receive messages, process must have **identifier**

- Identifier includes both **IP address** and **port numbers** associated with process on host

### App-layer Protocol defines

- Types of messages exchanged
- Message Syntax
- Message semantics
- Rules for when and how processes send & respond to messages
- Open Protocols
- Priprietary Protocols

### What transport service does an app need?

- Data integrity
- Timing
- Throughput



### Internet Transport Protocols Services

#### **TCP service**

- **Reliable transport** between sending and receiving process
- **Flow control** : sender won't overwhelm receiver
- **Congestion control** : throttle sender when network overloaded
- Does not provide : timing, minimum throughput guarantee, security
- **Connection-oriented** : setup required between client and server processes



#### **UDP service**

- **Unrealiable** data transfer between sending and receiving process
- Does not provide : 
  - reliability, flow control, congestion control
  - timing, throughput guarantee, security
  - connection setup



**Flow control과 Congestion control은 비슷해보이나 다르다.**

- Flow control : mechanism that controls the traffic between sender and receiver
- Congestion control : mechanism that controls the traffic that is placed by the transport layer into the network



### Web and HTTP

- HTTP : Hypertext Transfer Protocol

  - Web's application layer protocol

  - Client/Server model

    - **Client** : browser that **requests, receives** (using HTTP protocol) and **displays** Web objects
    - **Server** : Web server **sends**(using HTTP protocol) objects in **response to requests**

    ![http](https://user-images.githubusercontent.com/46207836/114900244-a783c300-9e4e-11eb-908d-dc51399382d6.png)

- TCP를 이용한 HTTP protocol
  - **Client** initiates TCP connection (creates socket) to server , port 80
  - **Server** accepts TCP connection from client
  - HTTP messages (application-layer protocol messages) exchanged between browser (HTTP client) and Web server(HTTP server)
  - TCP connection closed
  - HTTP is stateless
    - Server maintains no information about past client requests

### HTTP Connections

- Non-persistent HTTP 
  - **At most one** object sent over TCP connection
    - connection then closed
  - Downloading multiple objects required multiple connections
  - Requires a **TCP/IP 3-way handshake for every requested object**
- Persistent HTTP
  - **Multiple objects** can be sent over single TCP connection between client, server
  - allows the transfer of multiple objects **without re-establishing connections via 3-way handshakes**
- Both protocols have advantages but persistent wins out in most modern applications(e.g. Google)
- Roud-Trip Time(RTT) : the amount of time it takes for a signal **to be sent** + the amount of time it takes for acknowledgement of that signal **having been received.**
- Non-persistent HTTP는 2 * RTT  + file transmission time 필요. 
- Non-persistent HTTP에서 매 TCP connection마다 OS overhead가 있다.

![nonpersistentHTTP](https://user-images.githubusercontent.com/46207836/114903764-157db980-9e52-11eb-80c6-3f997265f0ef.png)

### Cookies

- 4 Components
  - Cookie header line of HTTP response
  - Cookie header line in next HTTP request message
  - Cookie file kept on user's host, managed by user's browser
  - Back-end database at Web site
- What cookies can be used for...
  - Authorization
  - Shopping cart
  - Recommendations
  - User session state(Web e-mail)

- How to keep "state"
  - protocol endpoints
  - cookies

### Web caches(proxy server)

- Goal : satisfy client request **without involving origin server**
- Web accesses via cache
- Browser sends all HTTP **requests to cache**
  - cache에 object가 있는 경우 : cache returns object
  - cache에 object가 없는 경우 : cache requests object from origin server, then returns object to client

- Cache acts as both client and server

  - Server for original requesting client (일반적인 client의 server)
  - Client to origin server(origin server의 client)

- Typically cache is installed by ISP

- Why?

  - Reduce response time for client request
  - Reduce traffic on an institution's access link
  - Internet dense with caches: enables "poor" content providers to effectively deliver content(P2P도 마찬가지)

- 대안 1 - Fatter access link

  ✰✰ 계산 시험 출제 예상 ✰✰

  - e.g.
    - Avg object size : 1Mbits
    - Avg request rate from browsers to origin servers : 15/sec
    - Avg data rate to browsers : 1.50 Mbps
    - RTT from institutional router to any origin server : 2 sec
    - Access link rate : 15Mbps → 150 Mbps
  - Consequences:
    - LAN utilization : 15 %
    - Access link utilization : 99 % → 9.9%
    - Total delay = Internet delay + access delay + LAN delay
    - 2sec + minutes + usecs → 2sec  + msecs + usecs 
  - Increased access link speed is Not cheap

- 대안 2 - Install local

  - Web cache is Cheap
  - 예를 들어 cache hit가 0.4라고 가정할 경우, 
    - 40% requests satisfied at cache
    - 60% requests satisfied at origin server
  - Access link utilization 
    - Data rate to browsers over access link
    - Access link rate * Data rate = 15Mbps * 0.6 = 9Mbps
    - Utilization = 9 /  15 = 60%
  - Total delay = (0.6 * delay from origin server) + (0.4 * delay when satisfied at cache) = 0.6 * 2.01 + 0.4 (~msecs) = ~1.2 secs
  - less than with 150Mbps link(and cheaper too!)

### Conditional GET

- Goal : don't send object if cache has up-to-date cached version
  - No bject transmission delay
  - Lower link utilization
- Cache : HTTP request에 날짜를 명시
  - If-modified-since : \<date\>
- Server : response contains no object if cached copy is up-to-date
  - HTTP/1.0 304 Not modified

### Electronic mail

- User agents

- Mail servers

  - Message queue of outgoing(to be sent) mail messages
  - SMTP protocol between mail servers to send email messages

- `Simple Mail Transfer Protocol(SMTP)`

  - Uses TCP to reliably transfer email message from client to server, port 25
  - Direct transfer : sending server to receiving server
  - 3 phases of transfer
    - handshaking
    - transfer of messages
    - closure

  - Command/response interaction(like HTTP)
    - Commands : ASCII text
    - Response : status code and phrase
  - Messages must be in7-bit ASCII

### SMTP

- Uses persistent connections

- Requires message (header & body) to be in 7-bit ASCII

- CRLF to determine end of message

- HTTP와의 비교

  - HTTP : pull(HTTP가 서버일 때 기다리면 클라이언트가 갖고감)

    SMTP : push(강제로 밀어서 보냄)

  - 둘 다 ASCII

  - HTTP : encapsulated in its own response message

    SMTP : sent in multipart message

### Mail access protocols

- SMTP는 delivery/storage to receiver's server
- Mail access protocol : retrieval from the server
  - POP : Post Office Protocol
  - IMAP : Internet Mail Protocol
  - HTTP : glial, Hotmail, Yahoo! Mail...

![mail](https://user-images.githubusercontent.com/46207836/115044502-897e9700-9f10-11eb-94ce-f06cafa58c07.png)

### DNS

- Domain Name System
- **Distributed, hierarchical** database
  - centralize되면....
    - Single Point of Failure(SPOF)
    - Traffic Volume
    - Distant Centralized Database
    - Maintenance
    - Scale 불가능
- People이 식별자 가지고 있듯...
  - SSN, name, passport #
- Internet hosts, routers도...
  - IP address
  - Name
- How to map between IP address and name?

![dns](https://user-images.githubusercontent.com/46207836/115047286-57226900-9f13-11eb-884f-f347bc0dd9f8.png)

e.g. www.amazon.com

1. Client queries root server to find <u>com</u> DNS server
2. Client queries .com DNS server to get <u>amazon.com</u> DNS server
3. Client queries amazon.com DNS server to get IP address for www.amazon.com



### TLD, authorative servers

- **Top-level Domain(TLD)** servers
- 예컨대 `ko.wikipedia.org`의 최상위 도메인은 `.org`가 된다. 최상위 도메인은 [.com](https://ko.wikipedia.org/wiki/.com)과 같은 [일반 최상위 도메인](https://ko.wikipedia.org/wiki/일반_최상위_도메인)과 [.kr](https://ko.wikipedia.org/wiki/.kr) 같은 [국가 코드 최상위 도메인](https://ko.wikipedia.org/wiki/국가_코드_최상위_도메인)으로 나뉜다.
- Organization's own DNS server(s), providing authoritative hostname to IP mappings for organization's named hosts

### Local DNS name server

- Hierarchical하지 않음
- Each ISP has one
- When host makes DNS query, query is sent to its local DNS server
  - Has local cache
  - Acs as proxy, forwards query into hierarchy

### DNS Name Resolution

1. Iterated query
2. Recursive query

---

**Iterative Resolution**

request에 대한 응답 혹은 다른 서버로 안내

When a client sends an iterative request to a name server, the server responds back with either the answer to the request (for a regular resolution, the IP address we want) or the name of another server that has the information or is closer to it. The original client must then *iterate* by sending a new request to this referred server, which again may either answer it or provide another server name. The process continues until the right server is found; the method is illustrated in Figure 243.

**Recursive Resolution**

request에 대한 응답 혹은 대리 클라이언트

When a client sends a recursive request to a name server, the server responds back with the answer if it has the information sought. If it doesn't, the server takes responsibility for finding the answer by becoming a client on behalf of the original client and sending new requests to other servers. The original client only sends one request, and eventually gets the information it wants (or an error message if it is not available). This technique is shown in Figure 244.

### DNS : caching, updating records

- once (any) name server learns mapping, it caches mapping
  - Cache entries timeout after some time
  - TLD servers typically cached in local name servers
- Cached entries may be out-of-date

### Pure P2P architecture

- No always-on server

- Arbitrary end systems directly communicate

- Peers are intermittently connected and change IP addresses

- Requesting Chunks

- Sending Chunks : **Tit-For-Tat**

  = Equivalent Relation = 나에게 이득이 되는 peers는 곧 높은 우선순위의 peer

### Client-server vs P2P

![c-s](https://user-images.githubusercontent.com/46207836/115052134-8daeb280-9f18-11eb-81ee-863d2a526c90.png)

![p2p](https://user-images.githubusercontent.com/46207836/115052125-8be4ef00-9f18-11eb-81c1-99ff0aa6cea4.png)

### Video Streaming and CDN(Content Distribution Networks)

- 제일 고민되는 부분 : Scale
- Challenge : **Heterogeneity**
  - Different users have different capabilities
- Solution : distributed, application-level infrastructure

### Streaming multimedia : DASH

- Dynamic Adaptive Streaming over HTTP

- Heterogeneity 해결

- Server:

  - chunk단위로 잘게 파일을 쪼개서 저장, 전송
  - 고화질/저화질 등 다양한 옵션으로 chunk 압축

- Client:

  - 주기적으로 서버-클라이언트 간의 bandwidth 측정
  - choose coding rate

- Intelligence at client

  - Higher quality when more bandwidth available
  - 즉 화질이 나빠질 뿐이지 끊기지는 않는다
  - 언제 chunk를 요구할 것인가
  - 어떤 encoding rate를 요구할 것인가
  - 어디서 chunk를 요구할 것인가

- Challenge : how to stream content to hundreds of thousands of simultaneous users?

  1. Single , large "mega-server" → This solution doesn't scale

  2. Store / Server multiple **copies** of videos at multiple geographically distributed sites(**CDN**)

     - Stores copies of content at CDN nodes
       - e.g. Netflix stores copies of Madmen
     - Subscribers requests content from CDN

     - 그 다음 문제로 OTT challenges : coping with a congested Internet
       - from which CDN node to retrieve content?
       - viewer behavior in presence of congestion
       - what content to place in which CDN node?

![image](https://user-images.githubusercontent.com/46207836/115098659-2cb0ca00-9f6c-11eb-8097-c7cb8661d90c.png)

- 가장 가까운 서버에서 받아오기

![image](https://user-images.githubusercontent.com/46207836/115098700-6681d080-9f6c-11eb-8f1c-17b5f9ab6ab6.png)

- No DNS redirection : redirection 시에 node selection algorithm으로 RTT(거리 traffic) 측정 (load balance)
- Push caching 
  - CDN 서버에 어떤 영화를 둘 것인가의 문제, 특정 지역에서 특정 영화를 많이 본다고 예측하여 트래픽이 적을 대 미리 영화를 서버에 넘겨둔다.
  - 나와 가까운 쪽에 요청해서 있으면 거기서 받아오고, 그 서버에 없으면 originial server에서 받아온다.(마치 web cache처럼)

### Socket programming

- Socket : door. between application process and end-end-transport protocol

- 크게 UDP, TCP transport services

  - UDP : **unreliable** datagram

    - **No connection** between client and server
    - **No handshaking** before sending data
    - sender는 IP주소와 포트 번호를 attach, receiver는 그걸 extract
    - lost나 순서 뒤바뀔 가능성 있음

  - TCP : **reliable**, byte stream-oriented

    - Client must contact server
    - Server process must first be running
    - Server must have created socket taht welcomes client's contact
    - 그러면 Client는 서버에 어떻게 접근하는가?
      - Create TCP socket(IP 주소 + 포트 번호)
      - Establish connection
      - Client에 의해 contact가 이루어지고, 서버도 새로운 socket을 만들어서 특정 client와 communicate
        - 이로써 하나의 서버가 여러 개의 클라이언트와 주고받을 수 있음
        - source port #는 클라이언트를 구별할 때 사용됨
    - Reliable ,in-order byte-stream transfer(즉 pipe)

    ---

    - Control vs. messages

      

    - Out-of-band vs. In-band

      - e.g. Out-of-band : FTP
      - e.g. In-band : HTTP, SMTP

    ![image](https://user-images.githubusercontent.com/46207836/115099548-954e7580-9f71-11eb-9a9d-01f85d8aac2e.png)

    