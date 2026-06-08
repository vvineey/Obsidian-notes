


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

Control Plane의 두 가지 방식 

전통적인 per-router control 
각 router 안에 routing algorithm이 있고 router들이 서로 정보를 교환해서 forwarding table을 만듦
![[Pasted image 20260608141327.png]]

SDN (Software-Defined Networking)
중앙 Controller가 전체 Network를 보고 forwarding table을 계산해서 각 router에게 내려줌 
![[Pasted image 20260608141345.png]]



# 5. IP의 Best-Effort Service

Network layer는 
- 보장된 전달
- 일정 지연 시간 이내 전달 
- 순서 보장
- 일정 속도 보장

근데 IP는 best-effort service 제공

Best-Effort Service란
	*최대한 전달하려고는 하는데 성공, 지연, 순서, 속도를 보장하지 않는 서비스*

패킷 유실 가능성 있음, 지연 가능성 있음, 순서 바뀔 가능성 있음 
But, router구조가 단순해지고 확장성이 좋음 



# 6. 목적지 기반 Forwarding 

가장 기본적인 forwarding 방식

1. Router가 *datagram header에 있는 목적지 IP* 확인
2. Forwarding table에서 해당 목적지에 맞는 output interface를 찾음 


# 7. LPM (Longest Prefix Matching)


