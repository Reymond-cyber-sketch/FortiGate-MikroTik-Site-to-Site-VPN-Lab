# FortiGate-MikroTik-Site-to-Site-VPN-Lab

Laboratorio de VPN Site-to-Site entre FortiGate y MikroTik en GNS3, con red de usuarios, servidor HTTPS, NAT y demostración de conectividad mediante túnel IPsec.

## Video de demostración

Video del funcionamiento completo del laboratorio:

(https://youtu.be/pYwDLJvQodY)
---

## Descripción

En este laboratorio se implementó una VPN Site-to-Site entre un FortiGate y un MikroTik utilizando GNS3.

El objetivo principal fue permitir la comunicación entre una red de usuarios y una red donde se encuentra un servidor web, utilizando un túnel VPN IPsec.

También se configuró un dispositivo MikroTik para simular el proveedor de Internet entre ambos extremos de la VPN.

Durante las pruebas se comprobó que el equipo de usuario podía comunicarse con el servidor web cuando la VPN estaba activa. Al desactivar el túnel VPN, la comunicación dejó de funcionar.

---

## Objetivo del laboratorio

Configurar una infraestructura que permita la comunicación entre una red de usuarios y un servidor remoto mediante una VPN Site-to-Site.

En el laboratorio se trabajó con:

- FortiGate
- MikroTik RouterOS
- GNS3
- Docker
- DHCP
- NAT
- VPN IPsec
- Servidor HTTPS
- VLAN
- Traceroute

---

## Topología

La topología utilizada fue la siguiente:

```text
USER-PC
   |
SW-LAB
   |
FGT-USER
   |
ISP
   |
VPN-PEER
   |
WEB-SERVER
```

El dispositivo ISP también se encuentra conectado al nodo NAT de GNS3 para permitir salida a Internet.

---

## Direccionamiento IP

### Red de usuarios

```text
Red: 10.24.96.0/25
Gateway: 10.24.96.1
Rango DHCP: 10.24.96.10 - 10.24.96.120
```

### Red del servidor

```text
Red: 10.24.96.128/28
Gateway: 10.24.96.129
WEB-SERVER: 10.24.96.130
```

### Red entre FGT-USER e ISP

```text
Red: 198.51.100.96/30
ISP: 198.51.100.97
FGT-USER: 198.51.100.98
```

### Red entre ISP y VPN-PEER

```text
Red: 203.0.113.96/30
ISP: 203.0.113.97
VPN-PEER: 203.0.113.98
```

### Administración del FortiGate

```text
FGT-USER port3: 192.168.178.2/24
```

---

## Configuración del FortiGate

El FortiGate utilizado en el laboratorio fue configurado como gateway de la red de usuarios y como uno de los extremos de la VPN.

### Interfaces

```text
port1 - WAN
198.51.100.98/30

port2 - USERS
10.24.96.1/25

port3 - MGMT
192.168.178.2/24
```

### DHCP

El FortiGate proporciona direcciones IP automáticamente a los equipos de la red de usuarios.

```text
Gateway: 10.24.96.1
Rango: 10.24.96.10 - 10.24.96.120

DNS:
8.8.8.8
1.1.1.1
```

---

## VPN Site-to-Site

Se configuró un túnel IPsec entre el FortiGate y el MikroTik VPN-PEER.

### FortiGate

```text
Nombre del túnel: VPN-TO-WEB
Gateway remoto: 203.0.113.98
Red local: 10.24.96.0/25
Red remota: 10.24.96.128/28
IKE: IKEv2
Encryption: DES
Authentication: SHA256
DH Group: 14
PFS: Disabled
```

La clave precompartida utilizada para la VPN no fue agregada al repositorio.

---

## Configuración de VPN-PEER

El MikroTik VPN-PEER funciona como el segundo extremo del túnel VPN.

### Interfaces

```text
ether1 - WAN
203.0.113.98/30

ether2 - LAN WEB
10.24.96.129/28
```

### Ruta por defecto

```text
0.0.0.0/0
Gateway: 203.0.113.97
```

### Parámetros de VPN

```text
Peer remoto: 198.51.100.98
Red local: 10.24.96.128/28
Red remota: 10.24.96.0/25
IKE: IKEv2
Encryption: DES
Authentication: SHA256
DH Group: modp2048
PFS: Disabled
```

También se configuró una regla para evitar NAT en el tráfico que viaja entre las dos redes privadas.

---

## Configuración del ISP

El dispositivo ISP fue configurado utilizando MikroTik RouterOS.

Su función fue simular el proveedor de Internet entre los dos extremos de la VPN.

### Interfaces

```text
Hacia FGT-USER:
198.51.100.97/30

Hacia VPN-PEER:
203.0.113.97/30

Hacia Internet:
DHCP Client mediante el nodo NAT de GNS3
```

También se configuró NAT masquerade para permitir salida a Internet.

---

## Configuración del switch

El switch fue utilizado para conectar la red de usuarios y la red de administración del FortiGate.

Se utilizaron las siguientes VLAN:

```text
VLAN 10 - USERS
VLAN 99 - MGMT
```

Puertos utilizados:

```text
Ethernet0
Access VLAN 10
USER-PC

Ethernet2
Access VLAN 10
FGT-USER port2

Ethernet4
Access VLAN 99
FGT-USER port3
```

---

## WEB-SERVER

El servidor web utilizado en el laboratorio tiene la siguiente configuración:

```text
IP: 10.24.96.130/28
Gateway: 10.24.96.129
Servicio: HTTPS
Puerto: 443
```

Para probar el servidor se utilizó:

```bash
curl -k https://10.24.96.130/
```

El servidor respondió correctamente mostrando la página configurada para el laboratorio.

---

## Pruebas realizadas

### Obtener IP por DHCP

Desde USER-PC se utilizó:

```bash
udhcpc -i eth0 -q -n
```

El equipo recibió una dirección perteneciente a la red:

```text
10.24.96.0/25
```

### Prueba hacia el gateway

```bash
ping -c 4 10.24.96.1
```

La prueba fue exitosa.

### Prueba hacia el WEB-SERVER

```bash
ping -c 4 10.24.96.130
```

Resultado:

```text
4 packets transmitted
4 packets received
0% packet loss
```

Esto confirmó que existía comunicación entre la red de usuarios y la red del servidor mediante la VPN.

---

## Prueba HTTPS

Se utilizó:

```bash
curl -k https://10.24.96.130/
```

El servidor respondió con:

```text
WEB SERVER - Laboratorio FortiGate

Servidor HTTPS funcionando correctamente.

IP: 10.24.96.130
```

---

## Traceroute

También se realizó una prueba de traceroute:

```bash
traceroute 10.24.96.130
```

Resultado obtenido:

```text
1  10.24.96.1
2  * * *
3  10.24.96.130
```

Esto permitió verificar la ruta desde el USER-PC hasta el servidor remoto.

---

## Prueba con VPN activa

Con el túnel VPN activo se realizó:

```bash
ping -c 4 10.24.96.130
```

Resultado:

```text
4 packets transmitted
4 packets received
0% packet loss
```

El USER-PC pudo comunicarse correctamente con el WEB-SERVER.

---

## Prueba con VPN inactiva

Para demostrar que la comunicación dependía del túnel VPN, se desactivó temporalmente el peer en VPN-PEER.

Después se ejecutó nuevamente:

```bash
ping -c 4 10.24.96.130
```

Resultado:

```text
4 packets transmitted
0 packets received
100% packet loss
```

Esto demostró que sin la VPN activa no existía comunicación entre las dos redes.

---

## Reactivación de la VPN

Después de volver a habilitar el peer VPN se realizó nuevamente:

```bash
ping -c 4 10.24.96.130
```

Resultado:

```text
4 packets transmitted
4 packets received
0% packet loss
```

La comunicación se restauró correctamente.

---

## Estructura del repositorio

```text
FortiGate-MikroTik-Site-to-Site-VPN-Lab/
|
|-- README.md
|
|-- configs/
|   |-- FGT-USER.txt
|   |-- VPN-PEER-MikroTik.txt
|   |-- ISP-MikroTik.txt
|
|-- images/
|   |-- 01-topologia.png
|   |-- 02-fortigate-interfaces.png
|   |-- 03-fortigate-vpn-active.png
|   |-- 04-mikrotik-vpn.png
|   |-- 05-ping-vpn-active.png
|   |-- 06-https-server.png
|   |-- 07-traceroute.png
|   |-- 08-vpn-inactive.png
|   |-- 09-ping-vpn-inactive.png
|
|-- scripts/
    |-- comandos-pruebas.txt
```

---

## Conclusión

En este laboratorio se logró establecer correctamente una VPN Site-to-Site entre un FortiGate y un MikroTik.

La red de usuarios pudo comunicarse con el servidor web ubicado en una red diferente utilizando el túnel IPsec.

También se comprobó el funcionamiento de DHCP, NAT, HTTPS y traceroute.

Finalmente, al desactivar la VPN se perdió la comunicación entre las dos redes y al volver a activarla la conectividad fue restaurada.

Con esto se comprobó que la comunicación entre USER-PC y WEB-SERVER dependía directamente del túnel VPN configurado.

---

## Autor

Reymond Daniel Guerrero Cruz

Matrícula: 2024-0963
