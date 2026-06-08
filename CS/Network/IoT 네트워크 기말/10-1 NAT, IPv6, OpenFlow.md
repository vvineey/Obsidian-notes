

# 요약


# Network Address Translation

내부 네트워크의 여러 장치가 외부 인터넷에 나갈 때 **하나의 public IP 주소를 공유**하게 해주는 기술
Local network를 떠나는 datagram은 같은 NAT IP 주소를 source address로 가진다. 
port number는 다르다.!!


# Private IP
NAT 내부에서는 **private IP**를 사용한다.  
Private IP는 인터넷 전체에서 직접 routing되지 않고, local network 안에서만 의미가 있다.


# NAT Translation Table

![[Pasted image 20260608154130.png|531]]

```
private IP + 내부 port <-> public IP + NAT가 새로 고른 port
```


# NAT 장점과 단점 

장점 
- ISP로부터 하나의 public IP만 받아도 local network의 여러 장치가 인터넷 사용 가능 
- Local network  안의 host 주소를 외부에 알리지 않고 변경 가능 
- 외부에서 내부 장치 노출이 되지 않아 보안측면 이점 있음 

단점 
- 원래 인터넷은 end-to-end 통신 구조인데 NAT가 중간에서 주소와 port를 변경하므로 
  원칙이 약해짐 
- 외부에서 내부 host로 접속하기 어려움 따라서 port forwarding 설정 필요 


# IPv6
IPv6는 128-bits 주소 

왜 필요 ?
- IPv4의 주소 공간 부족 
- IP header 크기를 줄여 전송 효율 향상
- Flow에 따라  network layer에서 다르게 처리 가능성 


# IPv6 Datagram Format

![[Pasted image 20260608155106.png|561]]

#IPv4_header 에 비해 크기가 작아짐 
- checksum 없음 -> router 처리 속도 향상
- option 없음 -> 기본 herder 단순화 
- fragmentation/reassembly 없음 -> router 처리 단순화 

-> Router가 packet을 더 빠르고 단순하게 처리하도록 header 구조도 바꿨다!



# Generalized Forwarding

목적지 기반 forwarding은 목적지 IP주소를 보고 output link 결정 
Generalized forwarding은 **Match + Action 방식

### Match
Header field에서 특정 pattern을 찾는다.

```
source IP
destination IP
TCP/UDP port
protocol type
flow label
```

### Action
Match된 packet에 대해 행동을 수행한다.

```
forward
drop
modify header
send to controller
```


# Flow Table과 Priority

generalized forwarding은 forwarding table이 아니라 **flow table**을 사용한다.
근데 하나의 Packet이 여러 rule에 걸리면 -> rule마다 우선순위를 정하고 결정함 



# OpenFlow

Generalized Forwarding을 실제 프로토콜로 구현한 방식중 하나!
header field에서 패턴을 찾아 특정 actiond을 수행하게 함 


![[Pasted image 20260608161551.png|513]]

OpenFlow는 SDN 연관지어서 
SDN Controller -> flow table 계산 -> OpenFlow switch가 match + action 수행