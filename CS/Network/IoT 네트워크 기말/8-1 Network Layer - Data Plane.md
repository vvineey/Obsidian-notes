


# 요약
Network layer는 Transport layer가 만든 segment를 받아서 출발 host에서 목적 host로 전달하는 계층임!!
Transport layer는 양 끝 host에만 있지만 Network layer는 router에도 존재함

---

# Network Layer의 역할

Transport의 TCP/UDP Segment + IP header -> IP datagram을 만들고 host로 전달 
- 송신자 : Transport Segment + IP header = IP datagram 
- 수신자 : 