---
tags:
  - cs/network
  - course/iot-network
  - type/lecture-note
  - network/network-layer
  - network/control-plane
  - network/routing
---


# Routing Algorithm 목적

송신자부터 수신자까지 packet을 지날 때 최적의 path를 결정하는 것 

Network는 그래프로 표현한다.
```
Node = router
Edge = link
Cost = link를 사용할 때 드는 비용 (hop 수, delat, congestion 정도 등등)
```


# Link State와 Distance Vector

![[Pasted image 20260608163043.png|642]]

|구분|Link State, LS|Distance Vector, DV|
|---|---|---|
|정보 범위|전체 network topology와 link cost를 알고 있음|처음에는 이웃 정보만 알고 있음|
|동작 방식|전체 graph를 기반으로 계산|이웃과 distance vector 교환|
|대표 알고리즘|Dijkstra|Bellman-Ford|
|성격|global|decentralized|

