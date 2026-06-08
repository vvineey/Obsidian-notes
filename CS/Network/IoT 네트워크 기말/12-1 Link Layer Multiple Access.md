
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



### Two-dimensional parity 