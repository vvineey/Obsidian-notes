
# Network layer  vs   Link layer

Network layer 
```
Host A -> Router 1 -> Router 2 -> Router 3 -> Host B
```

Link layer
```
Host A -> Router 1
Router 1 -> Router 2
Router 2 -> Router 3
Router 3 -> Host B
```




# Link layer

IP datagram을 link를 통해 물리적으로 옆에 있는 이웃 node에게 전달한다.

```
Network layer의 IP datagram + link layer header/ trailer가 붙어서 Frame이 된다.
```

이렇게 하는 이유는 link마다 필요한 정보가 다르기 때문이다.

예를 들어 Wi-Fi link와 Ethernet link는 물리적 환경이 다르다.  
무선은 간섭과 충돌 문제가 크고, 유선은 상대적으로 안정적이다.  
그래서 같은 IP datagram이라도 지나가는 link마다 다른 link layer protocol이 적용될 수 있다.

**Network layer의 IP는 end-to-end 관점이고, Link layer는 hop-by-hop 관점이다.

Link layer는 
- Error detection
- Error correction
- Multiple access (다중 접속)
기능을 제공한다.



# Error detection

왜 error detection이 필요함?
물리적 link에서는 bit가 깨질 수 있으므로 송신자가 data에 *EDC*라는 추가 bit를 붙여 전송함 

```
EDC bit 많음 → 오류 검출 능력 증가, overhead 증가
EDC bit 적음 → 오류 검출 능력 감소, overhead 감소
```



# Parity Checking 

### Single-bit parity 

Detection만 가능 -> 1의 개수가 짝수인지 홀수인지만 판별 
```
Data: 1011
1의 개수: 3개
Even parity bit: 1

전체: 10111
1의 개수: 4개
```


### Two-dimensional parity 

행과 열을 같이 확인 -> Correction도 가능 




# Multiple Access Link

Link의 두 종류 
- Point - to - point : 두 node 직접 연결, 전용 !!
- Broadcast : 여러 node가 같은 link를 공유 (Wi-Fi, 위성 통신)

broadcast의 경우 두 node가 동시에 통신 -> 충돌 -> frame 손상 
따라서 Multiple Access Protocol 필요 




# Multiple Access Protocol 

### Channel partitioning

미리 자원을 나눠주는 방식이다. 
- Time-Division MA : 시간 분할
  - 일정한 round로 동작, 각 node는 각 round에서 고정된 길이의 slot을 얻어 전송할 수 있음 
  
- Frequenct-division MA : 주파수 분할
  - 채널이 주파수 band로 나눠짐, 각 node는 고정된 주파수 band 할당받음 
  
충돌이 적지만 보낼 데이터가 없는 경우 자원이 낭비된다. 


### Random access
미리 자원을 나누지 않고 node가 보낼 data가 있으면 channel에 접근한다. 
자원을 더 유용하게 사용할 수 있지만, 충돌을 완전히 없애지는 못함 


