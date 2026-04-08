#  trabajo
1. Arquitectura de Virtualización en Windows 11
.El Aislamiento de Núcleo y VBS (Virtualization-Based Security) en Windows 11 tienen un impacto crítico en los entornos de virtualización porque modifican la forma en que el sistema operativo accede al hardware.
Bloqueo de acceso a VT-x / AMD-V

Cuando está activo VBS, Windows usa internamente un hipervisor (Hyper-V).

Esto provoca que:

El CPU ya no esté disponible directamente
Otros hipervisores no puedan usar:
VT-x (Intel)
AMD-V (AMD)

.Activación de VT-x/AMD-V: Procedimiento para habilitar el soporte de hardware y cómo verificarlo desde el sistema operativo.

¿Qué es VT-x / AMD-V?

Las tecnologías VT-x (Intel) y AMD-V (AMD) son extensiones de hardware del procesador que permiten ejecutar máquinas virtuales con aceleración directa por CPU.

Sin estas tecnologías:

Las VMs funcionan por emulación (muy lenta)
No se puede usar KVM en GNS3
El rendimiento del laboratorio es deficiente

2. GNS3 VM: El Motor de Simulación

• KVM (Kernel-based Virtual Machine): Investigar qué es y por qué es
obligatorio que aparezca como "True" en el servidor GNS3 para un rendimiento
profesional.

¿Qué es KVM?

KVM (Kernel-based Virtual Machine) es una tecnología de virtualización integrada en el kernel de Linux que permite ejecutar máquinas virtuales con acceso directo al hardware (virtualización asistida por CPU).

En GNS3, KVM se utiliza dentro de la GNS3 VM para ejecutar:

Routers (Cisco, MikroTik, etc.)
Firewalls
Equipos virtuales complejos

¿Por qué es obligatorio que diga “KVM: True”?

En GNS3 VM aparece:

KVM support available: True

Esto significa que:

La VM tiene acceso a VT-x / AMD-V
Se está usando virtualización por hardware
El rendimiento es óptimo

• Configuración de Recursos: Definir criterios para asignar CPU y RAM a la
GNS3 VM sin desestabilizar Windows 11.

¿Por qué es importante asignar bien los recursos?

La GNS3 VM comparte recursos con Windows 11.
Una mala asignación puede provocar:

Lentitud en GNS3
Congelamientos del sistema
Fallos en máquinas virtuales

¿Cuántos cores asignar?

| CPU del Host | CPU para GNS3 VM |
| ------------ | ---------------- |
| 2 cores      | 1 core           |
| 4 cores      | 2 cores          |
| 8 cores      | 2–4 cores        |

¿Cuánta RAM asignar?
Regla general:

Usar entre 50% – 60% de la RAM total

PC con 8 GB RAM:

| Uso        | RAM    |
| ---------- | ------ |
| Windows 11 | 3-4 GB |
| GNS3 VM    | 4 GB   |

Consumo por dispositivo

| Dispositivo | RAM aproximada |
| ----------- | -------------- |
| VPCS        | 50 MB          |
| Router IOSv | 512 MB         |
| CSR1000v    | 2-3 GB         |
| Firewall    | 1-2 GB         |

3. Integración con VirtualBox (Local)
• Configuración de Red: Pasos para crear y configurar el adaptador Host-Only
para la comunicación GUI-Server.

¿Qué es el adaptador Host-Only?

El Host-Only Adapter en VirtualBox es una red virtual privada que permite la comunicación entre:

Sistema host (Windows 11)
GNS3 VM (servidor)

Sin acceso a Internet, pero con comunicación directa entre ambos.

Procedimiento paso a paso
1. Crear red Host-Only
Abrir VirtualBox
Ir a:
File → Host Network Manager
Crear nueva red

Configurar la GNS3 VM

En VirtualBox:

Ir a:
Configuración → Red
Adaptadores:
Adaptador 1:
Tipo: NAT
Para acceso a Internet
Adaptador 2:
Tipo: Host-Only
Para comunicación con el host


• Modo Promiscuo: Explicar técnicamente por qué es necesario para el tráfico de
Capa 2.

El modo promiscuo permite que una interfaz de red capture todo el tráfico que pasa por la red, no solo el que está dirigido a su dirección MAC.

Concepto de Capa 2

La Capa 2 del modelo OSI corresponde a la capa de enlace de datos, donde los dispositivos manejan:

Tramas Ethernet
Direcciones MAC
Broadcast y multicast

Ejemplo: Switches, bridges y tarjetas de red virtuales trabajan a nivel de Capa 2.

Cómo funciona normalmente
Una tarjeta de red recibe solo las tramas dirigidas a su propia MAC.
Tramas de otros dispositivos se descartan automáticamente.

3. Integración con VirtualBox (Local)

• Configuración de Red: Pasos para crear y configurar el adaptador Host-Only
para la comunicación GUI-Server.

Pasos para configurar el adaptador Host-Only
Abrir VirtualBox → File → Host Network Manager
Crear nueva red Host-Only
Configurar parámetros:
Parámetro	Valor recomendado
IP	192.168.56.1
Máscara	255.255.255.0
DHCP	Activado
Configurar la GNS3 VM en VirtualBox:
Adaptador 1: NAT (para Internet)
Adaptador 2: Host-Only (para comunicación GUI ↔ VM)

¿Por qué es necesario?
GNS3 usa un modelo cliente-servidor:
GNS3 GUI (Windows) ↔ GNS3 VM (Linux)
El adaptador Host-Only permite:
Comunicación estable GUI ↔ VM
Acceso a la API de GNS3 (puerto 3080)
Transferencia de topologías y configuraciones

• Modo Promiscuo: Explicar técnicamente por qué es necesario para el tráfico de
Capa 2.

Qué es el modo promiscuo
Permite que una interfaz de red reciba todas las tramas que pasan por la red, no solo las dirigidas a su MAC.
Captura tráfico unicast, multicast y broadcast.

Por qué es necesario para Capa 2
La Capa 2 maneja:
Tramas Ethernet
Direcciones MAC
Protocolos como ARP, STP, VLANs
Los switches virtuales necesitan ver todas las tramas para:
Aprender direcciones MAC
Reenviar correctamente el tráfico entre VMs

Sin modo promiscuo, las VMs solo ven tráfico destinado a ellas → los routers y switches virtuales no pueden comunicarse.

4. Integración con VMware ESXi (Remoto)

• Arquitectura Cliente-Servidor: Cómo conectar el GUI de GNS3 de la laptop a
un servidor ESXi físico.

Conexión GUI ↔ ESXi
Instalar GNS3 VM en ESXi (como máquina virtual)
Configurar la GNS3 VM con:
Adaptador de red en port group con IP estática
Acceso a la red de tu laptop (LAN o VPN según el escenario)
En la laptop (GNS3 GUI):
Configurar GNS3 Server remoto apuntando a la IP de la VM en ESXi
Puerto predeterminado: 3080 TCP

Esto permite que la GUI controle los dispositivos virtuales en el servidor ESXi sin necesidad de ejecutar los routers localmente.


• Seguridad en vSwitch: Investigar la configuración de "Políticas de Seguridad"
(Promiscuous mode, MAC address changes) en el port group de ESXi.

las políticas de seguridad del vSwitch/Port Group determinan cómo se comporta la red virtual para las VMs:

Promiscuous Mode
Permite que la VM reciba todo el tráfico que pasa por el switch virtual, no solo el destinado a su MAC.
Es necesario para GNS3 porque la VM debe ver todo el tráfico de Capa 2, igual que en VirtualBox.
MAC Address Changes
Permite que la VM cambie su dirección MAC asignada por ESXi.
Necesario para algunas imágenes de routers que generan múltiples interfaces virtuales con MAC dinámicas.
Forged Transmits
Permite que la VM envíe tramas con una MAC diferente a la asignada por ESXi.
Requerido cuando los dispositivos virtuales necesitan replicar tráfico como si fueran múltiples nodos físicos.
