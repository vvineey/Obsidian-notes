


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

Forwarding table에 저장된 prefix 형태의 목적지 주소 범위에서 
여러 entry가 매칭되면 가장 긴 prefix가 일치하는 entry를 선택함 

![[Pasted image 20260608142553.png]]

```
ex) 200.23.16.0 ~ 200.23.23.255

1. 200.23.16.0를 2진수로 변경 -> 11001000 00010111 00010000 00000000
   
   200 = 11001000
   23  = 00010111
   16  = 00010000
   0   = 00000000
   
   
2. 200.23.23.255를 2진수로 변경 -> 11001000 00010111 00010111 11111111
   
   200 = 11001000
   23  = 00010111
   23  = 00010111
   255 = 11111111
   
3. 200.23.16.0/20 vs 200.23.24.0/24가 다 맞으면 /24 선택 
```



# 8. Router의 Buffering 

Router에 packet이 들어오는 속도 > 나가는 속도면  queue가 생김 
이때 router는 packet을 buffer에 임시 저장함 
이때 buffer는 무한하지 않으므로 buffer가 가득 차면  패킷 손실이 발생함 

- Buffering : 
- Scheduling :  


# 9. RED & ECN

- RED (Random Early Detection) 
  - buffer가 가득 차기 전에 확률적으로 packet drop 
  
- ECN (Explicit Congestion Notification) 
  - 혼잡 발생 전에 송신자측에서 혼잡을 감지하도록 함 
  - header에 ECN field 표시 -> 혼잡을 알림 

# 10. Packet Scheduling 

Router의 output queue에 여러 packet중 어떤 packet을 먼저 보낼지 결정 

### FCFS / FIFO
가장 먼저 온 packet을 먼저 보냄 

### Priority Scheduling 
우선순위가 높은 queue

## Round Robin
여러 queue를 순서대로 돌아가며 하나씩 

### Weighted Fair Queueing RR
RR의 일반화 방식
각 queue에 가중치 부여 -> 가중치가 높은 queue에 더 많은 전송 기회 부여 



