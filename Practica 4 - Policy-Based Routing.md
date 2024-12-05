*******
### Topologia

*******
### SalesRouter

~~~python

SalesRouter# configure terminal
SalesRouter(config)# interface f0/0
SalesRouter(config-if)# ip address 192.168.101.1 255.255.255.0
SalesRouter(config)# no shutdown

SalesRouter(config)# interface f0/1
SalesRouter(config-if)# ip address 192.168.3.1 255.255.255.248
SalesRouter(config)# no shutdown

SalesRouter(config)# interface f1/0
SalesRouter(config-if)# ip address 192.168.102.1 255.255.255.128
SalesRouter(config-if)# no shutdown

SalesRouter(config)# router eigrp 3
SalesRouter(config-router)# eigrp router-id 0.0.0.1
SalesRouter(config-router)# network 192.168.101.0 0.0.0.255
SalesRouter(config-router)# network 192.168.102.0 0.0.0.127
SalesRouter(config-router)# network 192.168.3.0 0.0.0.7
SalesRouter(config-router)# no auto-summary

~~~

### EngRouter

~~~python

EngRouter# configure terminal
EngRouter(config)# interface f0/0
EngRouter(config-if)# ip address 192.168.201.1 255.255.255.0
EngRouter(config-if)# no shutdown

EngRouter(config)# interface f1/0 
EngRouter(config-if)# ip address 192.168.202.1 255.255.255.128
EngRouter(config-if)# no shutdown

EngRouter(config)# interface f0/1
EngRouter(config-if)# ip address 192.168.3.2 255.255.255.248
EngRouter(config-if)# no shutdown

EngRouter(config)# router eigrp 3
EngRouter(config-router)# eigrp router-id 0.0.0.2
EngRouter(config-router)# network 192.168.201.0 0.0.0.255
EngRouter(config-router)# network 192.168.202.0 0.0.0.127
EngRouter(config-router)# network 192.168.3.0 0.0.0.7
EngRouter(config-router)# no auto-summary

~~~

### GDL

~~~python
GDL# configure terminal
GDL(config)# interface f1/0
GDL(config-if)# ip address 192.168.3.3 255.255.255.248
GDL(config-if)# no shutdown
GDL(config)# interface s0/0
GDL(config-if)# ip address 123.3.3.1 255.255.255.252
GDL(config-if)# no shutdown
GDL(config)# interface s0/1
GDL(config-if)# ip address 125.3.3.1 255.255.255.252
GDL(config-if)# no shutdown
GDL(config)# interface f0/0
GDL(config-if)# ip address 128.3.3.1 255.255.255.252
GDL(config-if)# no shutdown
GDL(config)# interface f0/1
GDL(config-if)# ip address 129.3.3.1 255.255.255.252
GDL(config)# no shutdown

GDL(config)# access-list 100 permit ip 192.168.101.0 0.0.0.255 any
GDL(config)# access-list 101 permit ip 192.168.102.0 0.0.0.127 any
GDL(config)# access-list 102 permit ip 192.168.201.0 0.0.0.255 any
GDL(config)# access-list 103 permit ip 192.168.202.0 0.0.0.127 any

GDL(config)# route-map PBR permit 10
GDL(config-route-map)# match ip address 100
GDL(config-route-map)# set ip next-hop 123.3.3.2 
GDL(config)# route-map PBR permit 20
GDL(config-route-map)# match ip address 101
GDL(config-route-map)# set ip next-hop 125.3.3.2
GDL(config)#route-map PBR permit 30
GDL(config-route-map)# match ip address 102
GDL(config-route-map)# set ip next-hop 128.3.3.2
GDL(config)# route-map PBR permit 40
GDL(config-route-map)# match ip address 103
GDL(config-route-map)# set ip next-hop 129.3.3.2

GDL(config)# interface f1/0
GDL(config-if)# ip policy route-map PBR

GDL(config)# router eigrp 3
GDL(config-route)# eigrp router-id 0.0.0.3
GDL(config-route)# network 192.168.3.0 0.0.0.7
GDL(config-route)# network 123.3.3.0 0.0u.0.3
GDL(config-route)# network 125.3.3.0 0.0.0.3
GDL(config-route)# network 128.3.3.0 0.0.0.3
GDL(config-route)# network 129.3.3.0 0.0.0.3
GDL(config-route)# no auto-summary

~~~

### ProovedorWAN1

~~~python

ProovedorWAN1# configure terminal
ProovedorWAN1(config)# interface s0/0
ProovedorWAN1(config-if)# ip address 123.3.3.2 255.255.255.252
ProovedorWAN1(config-if)# no shutdown

ProovedorWAN1(config)# interface f0/0
ProovedorWAN1(config-if)# ip address 172.16.3.1 255.255.255.128
ProovedorWAN1(config-if)# glbp 3 ip 172.16.3.126
ProovedorWAN1(config-if)# glbp 3 preempt
ProovedorWAN1(config-if)# glbp 3 priority 120
ProovedorWAN1(config-if)# no shutdown

ProovedorWAN1(config)# router eigrp 3
ProovedorWAN1(config-router)# eigrp router-id 0.0.0.4
ProovedorWAN1(config-router)# network 123.3.3.0 0.0.0.3
ProovedorWAN1(config-router)# network 172.16.3.0 0.0.0.127
ProovedorWAN1(config-router)# no auto-summary

~~~


### ProovedorWAN2

~~~python

ProovedorWAN1# configure terminal
ProovedorWAN1(config)# interface s0/0
ProovedorWAN1(config-if)# ip address 125.3.3.2 255.255.255.252
ProovedorWAN1(config-if)# no shutdown

ProovedorWAN1(config)# interface f0/0
ProovedorWAN1(config-if)# ip address 172.16.3.2 255.255.255.128
ProovedorWAN1(config-if)# glbp 3 ip 172.16.3.126
ProovedorWAN1(config-if)# glbp 3 preempt 
ProovedorWAN1(config-if)# glbp 3 priority 110
ProovedorWAN1(config-if)# no shutdown

ProovedorWAN1(config)# router eigrp 3
ProovedorWAN2(config-router)# eigrp router-id 0.0.0.5
ProovedorWAN1(config-router)# network 125.3.3.0 0.0.0.3
ProovedorWAN1(config-router)# network 172.16.3.0 0.0.0.127
ProovedorWAN1(config-router)# no auto-summary

~~~


### ProovedorWAN3

~~~python

ProovedorWAN3# configure terminal
ProovedorWAN3(config)# interface f0/0
ProovedorWAN3(config-if)# ip address 128.3.3.2 255.255.255.252
ProovedorWAN3(config-if)# no shutdown

ProovedorWAN3(config)# interface f0/1
ProovedorWAN3(config-if)# ip address 172.16.3.3 255.255.255.128
ProovedorWAN3(config-if)# glbp 3 ip 172.16.3.126
ProovedorWAN3(config-if)# glbp 3 preempt 
ProovedorWAN3(config-if)# glbp 3 priority 105
ProovedorWAN3(config-if)# no shutdown

ProovedorWAN3(config)# router eigrp 3
ProovedorWAN3(config-router)# eigrp router-id 0.0.0.6
ProovedorWAN3(config-router)# network 125.3.3.0 0.0.0.3
ProovedorWAN3(config-router)# network 172.16.3.0 0.0.0.127
ProovedorWAN3(config-router)# no auto-summary

~~~

### ProovedorWAN4

~~~python

ProovedorWAN4# configure terminal
ProovedorWAN4(config)# interface f0/0
ProovedorWAN4(config-if)# ip address 129.3.3.2 255.255.255.252
ProovedorWAN4(config-if)# no shutdown

ProovedorWAN4(config)# interface f0/0
ProovedorWAN4(config-if)# ip address 172.16.3.4 255.255.255.128
ProovedorWAN4(config-if)# glbp 3 ip 172.16.3.126
ProovedorWAN4(config-if)# glbp 3 preempt 
ProovedorWAN4(config-if)# glbp 3 priority 100
ProovedorWAN4(config-if)# no shutdown

ProovedorWAN4(config)# router eigrp 3
ProovedorWAN4(config-router)# eigrp router-id 0.0.0.7
ProovedorWAN4(config-router)# network 125.3.3.0 0.0.0.3
ProovedorWAN4(config-router)# network 172.16.3.0 0.0.0.127
ProovedorWAN4(config-router)# no auto-summary

~~~

### VPCS

#### FileServer

~~~python
FileServer> ip 172.16.3.5/25 172.16.3.254
FileServer> save
~~~

#### Sales1

~~~python
Sales1> ip 192.168.101.2/24 192.168.101.1
Sales1> save
~~~

#### MgmtSales

~~~python
MgmtSales> ip 192.168.102.2/25 192.168.102.1
MgmtSales> save
~~~

#### Engineering1

~~~python
Engineering1> ip 192.168.201.2/24 192.168.201.1
Engineering1> save
~~~

#### MgmtEng

~~~python
MgmtEng> ip 192.168.202.2/25 192.168.202.1
MgmtEng> save 
~~~
