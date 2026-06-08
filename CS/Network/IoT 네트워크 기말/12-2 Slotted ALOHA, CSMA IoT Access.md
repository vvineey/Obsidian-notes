
# 1. Slotted ALOHA

시간을 같은 크기의 slot으로 나누고, node는 slot의 시작 시점에만 전송한다.

- 모든 frame 크기는 같다.
- frame 하나를 보내는 데 slot 1개가 필요하다.
- 시간은 같은 크기의 slot으로 나뉜다.
- node는 slot 시작 지점에서만 전송한다.
- node들은 서로 동기화되어 있다.
- 둘 이상의 node가 같은 slot에서 전송하면 collision이 발생한다.
- 충돌 나면 랜덤 시간 기다린 후 재전송한다.


### slot의 세가지 상태 

```
0개 node 전송 → empty slot
1개 node 전송 → successful slot
2개 이상 전송 → collision slot
```

###  장점 
- node 하나만 전송할 때는 최대 속도를 사용할 수 있다.
- 충돌 후 재전송 시점이 확률적으로 분산된다.
- 동작이 단순해서 구현이 쉽다.

### 단점
- collision이 발생하면 slot 전체가 낭비된다.
- 전송 node가 없으면 empty slot이 생긴다.
- slot 단위 시간 동기화가 필요하다.
- 최대 효율이 높지 않다.




# 2. CSMA 

CSMA : Carrier Sense Multiple Access

- 전송 전에 channel이 idle인지 busy인지 확인한다.
- idle이면 전송한다.
- busy이면 전송을 연기한다.

### CSMA의 충돌 
**propagation delay**에 의해 충돌이 발생할 수 있다.



# 3. CSMA/CD

CSMA/CD : **Carrier Sense Multiple Access with Collision Detection**

- 전송 전에 channel을 듣는다.
- idle이면 전송한다.
- 전송 중 collision을 감지하면 즉시 전송을 멈춘다.

**collision detection** 
충돌이 났는데도 전체를 계속 보내면 channel 낭비 -> 충돌 감지시 중단 

Ethernet 같은 유선 환경에서 가능 ! 
(유선에서는 송신 중에도 전압 변화 등을 통해 충돌 감지가 가능하다.)



# 4. CSMA/CA

CSMA/CA : **Carrier Sense Multiple Access with Collision Avoidance**

```
collision이 바로 detection되지 않으므로, collision이 발생하지 않도록 최대한 피해야 한다.
```

무선 환경에서는 CD가 어려움 
왜냐?
내가 전송하는 신호가 너무 강해서 동시에 들어오는 약한 시신호를 감지하게 어려움 

-> 충돌을 감지해서 멈추기보다 충돌이 날 가능성을 줄이도록 backoff를 통해 기다렸다가 전송한다.


|구분|핵심|대표 환경|
|---|---|---|
|CSMA|전송 전 channel을 듣는다|기본 원리|
|CSMA/CD|collision을 감지하면 전송 중단|유선 Ethernet|
|CSMA/CA|collision을 피하도록 backoff|무선 Wi-Fi|
