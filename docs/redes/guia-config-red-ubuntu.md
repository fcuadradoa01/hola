---
title: "Ubuntu Server 26.04 - Configuración de Red con Netplan"

tags:
  - ubuntu
  - red
  - netplan
  - configuración
level: básico
---

# Guía: Ubuntu Server 26.04 - Configuración de Red con Netplan

> **Guía | Configuración de la red en Ubuntu Server 26.04**

## 🎯 Objetivo

Aprender a configurar la conectividad de red en Ubuntu Server mediante Netplan, tanto utilizando direcciones IP obtenidas por DHCP como configuraciones estáticas.

## 1. Identificar las interfaces de red

Antes de configurar la red, conviene comprobar los nombres de las interfaces instaladas en el sistema:

```bash
ip a
```

---

## 2. Localizar el fichero de configuración

Netplan almacena la configuración en:


```bash
/etc/netplan/
```

Normalmente encontraremos un fichero similar a:

```bash
/etc/netplan/00-installer-config.yaml
```

⚠️ En lugar de `00` puede mostrar otro número.

Lo editamos con `nano`:

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

## 3. Una interfaz de red mediante DHCP

Escenario habitual en laboratorios donde el router asigna automáticamente la configuración IP.

```yaml
network:
  version: 2
  ethernets:
    ens18:
      dhcp4: true
```

- `ens18` = es el nombre del adaptador de red.
- `dhcp4` = IP dinámica sobre IPv4.

### Aplicar cambios

... y ejecutamos

```bash
sudo netplan apply
```

- **Qué hace**: Lee tus archivos de configuración YAML en /etc/netplan/ y aplica los cambios de red de forma inmediata y definitiva.

- **Precaución**: Si cometes un error en la configuración (por ejemplo, una IP mal puesta o una interfaz incorrecta), puedes perder la conexión al servidor de inmediato (especialmente problemático si estás conectado por SSH).

> 🛡️ `sudo netplan try` Es el hermano "**seguro**" de `apply`. Aplica los cambios de forma temporal y te muestra un mensaje en pantalla pidiendo que confirmes

### Comprobar

```bash
ip a
ip route
```

## 4. Una interfaz con IP estática

Escenario habitual en servidores. Datos de ejemplo:

  - IP `192.168.1.10/24`
  - Gateway `192.168.1.1`
  - DNS `8.8.8.8, 1.1.1.1`

### Configuración

```yaml
network:
  version: 2
  ethernets:
    ens18:
      dhcp4: false
      addresses:
        - 192.168.1.10/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
```

... y, como ya sabemos, aplicar cambios `sudo netplan apply`

## 5. Dos interfaces: una DHCP y otra estática

Escenario típico en máquinas que actúan como servidor entre dos redes.

Datos de ejemplo:

- `ens18` (Red externa)
  - Obtiene la configuración automáticamente mediante DHCP.

- `ens19` (Red interna)
  - IP `192.168.10.10/24`

### Configuración

```bash
network:
  version: 2
  ethernets:
    ens18:
      dhcp4: true

    ens19:
      dhcp4: false
      addresses:
        - 192.168.10.10/24
```

... y, como ya sabemos, aplicar cambios `sudo netplan apply`

> 💡 `ens19` no lleva ni puerta enlace ni DNS ya que vienen configurados por DHCP sobre `ens18`.

## 6. Dos interfaces con IP estática

Escenario típico en servidores con varias redes conectadas.

Datos de ejemplo:

- `ens18` (Red externa)
  - IP `192.168.1.50/24`
  - Gateway `192.168.1.1/24`

- `ens19` (Red interna)
  - IP `192.168.10.10/24`

### Configuración

```yaml
network:
  version: 2
  ethernets:
    ens18:
      addresses:
        - 192.168.1.50/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
    ens19:
      addresses:
        - 192.168.10.1/24
```

... y, como ya sabemos, aplicar cambios `sudo netplan apply`

## 📜 Créditos y Licencia

* **Validación del Material:** Contenidos y prácticas validados por **Paco Cuadrado `fcuadradoa01@educantabria.es`**.
* **Licencia:** Licencia Creative Commons Atribución 4.0 Internacional ([CC BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.es)).
!!! note "CC BY 4.0"
    *Se permite compartir, copiar, redistribuir, adaptar y transformar este material para cualquier propósito (incluso comercial), siempre que se reconozca y mencione adecuadamente la autoría original.*
