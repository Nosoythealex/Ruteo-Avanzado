## Topologia

!

## R1

~~~python
configure terminal
!
mpls ip 
mpls label protocol ldp
!
interface Loopback0
	ip address 10.255.255.1 255.255.255.255
!
mpls ldp router-id Loopback0
!
interface f0/0
	ip address 10.1.10.1 255.255.255.252
	no shutdown
	mpls ip
!
interface f0/1
	ip address 10.1.20.1 255.255.255.252
	no shutdown
	mpls ip
!
interface f1/0
	ip address 10.1.50.1 255.255.255.252
	no shutdown
	mpls ip
!
router eigrp 100
	network 10.255.255.1 0.0.0.0
	network 10.1.10.0 0.0.0.3
	network 10.1.20.0 0.0.0.3
	network 10.1.50.0 0.0.0.3
	no auto-summary
!
mpls ip propagate-ttl

~~~

## R2

~~~python
configure terminal
!
mpls ip 
mpls label protocol ldp
!
interface loopback0
	ip address 10.255.255.2 255.255.255.255
	no shutdown
!
mpls ldp router-id loopback0
!
interface f0/0
	ip address 10.1.10.2 255.255.255.252
	no shutdown 
	mpls ip
!
interface f0/1
	ip address 10.1.40.2 255.255.255.252
	no shutdown
	mpls ip
!
router eigrp 100
	network 10.255.255.2 0.0.0.0
	network 10.1.10.0 0.0.0.3
	network 10.1.40.0 0.0.0.3
	no auto-summary
!
mpls ip propagate-ttl

~~~

## R3

~~~python
configure terminal
!
mpls ip 
mpls label protocol ldp 
!
interface loopback0
	ip address 10.255.255.3 255.255.255.255
	no shutdown
!
mpls ldp router-id loopback0
!
interface f0/0
	ip address 10.1.20.2 255.255.255.252
	no shutdown
	mpls ip
!
interface f0/1
	ip address 10.1.30.1 255.255.255.252
	no shutdown
	mpls ip
!
router eigrp 100
	network 10.255.255.3 0.0.0.0
	network 10.1.20.0 0.0.0.3
	network 10.1.30.0 0.0.0.3
	no auto-summary
!
mpls ip propagate-ttl
~~~

## PE-R4

~~~python
configure terminal
!
!
mpls ip
mpls label protocol ldp
!
!
interface Loopback0
	ip address 10.255.255.4 255.255.255.255
!
mpls ldp router-id Loopback0
!
!
interface f0/0
	ip address 10.1.40.1 255.255.255.252
	no shutdown
	mpls ip
!
!
interface f0/1
	ip address 172.16.1.1 255.255.255.252
	no shutdown
!
!
interface f1/0
	ip address 172.16.2.1 255.255.255.252
	no shutdown
!
!
router eigrp 100
	network 10.255.255.4 0.0.0.0
	network 10.1.40.1 0.0.0.3
	no auto-summary
!
!
ip vrf customerA
	rd 100:1
	route-target export 100:1
	router-target import 100:1
!
!
ip vrf customerB
	rd 200:1
	route-target export 200:1
	route-target import 200:1
!
!
router bgp 400
	neigh 10.255.255.5 remote-as 400
	neigh 10.255.255.5 update-source Loopback0
	neigh 10.255.255.6 remote-as 400
	neigh 10.255.255.6 update-source Loopback0
	no auto-summary
	!
	address-family vpnv4
		neigh 10.255.255.5 activate
		neigh 10.255.255.5 send-community both
		neigh 10.255.255.5 next-hop-self
		neigh 10.255.255.6 activate
		neigh 10.255.255.6 send-community both
		neigh 10.255.255.6 next-hop-self
	!
	address-family ipv4 vrf customerA
		redistribute connected
		redistribute static
		neigh 172.16.1.2 remote-as 65100
		neigh 172.16.1.2 activate
	!
	address-family ipv4 vrf customerB
		redistribute connected
		redistribute static
		neigh 172.16.2.2 remote-as 65100
		neigh 172.16.2.2 activate
	!
!
!
int f0/1
	ip vrf forwarding customerA
	ip address 172.16.1.1 255.255.255.252
	no shutdown
	mpls ip
!
int f1/0
	ip vrf forwarding customerB
	ip address 172.16.2.1 255.255.255.252
	no shutdown
	mpls ip
!
!
mpls ip propagate-ttl
~~~

## PE-R5

~~~python
configure terminal
!
mpls ip 
mpls label protocol ldp
!
interface loopback0
	ip address 10.255.255.5 255.255.255.255
	no shutdown
!
mpls ldp router-id loopback0
!
interface f0/0
	ip address 10.1.30.2 255.255.255.252
	no shutdown
	mpls ip
!
interface f0/1
	ip address 172.16.2.1 255.255.255.252
	no shutdown
	mpls ip
!
interface f1/0
	ip address 172.16.4.1 255.255.255.252
	no shutdown
	mpls ip
!
router eigrp 100
	network 10.255.255.5 0.0.0.0
	network 10.1.30.0 0.0.0.3
	no auto-summary
!
ip vrf customerA
	rd 100:1
	route-target export 100:1
	route-target import 100:1
!
ip vrf customerB
	rd 200:1
	route-target export 200:1
	route-target import 200:1
!
route bgp 400
	neighbor 10.255.255.4 remote-as 400
	neighbor 10.255.255.4 update-source loopback0
	neighbor 10.255.255.6 remote-as 400
	neighbor 10.255.255.6 update-source loopback0
	
	address-family vpnv4
		neigh 10.255.255.4 activate
		neigh 10.255.255.4 send-community both
		neigh 10.255.255.4 next-hop-self
		neigh 10.255.255.6 activate
		neigh 10.255.255.6 send-community both
		neigh 10.255.255.6 next-hop-self
	
	address-family ipv4 vrf customerA
		redistribute connected
		redistribute static
		neighbor 172.16.2.2 remote-as 65100
		neighbor 172.16.2.2 activate
	
	address-family ipv4 vrf customerB
		redistribute connected
		redistribute static
		neighbor 172.16.4.2 remote-as 65100
		neighbor 172.16.4.2 activate
	!
interface f0/1
	ip vrf forwarding customerA
	ip address 172.16.2.1 255.255.255.252
	no shutdown
	mpls ip
!
interface f1/0
	ip vrf forwarding customerB
	ip address 172.16.4.1 255.255.255.252
	no shutdown
	mpls ip
!
mpls ip propagate-ttl
~~~

## PE-R6

~~~python
configure terminal
!
mpls ip
mpls label protocol ldp
!
interface loopback0
	ip address 10.255.255.6 255.255.255.255
	no shutdown
!
mpls ldp router-id loopback0
!
interface f0/0
	ip address 10.1.50.2 255.255.255.252
	no shutdown
	mpls ip
!
interface f0/1
	ip address 172.16.3.1 255.255.255.252
	no shutdown
	mpls ip
!
router eigrp 100
	network 10.255.255.6 0.0.0.0
	network 10.1.50.0 0.0.0.3
	no auto-summary
!
ip vrf customerA
	rd 100:1
	route-target import 100:1
	route-target export 100:1
!
route bgp 400
	route bgp 400
	neighbor 10.255.255.4 remote-as 400
	neighbor 10.255.255.4 update-source loopback0
	neighbor 10.255.255.5 remote-as 400
	neighbor 10.255.255.5 update-source loopback0
	
	address-family vpnv4
		neigh 10.255.255.4 activate
		neigh 10.255.255.4 send-community both
		neigh 10.255.255.4 next-hop-self
		neigh 10.255.255.5 activate
		neigh 10.255.255.5 send-community both
		neigh 10.255.255.5 next-hop-self
	
	address-family ipv4 vrf customerA
		redistribute connected
		redistribute static
		neighbor 172.16.3.2 remote-as 65100
		neighbor 172.16.3.2 activate
	!
interface f0/1
	ip vrf forwarding customerA
	ip address 172.16.3.1 255.255.255.252
	no shutdown 
	mpls ip
!
mpls ip propagate-ttl
	
~~~

******

## CE-CustA-1

~~~python
configure terminal
!
interface loopback0
	ip address 192.168.1.1 255.255.255.0
	no shutdown
!
interface f0/0
	ip address 172.16.1.2 255.255.255.252
	no shut
!
ip route 0.0.0.0 0.0.0.0 172.16.1.1
!
router bgp 65100
	network 192.168.1.0 mask 255.255.255.0
	network 172.16.1.0 mask 255.255.255.252
	neighbor 172.16.1.1 remote-as 400
	no auto

~~~
