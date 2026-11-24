---
Maestro: LUIS RAYMUNDO ARAN SANCHEZ
---
## Comandos de implementación en la zona A

### Topología de referencia

| Dispositivo | Interfaz | Conecta a     | Modo    | VLAN nativa / VLAN |
| ----------- | -------- | ------------- | ------- | ------------------ |
| R5          | G0/1     | SW_A1 G0/1    | Troncal | Subinterfaces 301-305 |
| R5          | S0/0/0   | R1            | PPP/CHAP | 172.16.9.34/30    |
| SW_A1       | Fa0/11   | SW_A2 Fa0/11  | Troncal | 11                 |
| SW_A1       | Fa0/12   | SW_A3 Fa0/12  | Troncal | 12                 |
| SW_A1       | G0/1     | R5 G0/1       | Troncal | 1 (por defecto)    |
| SW_A2       | Fa0/13   | SW_A3 Fa0/13  | Troncal | 13                 |
| SW_A2/SW_A3 | Fa0/1-4  | PCs           | Acceso  | 301, 302, 303, 304 |
| SW_A2/SW_A3 | Fa0/5    | Access Point  | Acceso  | 305 (WiFi)         |

## 1. Subnetear
| Vlan | Nombre de red | Mascara         | Prefijo | Broadcast    | Rango                       | Gateway (R5)  |
| ---- | ------------- | --------------- | ------- | ------------ | --------------------------- | ------------- |
| 301  | 172.16.8.208  | 255.255.255.240 | /28     | 172.16.8.223 | 172.16.8.209 - 172.16.8.222 | 172.16.8.222  |
| 302  | 172.16.8.224  | 255.255.255.240 | /28     | 172.16.8.239 | 172.16.8.225 - 172.16.8.238 | 172.16.8.238  |
| 303  | 172.16.8.240  | 255.255.255.240 | /28     | 172.16.8.255 | 172.16.8.241 - 172.16.8.254 | 172.16.8.254  |
| 304  | 172.16.9.0    | 255.255.255.240 | /28     | 172.16.9.15  | 172.16.9.1 - 172.16.9.14    | 172.16.9.14   |
| 305  | 172.16.9.16   | 255.255.255.240 | /28     | 172.16.9.31  | 172.16.9.17 - 172.16.9.30   | 172.16.9.30   |
| WAN  | 172.16.9.32   | 255.255.255.252 | /30     | 172.16.9.35  | 172.16.9.33 - 172.16.9.34   | R5 = .34 / R1 = .33 |

> El gateway de cada VLAN es la **última IP utilizable** del rango.

## 2. Apagar la interfaces de los switches (Recomendado)
Aplicar en **SW_A1, SW_A2 y SW_A3**. Más adelante se habilitan solo los puertos que se usan.

```cisco
enable
configure terminal
interface range fastEthernet 0/1 - 24, gigabitEthernet 0/1 - 2
shutdown
exit
```

Asignar nombre a cada switch (opcional pero recomendado):

```cisco
hostname SW_A1
```
```cisco
hostname SW_A2
```
```cisco
hostname SW_A3
```

## 3. Direccionamiento de PCs
Ingresar en cada PC la IP, mascara y DG

| VLAN | IP (ejemplo)  | Máscara         | Default Gateway |
| ---- | ------------- | --------------- | --------------- |
| 301  | 172.16.8.209  | 255.255.255.240 | 172.16.8.222    |
| 302  | 172.16.8.225  | 255.255.255.240 | 172.16.8.238    |
| 303  | 172.16.8.241  | 255.255.255.240 | 172.16.8.254    |
| 304  | 172.16.9.1    | 255.255.255.240 | 172.16.9.14     |
| 305  | DHCP          | DHCP            | DHCP (172.16.9.30) |

> Si hay más de una PC en la misma VLAN (una en SW_A2 y otra en SW_A3), usar la siguiente IP del rango (ej. 172.16.8.210).

> Nota: Para los AP se usaron los AP-PT-AC y laptops con para la VLAN 305 se usaron laps con el modulo WPC300N para conectarse por wi-fi y se optorgo IPs de forma dinamica por DHCP con el router

## 4. VTP (server + Clientes)

4.1 Configurar el servidor y crear vlans (**SW_A1**)
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

4.2 Configurar en ambos clientes (**SW_A2 y SW_A3**)
```cisco
vtp version 2
vtp domain equipo5
vtp password equipo5
vtp mode client
```

> Las VLANs llegan a los clientes **solo cuando los troncales estén activos** (paso 6). Verificar con `show vlan brief` y `show vtp status`.

## 5. Spanning Tree (PVST)

Raiz (**SW_A1**)
```cisco
spanning-tree mode pvst
spanning-tree vlan 1,11-13,301-305 priority 0
```

Segundo (**SW_A2**)
```cisco
spanning-tree mode pvst
spanning-tree vlan 1,11-13,301-305 priority 4096
```

Tercero (**SW_A3**)
```cisco
spanning-tree mode pvst
spanning-tree vlan 1,11-13,301-305 priority 8192
```

## 6. Puertos troncales
**SW_A1**
```cisco
interface fastEthernet 0/11
switchport mode trunk
switchport trunk native vlan 11
no shutdown
exit

interface fastEthernet 0/12
switchport mode trunk
switchport trunk native vlan 12
no shutdown
exit

! Troncal hacia R5 (router-on-a-stick)
interface gigabitEthernet 0/1
switchport mode trunk
no shutdown
exit
```

**SW_A2**
```cisco
interface fastEthernet 0/11
switchport mode trunk
switchport trunk native vlan 11
no shutdown
exit

interface fastEthernet 0/13
switchport mode trunk
switchport trunk native vlan 13
no shutdown
exit
```

**SW_A3**
```cisco
interface fastEthernet 0/12
switchport mode trunk
switchport trunk native vlan 12
no shutdown
exit

interface fastEthernet 0/13
switchport mode trunk
switchport trunk native vlan 13
no shutdown
exit
```

> La VLAN nativa debe ser **la misma en ambos extremos** del enlace, si no aparece el error *native VLAN mismatch*.

## 7. Puertos de acceso
**SW_A2 y SW_A3** (misma configuración en ambos)
```cisco
interface fastEthernet 0/1
switchport mode access
switchport access vlan 301
no shutdown
exit

interface fastEthernet 0/2
switchport mode access
switchport access vlan 302
no shutdown
exit

interface fastEthernet 0/3
switchport mode access
switchport access vlan 303
no shutdown
exit

interface fastEthernet 0/4
switchport mode access
switchport access vlan 304
no shutdown
exit

! Puerto del Access Point (WiFi)
interface fastEthernet 0/5
switchport mode access
switchport access vlan 305
no shutdown
exit
```

## 8. Subinterfaces (Router-on-a-Stick)
**R5**
```cisco
enable
configure terminal
hostname R5

interface gigabitEthernet 0/1
no ip address
no shutdown
exit

interface gigabitEthernet 0/1.301
encapsulation dot1Q 301
ip address 172.16.8.222 255.255.255.240
exit

interface gigabitEthernet 0/1.302
encapsulation dot1Q 302
ip address 172.16.8.238 255.255.255.240
exit

interface gigabitEthernet 0/1.303
encapsulation dot1Q 303
ip address 172.16.8.254 255.255.255.240
exit

interface gigabitEthernet 0/1.304
encapsulation dot1Q 304
ip address 172.16.9.14 255.255.255.240
exit

interface gigabitEthernet 0/1.305
encapsulation dot1Q 305
ip address 172.16.9.30 255.255.255.240
exit
```

## 9. Port Security
**SW_A2 y SW_A3** (solo puertos de PCs; el puerto del AP Fa0/5 queda sin port-security)
```cisco
interface range fastEthernet 0/1 - 4
switchport mode access
switchport port-security
switchport port-security maximum 1
switchport port-security mac-address sticky
switchport port-security violation restrict
no shutdown
exit
```

> Las MAC *sticky* se aprenden cuando la PC envía tráfico (ej. un `ping` a su gateway). Verificar con `show port-security address`.

## 10. DHCP para la VLAN 305 (WiFi) en R5

```cisco
ip dhcp excluded-address 172.16.9.30

ip dhcp pool POOL_VLAN305
network 172.16.9.16 255.255.255.240
default-router 172.16.9.30
exit
```

## 11. Enlace WAN con PPP + CHAP (R5 ↔ R1)

**R5**
```cisco
username R1 secret <password>

interface serial 0/0/0
ip address 172.16.9.34 255.255.255.252
encapsulation ppp
ppp authentication chap
no shutdown
exit
```

**R1**
```cisco
username R5 secret <password>

interface serial 0/0/0
ip address 172.16.9.33 255.255.255.252
encapsulation ppp
ppp authentication chap
no shutdown
exit
```

> En CHAP cada router crea el `username` con el **hostname del otro router**, y la contraseña debe ser **idéntica** en ambos lados.

## 12. Enrutamiento EIGRP (R5)

```cisco
router eigrp 100
passive-interface default
no passive-interface serial 0/0/0
network 172.16.8.222 0.0.0.0
network 172.16.8.238 0.0.0.0
network 172.16.8.254 0.0.0.0
network 172.16.9.14 0.0.0.0
network 172.16.9.30 0.0.0.0
network 172.16.9.34 0.0.0.0
exit
```

> `passive-interface default` evita enviar hellos de EIGRP hacia las LAN; solo se forma vecindad por el serial con R1.

## 13. WiFi (Access Point)

> **Pendiente de completar manualmente.**
>
> - Dispositivo: AP-PT-AC conectado a **Fa0/5** de SW_A2 / SW_A3 (VLAN 305).
> - Clientes: laptops con el módulo WPC300N.
> - Red asignada: 172.16.9.16/28, gateway 172.16.9.30, IPs entregadas por el DHCP de R5 (paso 10).
> - Datos por llenar: SSID, tipo de seguridad (WPA2-PSK, etc.), contraseña y configuración de las laptops.

```text
SSID:       <por definir>
Seguridad:  <por definir>
Password:   <por definir>
```

## 14. Guardar y verificar

Guardar en cada dispositivo:
```cisco
end
copy running-config startup-config
```

Comandos de verificación:

| Dispositivo    | Comando                              | Qué revisar                                  |
| -------------- | ------------------------------------ | -------------------------------------------- |
| Switches       | `show vtp status`                    | Dominio, modo y número de VLANs              |
| Switches       | `show vlan brief`                    | VLANs creadas y puertos de acceso            |
| Switches       | `show interfaces trunk`              | Troncales y VLAN nativa                      |
| Switches       | `show spanning-tree`                 | SW_A1 como raíz (*This bridge is the root*)  |
| SW_A2 / SW_A3  | `show port-security address`         | MAC sticky aprendidas                        |
| R5             | `show ip interface brief`            | Subinterfaces y serial en *up/up*            |
| R5             | `show ip dhcp binding`               | Clientes WiFi con IP asignada                |
| R5             | `show ip eigrp neighbors`            | Vecindad con R1                              |
| R5             | `show ip route`                      | Rutas aprendidas por EIGRP (D)               |
