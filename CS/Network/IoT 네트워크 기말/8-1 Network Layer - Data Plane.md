


# 요약
Network layer는 Transport layer가 만든 segment를 받아서 출발 host에서 목적 host로 전달하는 계층임!!
Transport layer는 양 끝 host에만 있지만 Network layer는 router에도 존재함

---

# 1. Network Layer의 역할

Transport의 TCP/UDP Segment + IP header -> IP datagram을 만들고 host로 전달 
- 송신자 : transport Segment + IP header = IP datagram (캡슐화)
- 수신자 : IP datagram에서 header 제거 → segment를 transport layer로 전달


# 2. Forwarding vs Routing
### Forwarding
router 하나의 내부 동작
-> router의 input link로 datagram이 들어오면 forwarding table을 보고 적절한 output link로 내보냄

```
input link로 들어온 datagram → forwarding table 확인 → output link로 전달
```

### Routiong
전체 네트워크 관점 
출발지에서 목적지까지 어떤 router들을 거쳐 갈지 전체 경로를 결정함 

```
출발지 host → router A → router B → router C → 목적지 host
```



# 3. Data Plane & Control Plane

Data Plane은 datagram을 forwarding하는 기능이다.
각 router에서 local하게 일어난다.
"여기 라우터에서 이 패킷 어디로 보낼 거임?"

Control Plane은 datagram을 어떤 경로로 보낼지 결정하는 기능이다.
전체 네트워크 관점에서 routing 정보를 만든다. 
"전체적으로 어떤 경로 사용할 거임?"



# 4. 전통적인 Routing 방식과 SDN

Control Plane 구현 방식