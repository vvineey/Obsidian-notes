

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
- 

단점 

# IPv6


# IPv6 Datagram Format


# Generalized Forwarding


# Flow Table과 Priority


# OpenFlow


