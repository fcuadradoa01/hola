# Laboratorio MikroTik RouterOS 7: Guía de Segmentación y Hardening

Este tutorial detalla la implementación práctica de las políticas de seguridad (segmentación de red, filtrado entre zonas y endurecimiento del plano de control) para un entorno virtualizado en VMware Workstation con tres zonas: **WAN**, **LAN** y **DMZ**.

---

## Topología de Interfaces de Referencia

*   `ether1`: WAN (Conectada a Internet / NAT)
*   `ether2`: LAN (Red de empleados, ej. `192.168.10.0/24`)
*   `ether3`: DMZ (Servidores Ubuntu 24.04, ej. `192.168.20.0/24`)

---

## Paso 1: Creación de Listas de Interfaces (Interface Lists)

Para mantener un firewall limpio y escalable en RouterOS 7, agrupamos las interfaces físicas en listas lógicas.

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

## Paso 2: Hardening del Router (Cadena Input)

El plano de control del router debe protegerse estrictamente. En este laboratorio aplicaremos los siguientes requisitos específicos:
1. **LAN:** Administración permitida **únicamente** desde una IP específica (ej. `192.168.10.10`) mediante **SSH, Winbox y HTTP**.
2. **WAN:** Administración permitida desde una IP fija externa específica (ej. `192.168.16.123`) **solo mediante SSH** (para demostraciones o pruebas de acceso remoto).

### 1. Configurar puertos y servicios seguros
Desactivamos los servicios no utilizados y aseguramos los puertos de gestión:

```routeros
/ip service
set ssh port=2222
set winbox port=8291
set www port=80
set telnet disabled=yes
set ftp disabled=yes
set api disabled=yes
set api-ssl disabled=yes
set www-ssl disabled=yes
```

### 2. Reglas de Firewall para el plano de control (`input`)

**Las reglas del firewall se evalúan de arriba hacia abajo, una a una**. Cuando un paquete coincide con una regla que tiene una acción definitiva (accept, drop, reject), el proceso se detiene y no se siguen evaluando más reglas.



En casi todos los rúters/firewalls verás algo parecido a:
```routeros
accept established
accept related
drop invalid
````
#### ¿Qué significa established?

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

#### Concepto de firewall stateful (con estado)
Que es precisamente una de las bases del diseño de planes de securización que trabajamos en el RA1.

```routeros
/ip firewall filter

# 1. Permitir conexiones establecidas y relacionadas (vital para el tráfico de vuelta)
add chain=input action=accept connection-state=established,related comment="Aceptar trafico establecido/relacionado"

# 2. Descartar paquetes con estados inválidos
add chain=input action=drop connection-state=invalid comment="Descartar paquetes invalidos"

# 3. Permitir ICMP (Ping) con limitación para evitar ataques de inundación
add chain=input action=accept protocol=icmp limit=50/5s,5 comment="Permitir Ping limitado"

# 4. Administración desde la LAN (Solo IP específica: 192.168.10.50) para SSH, Winbox y HTTP
add chain=input action=accept in-interface-list=LAN src-address=192.168.10.50 protocol=tcp dst-port=2222 comment="Admin LAN: SSH"
add chain=input action=accept in-interface-list=LAN src-address=192.168.10.50 protocol=tcp dst-port=8291 comment="Admin LAN: Winbox"
add chain=input action=accept in-interface-list=LAN src-address=192.168.10.50 protocol=tcp dst-port=80 comment="Admin LAN: HTTP"

# 5. Administración desde la WAN (Solo IP externa específica: 203.0.113.50) solo para SSH
add chain=input action=accept in-interface-list=WAN src-address=203.0.113.50 protocol=tcp dst-port=2222 comment="Admin WAN (Demo): SSH"

# 6. Denegar cualquier otro intento de acceso al router (Default Deny en Input)
add chain=input action=drop comment="Bloquear resto de accesos al router"
```

---

## Paso 3: Segmentación y Políticas de Tráfico (`Forward`)

Para el tráfico que atraviesa el router entre zonas, aplicamos una política estricta de denegación por defecto, permitiendo únicamente el flujo necesario.

### 1. Separar el NAT de la LAN y de la DMZ

```routeros
/ip firewall nat

add chain=srcnat \
    src-address=192.168.10.0/24 \
    out-interface-list=WAN \
    action=masquerade \
    comment="LAN -> Internet"

add chain=srcnat \
    src-address=192.168.20.0/24 \
    out-interface-list=WAN \
    action=masquerade \
    comment="DMZ -> Internet"
```

### 2. Reglas de Firewall para el tráfico de tránsito (`forward`)

```routeros
/ip firewall filter

# 1. Permitir conexiones establecidas y relacionadas
add chain=forward action=accept connection-state=established,related comment="Forward: Permitir establecidas/relacionadas"

# 2. Descartar paquetes inválidos
add chain=forward action=drop connection-state=invalid comment="Forward: Descartar invalidos"

# --- TRÁFICO DESDE LA LAN ---
# Permitir que la LAN salga a Internet (WAN)
add chain=forward action=accept in-interface-list=LAN out-interface-list=WAN comment="LAN a WAN: Permitido"

# Permitir acceso controlado de la LAN a la DMZ (ej. SSH administrativo al servidor web desde una IP de la LAN)
add chain=forward action=accept in-interface-list=LAN out-interface-list=DMZ dst-address=192.168.20.10 protocol=tcp dst-port=22 comment="LAN a DMZ: Admin SSH a Servidor Web"

# --- TRÁFICO DESDE LA DMZ ---
# Permitir que los servidores de la DMZ salgan a Internet estrictamente para actualizaciones (o restringir si es necesario)
add chain=forward action=accept in-interface-list=DMZ out-interface-list=WAN comment="DMZ a WAN: Salida permitida para actualizaciones"

# Permitir comunicación interna de la DMZ: Servidor Web (192.168.20.10) a Base de Datos (192.168.20.20)
add chain=forward action=accept src-address=192.168.20.10 dst-address=192.168.20.20 protocol=tcp dst-port=3306 comment="DMZ Web a BD MySQL"

# --- BLOQUEOS DE AISLAMIENTO (Asegurando la DMZ y LAN) ---
# Bloquear estrictamente cualquier intento de la DMZ hacia la LAN (Movimiento lateral evitado)
add chain=forward action=drop in-interface-list=DMZ out-interface-list=LAN comment="AISLAMIENTO: Bloquear DMZ hacia LAN"

# --- TRÁFICO DESDE LA WAN (Publicación de servicios) ---
# Permitir acceso público HTTP/HTTPS hacia el Servidor Web ubicado en la DMZ (DNAT previo necesario)
add chain=forward action=accept in-interface-list=WAN out-interface-list=DMZ dst-address=192.168.20.10 protocol=tcp dst-port=80 comment="WAN a DMZ: Servidor Web HTTP"
add chain=forward action=accept in-interface-list=WAN out-interface-list=DMZ dst-address=192.168.20.10 protocol=tcp dst-port=443 comment="WAN a DMZ: Servidor Web HTTPS"

# 3. Regla de cierre por defecto para el Forward
add chain=forward action=drop comment="Default Deny Forward"
```

---

## Verificación en el Laboratorio

1. **Prueba de Hardening LAN:** Intenta acceder al router por Winbox o SSH desde otra IP de la LAN que **no** sea `192.168.10.50`; la conexión debe ser rechazada/bloqueada.
2. **Prueba de Aislamiento:** Desde el servidor web de la DMZ, intenta hacer un ping o conexión hacia un equipo de la LAN (`192.168.10.x`); la regla de bloqueo de DMZ a LAN debe descartar el paquete inmediatamente.