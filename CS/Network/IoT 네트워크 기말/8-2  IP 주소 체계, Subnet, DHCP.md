
# 요약


# IP Datagram Format 

![[Pasted image 20260608145850.png|435]]


Transport layer의 segment + IP header -> IP datagram가 된다. 

IP header에는 
- source IP address
- destination IP address
- TTL : router를 지날 때마다 하나씩 감소 0이 되면 폐기 
- checksum 
- 등등
이 들어감 



# IP Address 
IP Address는 장치가 아니라 Interface를 식별한다!

IP address = interface의 식별자
router = 여러 interface를 가진 장치
같은 router라도 interface마다 IP 주소가 다를 수 있음




# Subnet

	Router를 거치지 않고 직접 통신할 수 있는 interface들의 집합

![[Pasted image 20260608151013.png|311]]
같은 subnet에 속한 host들은 같은 local network에 있으므로 직접 통신 가능, 
다른 subnet으로 가려면 router를 거쳐야 함 




# Subent Mask

IP 주소 = prefix + Host part 

```
192.168.0.10/24
```
에서 **/24** -> 앞에 24 bits : subnet part, 나머지 8bits:  host part



# IP 주소 할당 방식 

1. 수동 할당 
   네트워크 관리자가 직접 입력, 관리 
   
2. DHCP 
   Host가 network에 들어올 때마다 server로 부터 IP 주소를 동적으로 할당받음 



# DHCP (Dynamic Host Configuration Protocol)
Host가 network에 들어오면 DHCP server에게 IP 주소를 요청 -> server가 사용 가능한 IP 주소 임대 

```
DHCP Discover → DHCP Offer → DHCP Request → DHCP ACK
```

1. DHCP Discover 
   
2. DHCP Offer 
   
3. DHCP Request
   
4. DHCP ACK

+ 이전에 할당받은 주소를 기억하고 재사용하는 경우 Discover, Offer 생략 가능 
