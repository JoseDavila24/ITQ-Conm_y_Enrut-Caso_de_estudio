# ITQ-Conm_y_Enrut-Caso_de_estudio

Caso de estudio (práctica integradora) de la materia **Conmutación y Enrutamiento de Redes de Datos** (ITQ).

**Maestro:** Luis Raymundo Arán Sánchez

## Descripción

El caso de estudio consiste en implementar en **Cisco Packet Tracer** la red de la **zona A** del equipo 5: tres switches con VLANs, un router que da salida entre VLANs y hacia la WAN, y una red inalámbrica para laptops.

### Topología

- **R5:** router de la zona A. Hace Router-on-a-Stick por G0/1, da DHCP a la red WiFi y se conecta con **R1** por un enlace serial PPP con autenticación CHAP.
- **SW_A1:** switch raíz de spanning tree y servidor VTP. Se conecta a R5 y a los otros dos switches.
- **SW_A2 y SW_A3:** switches de acceso y clientes VTP. Conectan las PCs (VLANs 301-304) y el Access Point (VLAN 305). También están conectados entre sí.

### Direccionamiento

| VLAN | Red             | Gateway (R5)  | Uso              |
| ---- | --------------- | ------------- | ---------------- |
| 301  | 172.16.8.208/28 | 172.16.8.222  | PCs              |
| 302  | 172.16.8.224/28 | 172.16.8.238  | PCs              |
| 303  | 172.16.8.240/28 | 172.16.8.254  | PCs              |
| 304  | 172.16.9.0/28   | 172.16.9.14   | PCs              |
| 305  | 172.16.9.16/28  | 172.16.9.30   | WiFi (DHCP)      |
| WAN  | 172.16.9.32/30  | R5 .34 / R1 .33 | Enlace R5 ↔ R1 |

VLANs nativas de los troncales entre switches: 11, 12 y 13.

### Temas que se implementan

- Subnetting
- VLANs y VTP (servidor / clientes)
- Spanning Tree (PVST) con prioridades por switch
- Troncales 802.1Q con VLAN nativa
- Router-on-a-Stick (subinterfaces)
- Port Security con MAC sticky
- DHCP en el router
- PPP con autenticación CHAP
- Enrutamiento dinámico con EIGRP 100
- Red inalámbrica con AP-PT-AC y laptops con módulo WPC300N (WPA-PSK)

## Contenido del repositorio

| Archivo        | Descripción                                                  |
| -------------- | ------------------------------------------------------------ |
| `Practica.md`  | Guía paso a paso con los comandos de cada equipo de la zona A |
| `*.png`        | Capturas de la configuración del AP y de las laptops          |
