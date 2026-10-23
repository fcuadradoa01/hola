---
title: "Laboratorio Docker: Creación de Imagen Personalizada con Dockerfile"
tags:
  - docker
  - dockerfile
  - image
  - alpine
  - apache2
level: intermedio
---

# Laboratorio Docker: Creación de Imagen Personalizada con Dockerfile

Este tutorial guía paso a paso en la creación de una imagen Docker ligera y personalizada utilizando **Alpine Linux** como sistema base, sobre la cual se instalará el servidor web **Apache2** y se desplegará un archivo `index.html` ya existente en el directorio personal del usuario (`~/index.html`).

---

## 🎯 Objetivos del Laboratorio

1. Comprender la estructura y sintaxis fundamental de un **Dockerfile** (`FROM`, `RUN`, `COPY`, `EXPOSE`, `CMD`).
2. Construir (*build*) una imagen personalizada optimizada sobre la distribución ultra ligera **Alpine Linux**.
3. Desplegar y probar un contenedor servidor web mapeando puertos hacia el host (**Ubuntu Server**).
4. Verificar el correcto funcionamiento del servicio web a través de la consola y el navegador.

---

!!! note "Nota conceptual"
    Los comandos y principios explicados en esta guía corresponden a la especificación estándar de Docker. Por lo tanto, el flujo de trabajo (`build` $\rightarrow$ `run` $\rightarrow$ `verify`) es idéntico e independiente del sistema operativo o entorno donde se ejecute (Ubuntu, Debian, macOS, Windows Server o plataformas Cloud).

---

## Preparación del Entorno de Trabajo

Para mantener una estructura organizada, crearemos un directorio exclusivo para la compilación de la imagen. 

!!! info "Requisito previo"
    Se asume que el alumno ya dispone de un archivo `index.html` creado previamente en la raíz de tu directorio personal (`~/index.html`).

```bash
# 1. Crear directorio del proyecto
mkdir -p ~/laboratorio-docker

# 2. Copiar el index.html existente en el home al directorio de trabajo
cp ~/index.html ~/laboratorio-docker/

# 3. Acceder al directorio del proyecto
cd ~/laboratorio-docker
```

---

## Estructura del Dockerfile Explicada Paso a Paso

Dentro del directorio `~/laboratorio-docker/`, creamos un archivo denominado **`Dockerfile`** (respetando la mayúscula inicial y sin extensión) con la siguiente estructura:

```dockerfile
# 1. Imagen Base
FROM alpine:latest

# 2. Instalación del Servidor Web
RUN apk update && apk add --no-cache apache2

# 3. Copia del Sitio Web
COPY index.html /var/www/localhost/htdocs/index.html

# 4. Declaración de Puerto Explicito
EXPOSE 80

# 5. Comando de Arranque del Contenedor
CMD ["httpd", "-D", "FOREGROUND"]
```

### Explicación de las Directivas:

* **`FROM alpine:latest`**: Define la imagen base sobre la cual se construirá la nueva. En este caso se utiliza ***Alpine Linux*** por su tamaño reducido (apenas ~5 MB de base).
* **`RUN apk update && apk add --no-cache apache2`**: Ejecuta comandos durante el proceso de compilación (*buildtime*). Actualiza los repositorios de Alpine e instala Apache2. La opción `--no-cache` evita guardar archivos temporales del gestor de paquetes para optimizar el peso final de la imagen.
* **`COPY index.html /var/www/localhost/htdocs/index.html`**: Copia el archivo `index.html` del directorio actual del anfitrión a la ruta raíz por defecto donde Apache2 busca los archivos web en Alpine.
* **`EXPOSE 80`**: Sirve como documentación indicando que el contenedor escucha en el puerto `80`. *Nota: No publica el puerto por sí solo, requiere mapeo en el comando `docker run`*.
* **`CMD ["httpd", "-D", "FOREGROUND"]`**: Define el comando principal que se ejecutará automáticamente cuando el contenedor se inicie.

!!! tip "¿Por qué `-D FOREGROUND`?"
    Un contenedor de Docker permanece en estado de ejecución (*running*) únicamente mientras su **proceso principal (PID 1)** siga activo. 

    Por defecto, Apache se inicia como un servicio en segundo plano (*daemon/background*). Si no añadiéramos el parámetro `-D FOREGROUND`, el comando de inicio entregaría el control al sistema y finalizaría inmediatamente, provocando que **Docker detecte que la tarea terminó y apague el contenedor de forma automática**. Al obligar a Apache a ejecutarse en primer plano (*foreground*), mantenemos el contenedor activo de forma continua.

---

## ⚙️ Construcción y Despliegue del Contenedor

### 1. Construir la Imagen (`docker build`)

Desde el directorio `~/laboratorio-docker`, ejecutamos el comando de compilación:

```bash
docker build -t mi-apache-alpine:v1 .
```

* **`-t mi-apache-alpine:v1`**: Asigna un nombre (*tag*) a la imagen personalizada con su respectiva versión.
* **`.`**: Indica el contexto de construcción (el directorio actual donde se encuentran el `Dockerfile` y el `index.html`).

### 2. Verificar la Imagen Creada

Comprobamos que la nueva imagen está lista en el repositorio local de Docker:

```bash
docker images
```

!!! info "tamaño"
    Se observará que el tamaño total de la imagen es sumamente pequeño, rondando entre los `15MB` y `20MB`.

### 3. Desplegar el Contenedor (`docker run`)

Creamos e iniciamos el contenedor basado en la imagen recién construida:

```bash
docker run -d --name servidor-web -p 8080:80 mi-apache-alpine:v1
```

* **`-d`**: Ejecuta el contenedor en modo desatendido (*detached* / segundo plano).
* **`--name servidor-web`**: Asigna un nombre identificativo al contenedor.
* **`-p 8080:80`**: Mapea y redirige el puerto `8080` de la máquina anfitriona (Ubuntu Server) hacia el puerto `80` interno del contenedor.

---

## 🧪 Pruebas de Verificación para los Alumnos

Para confirmar que la imagen y el contenedor funcionan correctamente, los alumnos deberán ejecutar y verificar las siguientes comprobaciones:

### 1. Verificar el Estado del Contenedor
```bash
docker ps
```
* **Resultado esperado:** El contenedor `servidor-web` debe aparecer con el estado `Up` y mostrando la redirección de puertos `0.0.0.0:8080->80/tcp`.

### 2. Comprobación mediante Consola Local (`curl`)
Desde la propia terminal de Ubuntu Server, realizar una petición HTTP local:

```bash
curl http://localhost:8080
```
* **Resultado esperado:** La consola debe imprimir directamente el código HTML de su archivo `index.html`.

### 3. Comprobación desde un Navegador Web Externo
Abrir el navegador web de la máquina cliente o equipo anfitrión e ingresar la siguiente URL:

```text
http://<IP_DE_TU_UBUNTU_SERVER>:8080
```
* **Resultado esperado:** Carga correcta de la página web renderizada en el navegador.

---

## 📜 Créditos y Licencia

* **Validación del Material:** Contenidos y prácticas validados por **Pepe Luis**.
* **Licencia:** Licencia Creative Commons Atribución 4.0 Internacional ([CC BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.es)).

  > *Se permite compartir, copiar, redistribuir, adaptar y transformar este material para cualquier propósito (incluso comercial), siempre que se reconozca y mencione adecuadamente la autoría original.*