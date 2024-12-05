## Practica 1 - FHRP

- En esta práctica, nos enfocaremos en la configuración y optimización de un entorno de red robusto y resiliente mediante el uso de protocolos de redundancia FHRP (First Hop Redundancy Protocol). Estos protocolos son esenciales en redes empresariales donde la alta disponibilidad y la continuidad del servicio son cruciales, ya que permiten que múltiples routers trabajen en conjunto para asegurar que el tráfico de la red siempre tenga un camino disponible hacia su destino, incluso en caso de fallos de hardware o desconexiones imprevistas.
- Nuestro objetivo principal es llevar a cabo la implementación y configuración de los protocolos de redundancia HSRP, VRRP, y GLBP en una serie de dispositivos de red clave (ESW1, ESW2, ESW3, y ESW4). 
- A través de esta configuración, buscaremos asegurar que la red pueda mantener su operatividad incluso si un router principal falla, permitiendo que otro dispositivo asuma de inmediato y automáticamente el rol de enrutador principal. Esto minimiza el tiempo de inactividad y evita que los usuarios finales experimenten interrupciones en el servicio.

**Protocolos Utilizados:**

- **HSRP (Hot Standby Router Protocol):** Protocolo de Cisco que permite que varios routers trabajen juntos para ofrecer una puerta de enlace redundante. Uno de los routers se convierte en el activo, y si falla, otro toma su lugar automáticamente.

- **VRRP (Virtual Router Redundancy Protocol):** Estándar abierto similar a HSRP, donde un grupo de routers comparte una IP virtual. El router con mayor prioridad se convierte en el maestro, y otro asume el control si el maestro falla.

- **GLBP (Gateway Load Balancing Protocol):** Protocolo de Cisco que no solo ofrece redundancia, sino también balanceo de carga entre varios routers, distribuyendo el tráfico para mejorar la eficiencia de la red.

***
#### Tabla de direccionamiento:

| Dispositivos |       IP's       | Interafaz | Default Gateway |
| :----------: | :--------------: | :-------: | :-------------: |
|    Vpcs1     |   10.0.3.3/24    |    e0     |   10.0.3.254    |
|    Vpcs2     |   20.0.3.3/24    |    e0     |   20.0.3.254    |
|    Vpcs3     |   30.0.3.3/24    |    e0     |   20.0.3.254    |
|     ESW1     |   10.0.3.1/24    |   F1/0    |        -        |
|     ESW1     |    1.1.3.1/30    |   F2/0    |        -        |
|     ESW2     |   10.0.3.2/24    |   F1/0    |        -        |
|     ESW2     |   20.0.3.1/24    |   F0/0    |        -        |
|     ESW2     |    2.2.3.1/30    |   F2/0    |        -        |
|     ESW3     |   20.0.3.2/24    |   F1/0    |        -        |
|     ESW3     |   30.0.3.1/24    |   F0/0    |        -        |
|     ESW3     |    3.3.3.1/30    |   F2/0    |        -        |
|     ESW4     |   30.0.3.2/24    |   F1/0    |        -        |
|     ESW4     |    4.4.3.1/30    |   F2/0    |        -        |
|    R1 L0     | 100.100.100.1/24 |           |        -        |
|      R1      |    1.1.3.2/30    |   F0/0    |        -        |
|      R1      |    2.2.3.2/30    |   F0/1    |        -        |
|      R1      |    3.3.3.2/30    |   F1/0    |        -        |
|      R1      |    4.4.3.2/30    |   F2/0    |        -        |
***
#### Configuración de VPC's:

Configuramos las VPC's (Virtual Private Cloud) asignándoles sus direcciones IP y su máscara de red, que en este caso será `/24`, junto a esto pusimos también la Default Gateway, la cual será la vIP (virtual IP) que emplearemos en la configuración de los tres protocolos de redundancia.
Además de guardar estas direcciones para evitar ponerlas manualmente cada vez que arranquemos GNS3, esto lo hicimos con el comando `save`:

- VPC1:

~~~bash
ip 10.0.3.3/24 10.0.3.254
save
~~~

- VPC2:

~~~bash
ip 20.0.3.3/24 20.0.3.254
save
~~~

- VPCS3:

~~~bash
ip 30.0.3.3/24 30.0.3.254
save
~~~

***
### Configuración de los routers ESW:

#### ESW1:

**Comandos de configuración básica:**

- Aquí establecemos el nombre del dispositivo. 
- Luego, configura la interfaz `f2/0` con una dirección especifica, que será a la que conectaremos con el router principal R1, igualmente le establecemos el ancho de banda y habilitamos la interfaz.
- Esta configuración inicial será igual en el resto de equipos ESW, por lo que no la volveremos a detallar:

~~~bash
ena
conf t
hostname ESW1

int f2/0
ip address 1.1.3.1 255.255.255.252
bandwidth 10000
no shutdown
~~~

**Configuración de HSRP:**

- Configuramos la interfaz `f1/0` con una dirección IP y la habilitamos, esta irá conectada a la VPC1.
- Luego, establecemos HSRP en la versión 2, asignamos la vIP, establecimos una prioridad de 110, habilitamos el ``preempt`` para que el router con mayor prioridad tome el control (en este caso este), y configuramos el seguimiento de la interfaz `f2/0` para reducir la prioridad si falla, es decir, si este router falla se tomará como prioritario el segundo router con HSRP:

~~~java

int f1/0
ip address 10.0.3.1 255.255.255.0
no shut
standby version 2
standby 13 ip 10.0.3.254
standby 13 priority 110
standby 13 preempt
standby 13 track f2/0 20
standby 13 time 1 4
~~~

***
#### ESW2:

**Comandos de configuración básica:**

~~~python
ena
conf t
hostname ESW2

int f2/0
ip address 2.2.3.1 255.255.255.252
bandwidth 20000
no shutdown
~~~

**Configuración de HSRP**

- Configuramos la interfaz `f1/0` con una dirección IP y la habilitamos, esta se conecta a la misma red de la VPC1.
- Establecemos HSRP en la versión 2, asignamos la vIP, establecemos una prioridad de 105, y habilitamos el `preempt` para que, si el router con mayor prioridad falla, este tome el control:

~~~bash
int f1/0
ip address 10.0.3.2 255.255.255.0
no shut
standby version 2
standby 13 ip 10.0.3.254
standby 13 priority 105
standby 13 preempt
standby 13 time 1 4
~~~

**Configuración de VRRP:**

- Configuramos la interfaz `f0/1` (la cual se conecta a la VPC2) con una dirección IP y la habilitamos.
- Configuramos VRRP para proveer redundancia, asignamos una vIP, establecemos la prioridad en 110 y habilitamos el `preempt`, en este caso este mismo equipo actuará como el de mayor prioridad.
- También configuramos el seguimiento de la interfaz `f2/0` para que, si falla, se reduzca la prioridad:

~~~bash
int f0/1
ip address 20.0.3.1 255.255.255.0
no shut
vrrp 13 ip 20.0.3.254
vrrp 13 priority 110
vrrp 13 preempt
track 13 int f2/0 line-protocol
exit
int f0/1
vrrp 13 track 13 decrement 10
~~~

***

#### ESW3:

**Comandos de configuración básica:**

~~~bash
ena
conf t
hostname ESW3

int f2/0
ip address 3.3.3.1 255.255.255.252
bandwidth 30000
no shutdown
~~~

**Configuración de VRRP:**

- Configuramos la interfaz `f1/0` con una dirección IP, esta interfaz estará enlazada a  la VPC2.
- Establecemos VRRP para proveer redundancia, asignamos la vIP, establecemos una prioridad de 105, y habilitamos el `preempt`, este actuará como el router de menor prioridad:

~~~bash
int f1/0
ip address 20.0.3.2 255.255.255.0
no shut
vrrp 13 ip 20.0.3.254
vrrp 13 priority 105
vrrp 13 preempt
~~~

**Configuración de GLBP:**

- Configuramos la interfaz `f0/1` con una dirección IP, esta estará conectada con la VPC3.
- Configuramos GLBP para balancear la carga de la red, asignamos la vIP, habilitamos el `preempt` y establecemos una prioridad de 110, este será nuestro router de mayor prioridad para este protocolo:

~~~bash
int f0/1
ip address 30.0.3.1 255.255.255.0
no shut
glbp 13 ip 30.0.3.254
glbp 13 preempt
glbp 13 priority 110
~~~

***

#### ESW4:

**Comandos de configuración básica:**

~~~bash
ena
conf t
hostname ESW4

int f2/0
ip address 4.4.3.1 255.255.255.252
bandwidth 40000
no shutdown
~~~

**Configuración de GLBP:**

- Configuramos la interfaz `f1/0` con una dirección IP, conectada a la VPC3.
- Configuramos GLBP para balancear la carga de la red, asignamos la vIP, habilitamos el `preempt` y establecemos una prioridad de 105, será nuestro router de menor prioridad:

~~~bash
int f1/0
ip address 30.0.3.2 255.255.255.0
no shut
glbp 13 ip 30.0.3.254
glbp 13 preempt
glbp 13 priority 105
~~~
***

#### R1:

**Configuración de la loopback y las interfaces conectadas a los equipos ESW:**

- Configuramos la interfaz de loopback para el ruteo y las interfaces conectadas a los equipos ESW1, ESW2, ESW3 y ESW4 con las direcciones IP correspondientes. 
- También establecemos el mismo ancho de banda que asignamos a cada uno de los equipos anteriores en la configuración principal, de esta forma evitaremos problemas de comunicación en las interfaces:

~~~bash
ena 
conf t
hostname R1
int loopback 0
ip address 100.100.100.1 255.255.255.0
no shutdown
int f0/0
ip address 1.1.3.2 255.255.255.252
bandwidth 10000
no shutdown
int f1/0
ip address 2.2.3.2 255.255.255.252
bandwidth 20000
no shutdown
int f0/1
ip address 3.3.3.2 255.255.255.252
bandwidth 30000
no shutdown
int f2/0
ip address 4.4.3.2 255.255.255.252
bandwidth 40000
no shutdown
exit
exit
wr
~~~

**Configuración de GIRP para aplicar el ruteo entre equipos de distintas redes:**

- Configuramos el protocolo EIGRP, especificando las redes a las cuales tenemos acceso desde este equipo, y deshabilitamos la auto-sumarización:

~~~bash
ena
configure terminal
router EIGRP 13
net 1.1.3.0 0.0.0.3
net 2.2.3.0 0.0.0.3
net 3.3.3.0 0.0.0.3
net 4.4.3.0 0.0.0.3
net 100.100.100.0 0.0.0.255
no auto
exit
exit
wr
~~~

***

### EIGRP en los equipos ESW:

- Configuraremos solamente las redes a las que los equipos tienen alcance de primera instancia, esto en cada uno de los equipos, para que cuando el protocolo se active, los demás equipos conozcan las redes faltantes y puedan comunicarse mutuamente.

**Para ESW1:**

~~~bash
exit
router eigrp 13 
network 10.0.3.0 0.0.0.255 
network 1.1.3.0 0.0.0.3 
no auto
~~~

**Para ESW2:**

~~~bash
exit
router eigrp 13 
network 10.0.3.0 0.0.0.255 
network 2.2.3.0 0.0.0.3 
network 20.0.3.0 0.0.0.255 
no auto
~~~

**Para ESW3:**

~~~bash
exit
router eigrp 13 
network 20.0.3.0 0.0.0.255 
network 3.3.3.0 0.0.0.3 
network 30.0.3.0 0.0.0.255 
no auto
~~~

**Para ESW4:**

~~~bash
exit
router eigrp 13 
network 30.0.3.0 0.0.0.255 
network 4.4.3.0 0.0.0.3 
no auto
~~~
