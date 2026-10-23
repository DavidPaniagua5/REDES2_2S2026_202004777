# Proyecto 2

**Carné:** 202004777 · **Dominio:** `www.proyecto2_202004777.com`

---

## Direccionamiento asignado

| Elemento | Red |
| :---: | :---: |
| Interconexión BGP | `192.168.77.0` (trabajado con VLSM /30 y /27) |
| FibraNet GT | `172.16.17.0/24` |
| AndinaCom | `172.16.27.0/24` |
| MoonLink | `172.16.32.0/24` |

### Sistemas autónomos BGP

| ISP | AS |
| :---: | :---: |
| FibraNet | 65017 |
| AndinaCom | 65027 |
| MoonLink | 65032 |

### Enlaces BGP entre ISPs (planificados)

| Enlace | Red |
| :---: | :---: |
| FibraNet ↔ AndinaCom | 192.168.77.0/30 (.1 Fibra, .2 Andina) |
| AndinaCom ↔ MoonLink | 192.168.77.4/30 (.5 Andina, .6 Moon) |
| MoonLink ↔ FibraNet | 192.168.77.8/30 (.9 Moon, .10 Fibra) |

---

## Andinacom

- **Topología:** jerárquica de 3 capas (Core, Distribución, Acceso) más una rama de servicios.
- **IGP:** OSPF proceso 1, área 0.
- **Redundancia L3:** HSRP entre AND-DIST1 (Active, prioridad 110) y AND-DIST2 (Standby, prioridad 100), con `preempt`.
- **Agregación de enlaces:** 2 EtherChannel LACP (Po1 entre DIST1 y DIST2; Po2 entre CORE1 y CORE2).
- **DHCP centralizado:** servidor en la subred de servicios, con `ip helper-address` en las SVIs.

### Dispositivos

| Capa | Dispositivo | Modelo | Función |
| :---: | :---: | :---: | :---: |
| Core | AND-CORE1 | 3650-24PS | Borde BGP, router-id 2.2.2.2 |
| Core | AND-CORE2 | 3650-24PS | Core redundante, router-id 2.2.2.3 |
| Distribución | AND-DIST1 | 3650-24PS | HSRP Active, root STP, router-id 2.2.2.4 |
| Distribución | AND-DIST2 | 3650-24PS | HSRP Standby, root secundario, router-id 2.2.2.5 |
| Servicios | AND-RSRV | Router 2911 | Gateway del servidor DHCP, router-id 2.2.2.6 |
| Acceso | AND-ACC1, AND-ACC2 | 2960-24TT | Conexión de hosts |

### Subneteo y VLANs

| VLAN | Departamento | Red | IP virtual HSRP | DIST1 | DIST2 |
| :---: | :---: | :---: | :---: | :---: | :---: |
| 10 | Ventas | 172.16.27.0/26 | 172.16.27.1 | .2 | .3 |
| 20 | Facturación | 172.16.27.64/26 | 172.16.27.65 | .66 | .67 |
| – | Servidor DHCP | 172.16.27.128/27 | – | RSRV .129 | – |

Servidor DHCP: `172.16.27.130/27`, gateway `172.16.27.129`.

### Enlaces internos (/30 dentro de 192.168.77.64/27)

| Enlace | Medio | Red | Lado A | Lado B |
| :---: | :---: | :---: | :---: | :---: |
| CORE1 Gi1/0/1 ↔ DIST1 Gi1/0/1 | Cobre | 192.168.77.64/30 | .65 | .66 |
| CORE1 Gi1/0/2 ↔ DIST2 Gi1/0/2 | Cobre | 192.168.77.68/30 | .69 | .70 |
| CORE2 Gi1/0/2 ↔ DIST1 Gi1/0/2 | Cobre | 192.168.77.72/30 | .73 | .74 |
| CORE2 Gi1/0/1 ↔ DIST2 Gi1/0/1 | Cobre | 192.168.77.76/30 | .77 | .78 |
| CORE1 Po2 ↔ CORE2 Po2 (Gi1/1/1-2) | **Fibra** | 192.168.77.80/30 | .81 | .82 |
| CORE2 Gi1/0/3 ↔ RSRV Gi0/0 | Cobre | 192.168.77.84/30 | .85 | .86 |
| RSRV Gi0/1 ↔ Servidor DHCP | Cobre | 172.16.27.128/27 | .129 | .130 |

### Enlaces L2

| Origen | Destino | Medio | Función |
| :---: | :---: | :---: | :---: |
| DIST1 Gi1/1/1-2 | DIST2 Gi1/1/1-2 | **Fibra** | Po1 (LACP, trunk) |
| DIST1 Gi1/0/3 | ACC1 Gi0/1 | Cobre | Trunk |
| DIST1 Gi1/0/4 | ACC2 Gi0/1 | Cobre | Trunk |
| DIST2 Gi1/0/3 | ACC1 Gi0/2 | Cobre | Trunk |
| DIST2 Gi1/0/4 | ACC2 Gi0/2 | Cobre | Trunk |

### Pools DHCP configurados

| Pool | Gateway | DNS | Inicio | Máscara |
| :---: | :---: | :---: | :---: | :---: |
| ventas | 172.16.27.1 | 172.16.17.10 | 172.16.27.10 | 255.255.255.192 |
| facturacion | 172.16.27.65 | 172.16.17.10 | 172.16.27.74 | 255.255.255.192 |
| administracion | 172.16.17.1 | 172.16.17.10 | 172.16.17.20 | 255.255.255.128 |
| atencion | 172.16.17.129 | 172.16.17.10 | 172.16.17.140 | 255.255.255.128 |
| soporte | 172.16.32.1 | 172.16.17.10 | 172.16.32.10 | 255.255.255.192 |
| seguridad | 172.16.32.65 | 172.16.17.10 | 172.16.32.74 | 255.255.255.192 |

### Hosts (6 PCs, todos por DHCP)

| Switch | Puerto | PC | VLAN |
| :---: | :---: | :---: | :---: |
| ACC1 | Fa0/1 | PC-VEN1 | 10 |
| ACC1 | Fa0/2 | PC-VEN2 | 10 |
| ACC1 | Fa0/3 | PC-FAC1 | 20 |
| ACC2 | Fa0/1 | PC-VEN3 | 10 |
| ACC2 | Fa0/2 | PC-FAC2 | 20 |
| ACC2 | Fa0/3 | PC-FAC3 | 20 |

### Decisiones de diseño

1. **Arquitectura por capas en medios de red:** fibra en el core (Po2) y en la interconexión DIST1↔DIST2 (Po1), por el tráfico agregado, la distancia, la inmunidad a interferencias y la escalabilidad. Cobre en acceso y en enlaces CORE↔DIST para controlar costos. El enunciado exige fibra entre routers.
2. **Un solo área OSPF (0):** la topología es pequeña y no justifica múltiples áreas.
3. **Ruta estática a `Null0` en CORE1** para `172.16.27.0/24`, de modo que BGP pueda anunciar el /24 completo.
4. **CORE1 inyecta ruta por defecto en OSPF** (`default-information originate always`) para dar salida hacia los otros ISPs.
5. **Gateway del DHCP = IP virtual de HSRP** (.1 y .65), para que los hosts no pierdan salida cuando conmuta el gateway.
6. **Servidor DHCP en una subred propia**, detrás del router AND-RSRV. El tutor confirmó que los servidores pueden tener IP estática.
7. **Redes SVI en `passive-interface`** para no enviar hellos OSPF hacia los hosts.
8. **STP Rapid-PVST** con DIST1 como root primario y DIST2 como secundario.

### Verificación realizada

| Prueba | Resultado |
| :---: | :---: |
| `show etherchannel summary` | Po1 (SU) y Po2 (RU), puertos en (P) |
| `show ip ospf neighbor` en CORE1 | 3 vecinos en FULL (CORE2, DIST1, DIST2) |
| `show ip route ospf` en CORE1 | Aprende 172.16.27.0/26, .64/26 y .128/27 |
| `show standby brief` | DIST1 Active / DIST2 Standby en los grupos 10 y 20 |
| `show spanning-tree vlan 10` | DIST1 root; Po1 en DIST2 como Root FWD |
| DHCP | Los PCs reciben IP, máscara, gateway virtual y DNS |
| Ping entre hosts de la misma VLAN | Exitoso |
| Ping de hosts a dispositivos L3 | Exitoso |
| Conmutación HSRP (caída de SVIs en DIST1) | La conectividad se restablece en pocos segundos |

### Problema encontrado y solución

**Síntoma:** HSRP en ambos DIST aparecía como `Active local / unknown`, y los PCs recibían IP de forma inconsistente.

**Causa:** DIST2 mantenía en bloqueo STP el Po1 (los hellos de HSRP no cruzaban entre los DIST). Era un estado atascado de convergencia, ya que el trunk y las VLANs estaban correctos.

**Solución:** reinicio del Po1 en ambos DIST (`shutdown` / `no shutdown` del port-channel y de sus interfaces físicas). Tras ~1 minuto, HSRP formó el par y STP convergió.

```a
enable
configure terminal
hostname FNET-CORE
no ip domain-lookup
ip routing
interface gigabitEthernet 1/0/1
 description A-FNET-R1
 no switchport
 ip address 192.168.77.33 255.255.255.252
 no shutdown
interface gigabitEthernet 1/1/1
 description FIBRA-BGP-ANDINACOM
 no switchport
 ip address 192.168.77.1 255.255.255.252
 no shutdown
interface gigabitEthernet 1/1/2
 description FIBRA-BGP-MOONLINK
 no switchport
 ip address 192.168.77.10 255.255.255.252
 no shutdown
ip route 172.16.17.0 255.255.255.0 null0
router ospf 1
 router-id 1.1.1.1
 network 192.168.77.32 0.0.0.31 area 0
 default-information originate always
router bgp 65017
 bgp router-id 1.1.1.1
 neighbor 192.168.77.2 remote-as 65027
 neighbor 192.168.77.9 remote-as 65032
 network 172.16.17.0 mask 255.255.255.0
end
write memory
```

R1

```cisco
enable
configure terminal
hostname FNET-R1
no ip domain-lookup
interface gigabitEthernet 0/0
 ip address 192.168.77.34 255.255.255.252
 no shutdown
interface gigabitEthernet 0/1
 ip address 192.168.77.37 255.255.255.252
 no shutdown
interface gigabitEthernet 0/2
 ip address 192.168.77.41 255.255.255.252
 no shutdown
router ospf 1
 router-id 1.1.1.2
 network 192.168.77.32 0.0.0.31 area 0
end
write memory
```

R2 y R3

```cisco
hostname FNET-R2
interface gigabitEthernet 0/0
 ip address 192.168.77.38 255.255.255.252
 no shutdown
interface gigabitEthernet 0/1
 ip address 192.168.77.45 255.255.255.252
 no shutdown
router ospf 1
 router-id 1.1.1.3 
 network 192.168.77.32 0.0.0.31 area 0

hostname FNET-R2     
interface gigabitEthernet 0/0
 ip address 192.168.77.42 255.255.255.252 
 no shutdown
interface gigabitEthernet 0/1
 ip address 192.168.77.49 255.255.255.252 
 no shutdown
router ospf 1
 router-id 1.1.1.4 
 network 192.168.77.32 0.0.0.31 area 0
```

R4

```cisco
hostname FNET-R4
interface gigabitEthernet 0/0
 ip address 192.168.77.46 255.255.255.252
 no shutdown
interface gigabitEthernet 0/1
 description LAN-ADMINISTRACION
 ip address 172.16.17.1 255.255.255.128
 ip helper-address 172.16.27.130
 no shutdown
router ospf 1
 router-id 1.1.1.5
 network 192.168.77.32 0.0.0.31 area 0
 network 172.16.17.0 0.0.0.127 area 0
 passive-interface gigabitEthernet 0/1
```

R5

```cisco
hostname FNET-R5
interface gigabitEthernet 0/0
 ip address 192.168.77.50 255.255.255.252
 no shutdown
interface gigabitEthernet 0/1
 description LAN-ATENCION
 ip address 172.16.17.129 255.255.255.128
 ip helper-address 172.16.27.130
 no shutdown
router ospf 1
 router-id 1.1.1.6
 network 192.168.77.32 0.0.0.31 area 0
 network 172.16.17.128 0.0.0.127 area 0
 passive-interface gigabitEthernet 0/1
```

Switchs de acceso (2960)

```cisco
enable
configure terminal
hostname FIB-SW-ADM1                  ! ADM2, ATC1, ATC2 según corresponda
interface range gigabitEthernet 0/1 - 2
 channel-protocol lacp
 channel-group 1 mode active
 no shutdown
end
write memory
````

```cisco
enable
configure terminal
hostname FIB-SW-ADM1                  ! ADM2, ATC1, ATC2 según corresponda
no ip domain-lookup
interface range fastEthernet 0/23 - 24
 channel-protocol lacp
 channel-group 1 mode active
 no shutdown
end
write memory
```
