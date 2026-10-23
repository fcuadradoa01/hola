---
title: "Laboratorio MikroTik RouterOS 7: Guía de Segmentación y Hardening"
tags:
  - Mikrotik
  - RouterOS
  - firewall
  - redes
  - segmentacion
level: intermedio
---

# Laboratorio MikroTik RouterOS 7: Guía de Segmentación y Hardening

Este documento sirve como guía de laboratorio para la implementación práctica de políticas de seguridad (segmentación de red, filtrado entre zonas y endurecimiento del plano de control) en un entorno virtualizado con **MikroTik RouterOS 7**.

---

## 🎯 Requisitos y Escenario del Laboratorio

Se requiere configurar las políticas de seguridad y direccionamiento en un router MikroTik RouterOS 7 que gestiona tres zonas de red: **WAN**, **LAN** (`192.168.10.0/24`) y **DMZ** (`192.168.11.0/24`).

### Topología de Interfaces
* **ether1:** WAN (Conectada a Internet / NAT externa)
* **ether2:** LAN (Red de empleados, `192.168.10.0/24`)
* **ether3:** DMZ (Red de servidores, `192.168.11.0/24`)

!!! info "IP de WAN"
    Entendemos tiene un IP pública proporcionada por ISP.


### Políticas de Acceso Requeridas
1. **Deshabilitación de servicios innecesarios:** Desactivar Telnet, FTP, API, API-SSL y WWW-SSL. Mantener únicamente SSH, Winbox y HTTP.
2. **Acceso ICMP (Ping):** Permitir `ping` hacia las IPs de las interfaces del router únicamente desde la **LAN** y desde la **DMZ**.
3. **Acceso al Router desde la WAN:** Permitir **únicamente Winbox** (`tcp/8291`) desde una IP pública autorizada concreta (ej. `203.0.113.50`).
4. **Acceso al Router desde la LAN:** Permitir **SSH** (`tcp/2222`), **HTTP** (`tcp/80`) y **Winbox** (`tcp/8291`) exclusivamente desde la máquina del administrador de sistemas (`192.168.10.50`).
5. **Acceso RDP desde la WAN (DNAT):** Redirigir el tráfico RDP (`tcp/3389`) procedente de la IP externa (ej. `203.0.113.50`) hacia el equipo del Sysadmin (`192.168.10.50`).
6. **Publicación Web en DMZ (DNAT):** Publicar el sitio WordPress ubicado en `192.168.11.11` haciéndolo accesible desde cualquier IP de la WAN a través del puerto público `8080` (redirigido al puerto `80` interno).

---

## Paso 1: Creación de Listas de Interfaces (Interface Lists)

Agrupamos las interfaces físicas en listas lógicas para mantener un firewall escalable.

```routeros
/interface list
add name=WAN
add name=LAN
add name=DMZ

/interface list member
add interface=ether1 list=WAN
add interface=ether2 list=LAN
add interface=ether3 list=DMZ
```

---

## Paso 2: Hardening de Servicios del Router

Deshabilitamos todos los servicios de gestión no utilizados y reconfiguramos los puertos de administración activa.

```routeros
/ip service
set ssh port=2222 disabled=no
set www port=80 disabled=no
set winbox port=8291 disabled=no
set telnet disabled=yes
set ftp disabled=yes
set api disabled=yes
set api-ssl disabled=yes
set www-ssl disabled=yes
```

!!! info "SSH socket"
    Cambiamos el puerto 'ssh' del rúter para que atienda por el '2222' en lugar del '22', oculatando puertos conocidos.

---

## Paso 3: Configuración de Traductores de Dirección (NAT)

Configuramos las reglas de enmascaramiento para la salida a Internet y los reenvíos de puertos (DNAT).

```routeros
/ip firewall nat
# 1. Salida a Internet (Masquerade) para LAN y DMZ
add chain=srcnat out-interface-list=WAN src-address=192.168.10.0/24 action=masquerade comment="NAT LAN a WAN"
add chain=srcnat out-interface-list=WAN src-address=192.168.11.0/24 action=masquerade comment="NAT DMZ a WAN"

# 2. DNAT: Acceso RDP desde IP externa específica (80.80.3.1) hacia la PC del Sysadmin (192.168.10.50)
add chain=dstnat in-interface-list=WAN src-address=80.80.3.1 protocol=tcp dst-port=3389 action=dst-nat to-addresses=192.168.10.50 to-ports=3389 comment="DNAT: RDP externo a PC Sysadmin"

# 3. DNAT: Publicar WordPress desde cualquier origen WAN en puerto 8080 hacia la DMZ (192.168.11.11:80)
add chain=dstnat in-interface-list=WAN protocol=tcp dst-port=8080 action=dst-nat to-addresses=192.168.11.11 to-ports=80 comment="DNAT: Publicar WordPress en DMZ"
```

!!! info "internet"
    Con esto conseguimos que tanto los equipos de **LAN** como los de la **DMZ** tengan salida a internet a traves de nuestro rúter.

---

## Paso 4: Reglas de Firewall para el Plano de Control (Cadena `Input`)

Protegemos el acceso directo al propio router.

> **Resumen del Firewall Stateful:**
> * **Established:** Conexiones en curso ya validadas.
> * **Related:** Conexiones secundarias derivadas de una activa.
> * **Invalid:** Paquetes fuera de secuencia o malformados (se descartan).

```routeros
/ip firewall filter
# 1. Estados de conexión básicos
add chain=input action=accept connection-state=established,related comment="Input: Aceptar establecidas/relacionadas"
add chain=input action=drop connection-state=invalid comment="Input: Descartar invalidos"

# 2. Permitir ICMP (Ping) desde la LAN y la DMZ al router
add chain=input action=accept in-interface-list=LAN protocol=icmp comment="Permitir Ping desde LAN al router"
add chain=input action=accept in-interface-list=DMZ protocol=icmp comment="Permitir Ping desde DMZ al router"

# 3. Administración desde la WAN: Solo Winbox desde IP externa autorizada (203.0.113.50)
add chain=input action=accept in-interface-list=WAN src-address=203.0.113.50 protocol=tcp dst-port=8291 comment="Admin WAN: Solo Winbox desde IP autorizada"

# 4. Administración desde la LAN: SSH, HTTP y Winbox solo desde la IP del Sysadmin (192.168.10.50)
add chain=input action=accept in-interface-list=LAN src-address=192.168.10.50 protocol=tcp dst-port=2222 comment="Admin LAN: SSH Sysadmin"
add chain=input action=accept in-interface-list=LAN src-address=192.168.10.50 protocol=tcp dst-port=80 comment="Admin LAN: HTTP Sysadmin"
add chain=input action=accept in-interface-list=LAN src-address=192.168.10.50 protocol=tcp dst-port=8291 comment="Admin LAN: Winbox Sysadmin"

# 5. Política de Cierre (Default Deny en Input)
add chain=input action=drop comment="Default Deny Input (Bloquear resto de accesos al router)"
```

!!! info "¿Qué significa established?"
    "Es una conversación ya iniciada." Sin esta regla cada paquete (cada vez que haces clic en winbox por ejemplo) tendría que recorrer todas las reglas del firewall.

#### ¿Qué significa related?

"Es una conversación secundaria derivada de otra." Son conexiones nuevas, pero que están relacionadas con otra ya existente.

#### ¿Qué significa invalid?

"Es alguien hablando en mitad de una conversación sin que nadie sepa quién es."

**¿Por qué descartarlos?** Porque normalmente son:

- Errores.
- Ruido de red.
- Tráfico malformado.
- Intentos de evasión del firewall.
- Escaneos

... y no suelen tener utilidad legítima.


#### ...con una analogía física:

- New → llama a la puerta.
- Established → está dentro de casa.
- Related → viene acompañado de alguien que ya conocemos.
- Invalid → aparece en mitad del salón sin haber entrado por la puerta.


---

## Paso 5: Reglas de Firewall para Tráfico en Tránsito (Cadena `Forward`)

Controlamos el flujo de paquetes que atraviesan el router entre las diferentes zonas.

```routeros
/ip firewall filter
# 1. Estados de conexión básicos en tránsito
add chain=forward action=accept connection-state=established,related comment="Forward: Aceptar establecidas/relacionadas"
add chain=forward action=drop connection-state=invalid comment="Forward: Descartar invalidos"

# 2. Tráfico RDP entrante desde la WAN hacia la IP del Sysadmin
add chain=forward action=accept in-interface-list=WAN out-interface-list=LAN dst-address=192.168.10.50 protocol=tcp dst-port=3389 comment="Forward: RDP WAN a PC Sysadmin"

# 3. Tráfico público hacia el servidor WordPress en la DMZ
add chain=forward action=accept in-interface-list=WAN out-interface-list=DMZ dst-address=192.168.11.11 protocol=tcp dst-port=80 comment="Forward: WAN a WordPress DMZ"

# 4. Salida a Internet desde LAN y DMZ
add chain=forward action=accept in-interface-list=LAN out-interface-list=WAN comment="LAN a WAN: Permitida"
add chain=forward action=accept in-interface-list=DMZ out-interface-list=WAN comment="DMZ a WAN: Permitida (actualizaciones)"

# 5. Aislamiento de Seguridad: Prohibir cualquier tráfico originado en la DMZ hacia la LAN
add chain=forward action=drop in-interface-list=DMZ out-interface-list=LAN comment="Aislamiento: Bloquear DMZ hacia LAN"

# 6. Política de Cierre (Default Deny en Forward)
add chain=forward action=drop comment="Default Deny Forward"
```