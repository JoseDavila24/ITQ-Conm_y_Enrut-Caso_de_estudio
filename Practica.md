---
Maestro: LUIS RAYMUNDO ARAN SANCHEZ
---
## Comandos de implementación en la zona A
## 1. Subnetear
| Vlan | Nombre de red | Mascara         | Prefijo | Broadcast    | Rango                       |
| ---- | ------------- | --------------- | ------- | ------------ | --------------------------- |
| 301  | 172.16.8.208  | 255.255.255.240 | /28     | 172.16.8.223 | 172.16.8.209 - 172.16.8.222 |
| 302  | 172.16.8.224  | 255.255.255.240 | /28     | 172.16.8.239 | 172.16.8.225 - 172.16.8.238 |
| 303  | 172.16.8.240  | 255.255.255.240 | /28     | 172.16.8.255 | 172.16.8.241 - 172.16.8.254 |
| 304  | 172.16.9.0    | 255.255.255.240 | /28     | 172.16.9.15  | 172.16.9.1 - 172.16.9.14    |
| 305  | 172.16.9.16   | 255.255.255.240 | /28     | 172.16.9.31  | 172.16.9.17 - 172.16.9.30   |
## 2. Apagar la interfaces de los switches (Recomendado)

```cisco
interface range fastEthernet 0/1 - 24, gigabitEthernet 0/1 - 2
shutdown
```
## 3. Direccionamiento de PCs
Ingresar en cada PC la IP, mascara y DG

> Nota: Para los AP se usaron los AP-PT-AC y laptops con para la VLAN 305 se usaron laps con el modulo WPC300N para conectarse por wi-fi y se optorgo IPs de forma dinamica por DHCP con el router
## 4. VTP (server + Clientes)

4.1 Configurar el servidor y crear vlans
```cisco
vtp version 2
vtp domain equipo5
vtp password equipo5
vtp mode server

vlan 301
exit
vlan 302
exit
vlan 303
exit
vlan 304
exit
vlan 305
exit
vlan 11
exit
vlan 12
exit
vlan 13
exit
```

4.2 Configurar en ambos clientes
```cisco
vtp version 2
vtp domain equipo5
vtp password equipo5
vtp mode client
```
## 5. Spanning Tree (PVST)

Raiz
```cisco
spanning-tree mode pvst
spanning-tree vlan 301-305,11-13 priority 0
```

Segundo
```cisco
spanning-tree mode pvst
spanning-tree vlan 301-305,11-13 priority 4096
```

Tercero
```cisco
spanning-tree mode pvst
spanning-tree vlan 301-305,11-13 priority 8192
```

## 6. Puertos troncales
Ejemplo general para puerto troncal entre dos switches

```cisco
interface <interfaz>
switchport mode trunk
switchport trunk native vlan <id>
no shutdown
```

## 7. Puertos de acceso
Ejemplo general para asignar un puerto a una sola VLAN

```cisco
interface <interfaz>
switchport mode access
switchport access vlan <id>
no shutdown
```

## 8. Subinterfaces (Router-on-a-Stick)
Ejemplo general para crear una subinterfaz

```cisco
interface <interfaz_fisica>
no shutdown

interface <interfaz_fisica>.<vlan>
encapsulation dot1Q <vlan>
ip address <ip_gateway> <máscara>
exit
```

## 9. Port Security
Ejemplo general para limitar MAC en un puerto de acceso

```cisco
interface range <interfaces>
switchport mode access
switchport port-security
switchport port-security maximum 1
switchport port-security mac-address sticky
switchport port-security violation restrict
no shutdown
exit
```

