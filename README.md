# vlan-intervlan-cisco-packet-tracer
Diseño, segmentación lógica mediante VLANs y configuración de enrutamiento Inter-VLAN (Router-on-a-Stick) en Cisco Packet Tracer para TechSolutions S.A. Incluye conectividad a servicios externos.

# Implementación de VLANs y Enrutamiento Inter-VLAN en Cisco Packet Tracer

## 📝 Descripción del Proyecto
Este repositorio contiene el diseño, configuración y validación de una red empresarial segmentada lógicamente mediante **VLANs**, utilizando una arquitectura de enrutamiento **Router-on-a-Stick**. El escenario simula la red interna de la empresa *TechSolutions S.A.*, garantizando el aislamiento de sus departamentos críticos y proveyendo conectividad externa hacia servicios web simulados (Google y Facebook) a través de un router perimetral.

---

## 🗺️ Topología de la Red
La arquitectura física y lógica implementada se compone de los siguientes elementos:
* **Routers:** 1 Cisco ISR 4331 (`Router-Interno`) y 1 Cisco ISR 4331 (`RauterPerimetral`).
* **Switches:** 2 Cisco Catalyst 2960 (`Switch-1` y `Switch-2`) para la red de acceso, y 1 Cisco Catalyst 2960 (`Switch-Internet`) como concentrador de servidores.
* **Hosts:** Mínimo 3 PCs por departamento distribuidas entre ambos switches de acceso.
* **Servidores:** 2 Servidores Web dedicados (`Facebook` y `Google`).

### Esquema de Conexiones Físicas
* **Router-on-a-Stick:** `Router-Interno` [Gi0/0/0] ↔ `Switch-1` [Gi0/1] *(Enlace Troncal)*
* **Enlace entre Routers:** `Router-Interno` [Gi0/0/1] ↔ `RauterPerimetral` [Gi0/0/0] *(Red: 10.0.0.0/30)*
* **Enlace Troncal entre Switches:** `Switch-1` [Fa0/24] ↔ `Switch-2` [Fa0/24]
* **Enlace hacia Red Externa:** `RauterPerimetral` [Gi0/0/1] ↔ `Switch-Internet` [Fa0/24] *(Red: 200.0.0.0/24)*
* **Servidores Web:** `Switch-Internet` [Fa0/1] ↔ `Server Google` y `Switch-Internet` [Fa0/2] ↔ `Server Facebook`

---

## 📊 Segmentación y Direccionamiento IP

### Departamentos y VLANs
| Departamento | VLAN | Red IP | Máscara | Puerta de Enlace (Gateway) | Rangos de Puertos Asignados |
| :--- | :---: | :--- | :--- | :--- | :--- |
| **Administración** | 10 | 192.168.10.0/24 | 255.255.255.0 | 192.168.10.1 | `Fa0/1` - `Fa0/5` |
| **Recursos Humanos** | 20 | 192.168.20.0/24 | 255.255.255.0 | 192.168.20.1 | `Fa0/6` - `Fa0/10` |
| **Ingeniería** | 30 | 192.168.30.0/24 | 255.255.255.0 | 192.168.30.1 | `Fa0/11` - `Fa0/15` |

### Enlaces de Infraestructura y Servidores
* **Enlace Inter-Router:** `Router-Interno` (10.0.0.1) | `RauterPerimetral` (10.0.0.2)
* **Servidor Facebook:** IP `200.0.0.10` | Gateway `200.0.0.1`
* **Servidor Google:** IP `200.0.0.20` | Gateway `200.0.0.1`

---

## 💻 Comandos de Configuración (IOS)

### Switch-1

enable
configure terminal
hostname Switch-1
vlan 10
 name Administracion
vlan 20
 name Recursos_Humanos
vlan 30
 name Ingenieria
exit
interface range fa0/1 - 5
 switchport mode access
 switchport access vlan 10
exit
interface range fa0/6 - 10
 switchport mode access
 switchport access vlan 20
exit
interface range fa0/11 - 15
 switchport mode access
 switchport access vlan 30
exit
interface fa0/24
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
exit
interface gi0/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
exit

---

### Switch-2

enable
configure terminal
hostname Switch-2
vlan 10
 name Administracion
vlan 20
 name Recursos_Humanos
vlan 30
 name Ingenieria
exit
interface range fa0/1 - 5
 switchport mode access
 switchport access vlan 10
exit
interface range fa0/6 - 10
 switchport mode access
 switchport access vlan 20
exit
interface range fa0/11 - 15
 switchport mode access
 switchport access vlan 30
exit
interface fa0/24
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
exit

---

### Router-Interno

enable
configure terminal
hostname Router-Interno
interface gi0/0/0
 no shutdown
exit
interface gi0/0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
exit
interface gi0/0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
exit
interface gi0/0/0.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
exit
interface gi0/0/1
 ip address 10.0.0.1 255.255.255.252
 no shutdown
exit
ip route 0.0.0.0 0.0.0.0 10.0.0.2

---

### RauterPerimetral

enable
configure terminal
hostname RauterPerimetral
interface gi0/0/0
 ip address 10.0.0.2 255.255.255.252
 no shutdown
exit
interface gi0/0/1
 ip address 200.0.0.1 255.255.255.0
 no shutdown
exit
ip route 192.168.10.0 255.255.255.0 10.0.0.1
ip route 192.168.20.0 255.255.255.0 10.0.0.1
ip route 192.168.30.0 255.255.255.0 10.0.0.1


