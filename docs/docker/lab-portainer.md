---
title: "Laboratorio: Administración docker-engine - Portainer"
tags:
  - docker
  - web
  - admin
level: intermedio
---
# Laboratorio: Administración docker-engine - Portainer

> **Docker Administración Web | Laboratorio guiado**

## 🎯 Objetivos

Desplegar y configurar una instancia de Portainer sobre Docker para administrar visualmente un entorno de contenedores, verificando el acceso web seguro y la gestión básica de recursos Docker.

## 1. Crear el volumen da datos Portainer

```bash
docker volume create portainer_data
```
---

## 2. Lanzar el contenedor

```bash
 docker run -d \
  --name portainer \
  --restart=always \
  -p 8000:8000 \
  -p 9443:9443 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest
```
Comprueba que está funcionando:

```bash
docker ps
```

Deberías ver el contenedor ```portainer``` en estado ```Up```.

## 3. Abrir la interfaz web
Desde tu navegador: ```https://IP_DEL_SERVIDOR:9443 ```


⚠️ La primera vez aparecerá una advertencia del certificado SSL autofirmado. Es normal.


## 4. Crear usuario administrador
En el primer acceso:

- Usuario: el que quieras (por ejemplo ```admin```)
- Contraseña: una robusta
- Selecciona ```Get Started```
- Después elige: ```Local Environment```

---

## 📜 Créditos y Licencia

* **Validación del Material:** Contenidos y prácticas validados por **Paco Cuadrado `fcuadradoa01@educantabria.es`**.
* **Licencia:** Licencia Creative Commons Atribución 4.0 Internacional ([CC BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.es)).
!!! note "CC BY 4.0"
    *Se permite compartir, copiar, redistribuir, adaptar y transformar este material para cualquier propósito (incluso comercial), siempre que se reconozca y mencione adecuadamente la autoría original.*
