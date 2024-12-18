### Topologia

![practica6](images/image5.png)

******
## AS 65000

*****
*Importante:* En este caso estamos utilizando confederaciones porque podemos por decirlo de una manera: "Necesito control especifico por regiones/clientes/servicios" y no route reflector por que es mas general, mas por encima, es decir: "Solo necesito que todos se comuniquen".

### ProviderSF


~~~python

ProviderSF(config)# interface f0/0
ProviderSF(config-int)# ip address 10.1.1.1 255.255.255.0
ProviderSF(config-int)# no shutdown

ProviderSF(config)# interface f0/1
ProviderSF(config-int)# ip address 10.1.2.1 255.255.255.0
ProviderSF(config-int)# no shutdown

ProviderSF(config)# interface s0/0
ProviderSF(config-int)# ip address 192.168.1.1 255.255.255.252
ProviderSF(config-int)# no shutdown

ProviderSF(config)# interface s0/1
ProviderSF(config-int)# ip address 172.16.0.1 255.255.255.252
ProviderSF(config-int)# no shut

ProviderSF(config)# router bgp 65000
ProviderSF(config-router)# no synchronization
ProviderSF(config-router)# bgp router-id 1.0.0.0
ProviderSF(config-router)# bgp log-neighbor-changes
ProviderSF(config-router)# bgp bestpath med missing-as-worst

ProviderSF(config-router)# network 10.1.1.0 mask 255.255.255.0
ProviderSF(config-router)# network 10.1.2.0 mask 255.255.255.0

ProviderSF(config-router)# neighbor 172.16.0.2 remote-as 65000
ProviderSF(config-router)# neighbor 172.16.0.2 next-hop-self
ProviderSF(config-router)# neighbor 192.168.1.2 remote-as 65001
ProviderSF(config-router)# no auto-summary
~~~

*Nota: Aquí no anunciamos las rutas /30 porque son point to point, solo son para comunicación entre routers y compartir ahí sus rutas.*

*Importante: `next-hop-self` le avisamos a nuestro vecino que las rutas que anunciamos pueden ser alcanzadas por nosotros mismos.*


### ProviderNY

~~~python
ProviderNY(config)# interface f0/0
ProviderNY(config-if)# ip address 10.2.1.1. 255.255.255.0
ProviderNY(config-if)# no shutdown

ProviderNY(config)# interface f0/1
ProviderNY(config-if)# ip address 10.2.2.1 255.255.255.0
ProviderNY(config-if)# no shutdown

ProviderNY(config)# interface f1/0
ProviderNY(config-if)# ip address 1.1.1.1 255.255.255.0
ProviderNY(config-if)# no shutdown

ProviderNY(config)# interface s0/0
ProviderNY(config-if)# ip address 192.168.2.1 255.255.255.252
ProviderNY(config-if)# no shutdown

ProviderNY(config-if)# interface s0/1
ProviderNY(config-if)# ip address 172.16.0.2 255.255.255.252
ProviderNY(config-if)# no shutdown

ProviderNY(config)# router bgp 65000
ProviderNY(config-router)# router-id 2.0.0.0
ProviderNY(config-router)# no synchronization
ProviderNY(config-router)# bgp log-neighbor-changes
ProviderNY(config-router)# bgp bestpath med missing-as-worst 

ProviderNY(config-router)# network 10.2.1.0 mask 255.255.255.0
ProviderNY(config-router)# network 10.2.2.0 mask 255.255.255.0
ProviderNY(config-router)# network 1.1.1.0 mask 255.255.255.0


ProviderNY(config-router)# neighbor 172.16.0.1 remote-as 65000
ProviderNY(config-router)# neighbor 172.16.0.1 next-hop-self 
ProviderNY(config-router)# neighbor 192.168.2.2 remote-as 65001
ProviderNY(config-router)# no auto-summary
~~~

*Nota:* El comando `bgp bestpath med missing-as-worst` se implementa para rutas sin MED no las trate como la mejor ruta (MED = 0), si no, que las trate como la peor (MED = 4,294,967,295 )

*Por cierto boludo, ese comando me lo di Claude, es un buen tip*


****
## AS 65001

*****

### CustomerSF

*Nota:* Aquí en el AS 65001 (Corrección: No solo es aquí, sino en el AS 65000 ) es mejor manejar las rutas con iBGP ya que si lo manejamos con otro protocolo de ruteo interno, como OSPF o EIGRP no son tan compatibles con políticas avanzadas como BGP.  

~~~python

CustomerSF(config)# interface f0/0
CustomerNY(config-int)# ip address 192.168.3.1 255.255.255.252
CustomerNY(config-int)# no shutdown

CustomerNY(config)# interface f0/1
CustomerNY(config)# ip address 192.168.10.1 255.255.255.0
CustomerNY(config)# no shutdown

CustomerNY(config)# interface s0/0
CustomerNY(config-int)# ip address 192.168.1.2 255.255.255.252
CustomerNY(config-int)# no shutdown

CustomerSF(config)# router bgp 65001
CustomerSF(config-router)# router-id 3.0.0.0
CustomerSF(config-router)# network 192.168.10.0 mask 255.255.255.0
CustomerSF(config-router)# neighbor 192.168.1.1 remote-as 65000
CustomerSF(config-router)# neighbor 192.168.3.2 remote-as 65001
CustomerSF(config-router)# neighbor 192.168.3.2 next-hop-self

# ------------ * -------------

ip prefix-list C2 seq 5 permit 10.1.1.0/24
ip prefix-list C3 seq 5 permit 10.1.2.0/24
!
route-map EBGP-ProviderSF_in permit 10
 match ip address prefix-list C2 C3
 set local-preference 300
route-map EBGP-ProviderSF_in permit 20
 set local-preference 200
!
router bgp 65001
 neighbor 192.168.1.1 route-map EBGP-ProviderSF_in in
 
# ------------ * -------------

ip prefix-list Net10 seq 5 permit 192.168.10.0/24
!
route-map EBGP-ProviderSF_out permit 10
	match ip address prefix-list Net10
	set metric 200
route-map EBGP-ProviderSF_out permit 20
	set metric 300
!
router bgp 65001
	neighbor 192.168.1.1 route-map EBGP-ProviderSF_out out

~~~

*Explicación:* Cuando las redes de AS 65001 lleguen y concidan con nuestros prefijos 10.1.2.0/24 y 10.1.1.0/24 se le asigne una preferencia local y se manden siempre por el router CustomerSF y todas las demás se le agrega una preferencia menor. 

Casi el mismo caso para que las red 192.168.10.0/24, solo que aquí se especifica que la red 192.168.10.0/24 se le asignara un MED bajo para que el proveedor siempre tome esa ruta cuando quiera llegar a esa red.  

### CustomerNY

~~~python
CustomerNY(config)# interface f0/0
CustomerNY(config-if)# ip address 192.168.3.2 255.255.255.252
CustomerNY(config-if)# no shutdown

CustomerNY(config)# interface f0/1
CustomerNY(config-if)# ip address 192.168.20.1 255.255.255.0
CustomerNY(config-if)# no shutdown

CustomerNY(config)# interface s0/0
CustomerNY(config-if)# ip address 192.168.2.2 255.255.255.252
CustomerNY(config-if)# no shutdown

CustomerNY(config)# router bgp 65001
CustomerNY(config-router)# router-id 4.0.0.0
CustomerNY(config-router)# network 192.168.20.0 mask 255.255.255.0
CustomerNY(config-router)# neighbor 192.168.2.1 remote-as 65000
CustomerNY(config-router)# neighbor 192.168.3.1 remote-as 65001
CustomerNY(config-router)# neighbor 192.168.3.1 next-hop-self

# ---------- * ------------
ip prefix-list C4 seq 5 permit 10.2.1.0/24
ip prefix-list C5 seq 5 permit 10.2.2.0/24
!
route-map EBGP-ProviderNY_in permit 10
 match ip address prefix-list C4 C5
 set local-preference 300
route-map EBGP-ProviderNY_in permit 20
 set local-preference 200
!
router bgp 65001
 neighbor 192.168.2.1 route-map EBGP-ProviderNY_in in

# ---------- * ------------
ip prefix-list Net20 seq 5 permit 192.168.20.0/24
!
route-map EBGP-ProviderNY_out permit 10
 match ip address prefix-list Net20
 set metric 200
route-map EBGP-ProviderNY_out permit 20
 set metric 300
!
router bgp 65001
 neighbor 192.168.2.1 route-map EBGP-ProviderNY_out out
~~~

*Otra cosa: Recuerda que local preference entre mas grande es mejor, pero en MED es peor si es mas grande.*
