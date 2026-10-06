# Laboratorio: WordPress + MariaDB con redes Docker

> **Docker Networking | Laboratorio guiado**

## 🎯 Objetivos

En este laboratorio desplegarás una aplicación **WordPress** utilizando dos contenedores Docker:

- Un contenedor con **WordPress + Apache**.
- Un contenedor con **MariaDB** como servidor de bases de datos.

La infraestructura utilizará dos redes Docker independientes:

- `red_publica`
- `red_privada`

El resultado final será:

```text
                         USUARIO
                            │
                            │ HTTP :8080
                            ▼
                       HOST DOCKER
                            │
                            ▼
              ═══════ red_publica ═══════
                            │
                     ┌──────▼──────┐
                     │  WordPress  │
                     │   Apache    │
                     └──────┬──────┘
                            │
                       SQL :3306
                            │
              ═══════ red_privada ═══════
                            │
                     ┌──────▼──────┐
                     │   MariaDB   │
                     └─────────────┘
```

La idea fundamental es que **WordPress estará conectado a las dos redes**, mientras que **MariaDB solamente pertenecerá a la red privada**.

---

## 1. Crear las redes

Consulta primero las redes existentes:

```bash
docker network ls
```

Docker dispone de una red `bridge` por defecto, pero en este laboratorio utilizaremos nuestras propias redes `bridge`.

Crea `red_publica`:

```bash
docker network create \
  --driver bridge \
  --subnet 172.20.0.0/24 \
  red_publica
```

Crea `red_privada`:

```bash
docker network create \
  --driver bridge \
  --subnet 172.21.0.0/24 \
  red_privada
```

Comprueba el resultado:

```bash
docker network ls
```

Puedes obtener información detallada de cada red con:

```bash
docker network inspect red_publica
```

```bash
docker network inspect red_privada
```

> ### 🧠 CHECKPOINT
> Observa las subredes:
>
> ```text
> red_publica  → 172.20.0.0/24
> red_privada  → 172.21.0.0/24
> ```
>
> **Si dos contenedores están conectados a redes diferentes, ¿podrán comunicarse directamente entre ellos?**
>
> Piensa la respuesta antes de continuar.

---

## 2. Crear el servidor MariaDB

Comenzaremos por la base de datos. MariaDB debe encontrarse exclusivamente en:

```text
red_privada
```

Crea el contenedor:

```bash
docker run -d \
  --name mariadb \
  --network red_privada \
  -e MARIADB_ROOT_PASSWORD=root1234 \
  -e MARIADB_DATABASE=wordpress \
  -e MARIADB_USER=wordpress \
  -e MARIADB_PASSWORD=wp1234 \
  mariadb
```

> **Nota de laboratorio:** las contraseñas son deliberadamente sencillas. No deben utilizarse credenciales de este tipo en un entorno real.

Comprueba que el contenedor está funcionando:

```bash
docker ps
```

Si quieres examinar su proceso de inicialización:

```bash
docker logs mariadb
```

Ahora observa la red:

```bash
docker network inspect red_privada
```

Localiza el contenedor `mariadb` y comprueba qué dirección IP le ha asignado Docker.

> ### 🧠 CHECKPOINT
> Observa `docker ps`. MariaDB utiliza el puerto:
>
> ```text
> 3306/tcp
> ```
>
> Sin embargo, **no hemos utilizado**:
>
> ```bash
> -p 3306:3306
> ```
>
> **¿Qué diferencia crees que existe entre que MariaDB utilice el puerto 3306 y que nosotros publiquemos ese puerto en el host Docker?**
>
> No hace falta una definición formal. Lo importante es distinguir entre **usar un puerto dentro de Docker** y **hacerlo accesible desde el exterior**.

---

## 3. Crear el servidor WordPress

Ahora desplegaremos WordPress. De momento lo conectaremos únicamente a:

```text
red_publica
```

Ejecuta:

```bash
docker run -d \
  --name wordpress \
  --network red_publica \
  -p 8080:80 \
  -e WORDPRESS_DB_HOST=mariadb:3306 \
  -e WORDPRESS_DB_NAME=wordpress \
  -e WORDPRESS_DB_USER=wordpress \
  -e WORDPRESS_DB_PASSWORD=wp1234 \
  wordpress
```

Observa especialmente:

```bash
-p 8080:80
```

Estamos haciendo accesible el puerto `80` del contenedor mediante el puerto `8080` del host:

```text
Navegador
    │
    ▼
HOST DOCKER :8080
    │
    ▼
WORDPRESS :80
```

> ### 🧠 CHECKPOINT
> Compara los dos contenedores:
>
> ```text
> WordPress → hemos utilizado -p 8080:80
> MariaDB   → no hemos utilizado -p
> ```
>
> **¿Por qué tiene sentido publicar el servidor web pero no publicar el servidor de bases de datos?**

---

## 4. Tenemos un problema

Comprueba las dos redes:

```bash
docker network inspect red_publica
```

```bash
docker network inspect red_privada
```

En este momento tienes:

```text
red_publica
└── wordpress

red_privada
└── mariadb
```

Pero recuerda que hemos configurado WordPress para conectarse a:

```text
mariadb:3306
```

Tenemos, por tanto, una aplicación en la que WordPress necesita hablar con MariaDB, pero ambos están en redes diferentes.

```text
red_publica                 red_privada

┌───────────┐               ┌───────────┐
│ WordPress │       X       │  MariaDB  │
└───────────┘               └───────────┘
```

> ### 🧠 CHECKPOINT
> **Antes de continuar, piensa: ¿qué le falta a nuestra arquitectura para que WordPress y MariaDB puedan comunicarse?**
>
> Pista: la solución **no pasa por publicar el puerto 3306 de MariaDB**.

---

## 5. Conectar WordPress a la red privada

WordPress necesita poder atender a los usuarios, pero también comunicarse con la base de datos. Por tanto, tendrá que pertenecer a **las dos redes**.

Conecta el contenedor existente a `red_privada`:

```bash
docker network connect red_privada wordpress
```

Comprueba el resultado:

```bash
docker network inspect red_publica
```

```bash
docker network inspect red_privada
```

Ahora deberías tener:

```text
red_publica
└── wordpress

red_privada
├── wordpress
└── mariadb
```

También puedes inspeccionar directamente WordPress:

```bash
docker inspect wordpress
```

Localiza la información de `NetworkSettings` y `Networks`. WordPress debería aparecer conectado a:

```text
red_publica
red_privada
```

La arquitectura comienza a parecerse a la siguiente:

```text
              ═══════ red_publica ═══════
                            │
                     ┌──────▼──────┐
                     │  WordPress  │
                     └──────┬──────┘
                            │
              ═══════ red_privada ═══════
                            │
                     ┌──────▼──────┐
                     │   MariaDB   │
                     └─────────────┘
```

> ### 🧠 CHECKPOINT
> **¿Cuántas direcciones IP tendrá ahora el contenedor WordPress? ¿Por qué?**
>
> Relaciona tu respuesta con un ordenador físico o una máquina virtual que tenga **dos tarjetas de red**.

---

## 6. WordPress encuentra a MariaDB

Hay un detalle importante. Cuando configuramos WordPress escribimos:

```bash
WORDPRESS_DB_HOST=mariadb:3306
```

No escribimos una dirección como:

```text
172.21.0.X:3306
```

Vamos a comprobar qué ocurre.

Entra en WordPress:

```bash
docker exec -it wordpress bash
```

Dentro del contenedor ejecuta:

```bash
getent hosts mariadb
```

Deberías obtener la dirección IP que Docker ha asignado al contenedor `mariadb` dentro de la red que ambos comparten.

Sal del contenedor:

```bash
exit
```

> ### 🧠 CHECKPOINT
> **¿Qué ventaja tiene configurar WordPress utilizando el nombre `mariadb` en lugar de escribir directamente la dirección IP del contenedor?**
>
> Pista: piensa qué podría ocurrir con esa IP si mañana eliminamos el contenedor y lo volvemos a crear.

---

## 7. Probar WordPress

Ya tenemos la arquitectura necesaria:

```text
                        HOST
                         │
                       :8080
                         │
                         ▼
                    WordPress
                         │
                         │ :3306
                         ▼
                     MariaDB
```

Desde un navegador accede a:

```text
http://IP_DEL_SERVIDOR:8080
```

Por ejemplo, si la IP de tu servidor Docker fuese `192.168.1.50`:

```text
http://192.168.1.50:8080
```

Si todo funciona correctamente aparecerá el asistente inicial de WordPress.

Completa la instalación y comprueba que puedes acceder al sitio.

> ### 🧠 CHECKPOINT
> Cuando cargas una página de WordPress se producen dos comunicaciones diferentes:
>
> ```text
> Cliente ──────► WordPress
>
> WordPress ────► MariaDB
> ```
>
> **¿Por cuál de nuestras dos redes Docker circula cada una?**
>
> Si puedes seguir mentalmente esas dos comunicaciones, tienes comprendida la idea fundamental del laboratorio.

---

## 8. Romper para comprender

Una buena forma de entender una arquitectura es romperla de forma controlada.

Desconecta WordPress de la red privada:

```bash
docker network disconnect red_privada wordpress
```

Intenta volver a cargar WordPress desde el navegador.

Después vuelve a conectarlo:

```bash
docker network connect red_privada wordpress
```

Comprueba que la aplicación vuelve a funcionar correctamente.

> ### 🧠 CHECKPOINT
> Al desconectar `red_privada`, **ningún contenedor ha sido eliminado y ningún servicio ha sido desinstalado**.
>
> **Entonces, ¿qué es exactamente lo que hemos roto?**
>
> Intenta expresarlo utilizando los conceptos **conectividad**, **red privada** y **base de datos**.

---

## 9. Resultado final

Tu infraestructura debe quedar finalmente así:

```text
                         CLIENTE
                            │
                            │ HTTP
                            ▼
                      HOST DOCKER
                        TCP 8080
                            │
                            ▼
                ═════ red_publica ═════
                            │
                     ┌──────▼──────┐
                     │  WordPress  │
                     │   Apache    │
                     │    PHP      │
                     └──────┬──────┘
                            │
                       TCP 3306
                            │
                            ▼
                ═════ red_privada ═════
                            │
                     ┌──────▼──────┐
                     │   MariaDB   │
                     └─────────────┘
```

---

## 📸 Evidencias

### Evidencia 1. Arquitectura de red

Realiza una captura de:

```bash
docker network inspect red_privada
```

En ella deben poder identificarse:

- `wordpress`
- `mariadb`
- Las direcciones IP asignadas.

### Evidencia 2. Servicio

Realiza una captura del navegador mostrando **WordPress funcionando correctamente** y el acceso mediante el puerto `8080`.

---

## ✅ Autoevaluación final

Antes de considerar terminado el laboratorio, observa nuevamente esta representación:

```text
EXTERIOR
   │
   ▼
WordPress
   │
   ▼
MariaDB
```

Deberías ser capaz de explicar, sin consultar los pasos anteriores:

> **¿Por qué WordPress necesita dos redes, por qué MariaDB solo necesita una y por qué no ha sido necesario publicar el puerto 3306 de MariaDB?**

Si puedes explicarlo con tus propias palabras, has entendido la idea principal del laboratorio.

---

## 🧹 Opcional: limpiar el laboratorio

Cuando el profesor lo indique, puedes eliminar los recursos creados.

Primero elimina los contenedores:

```bash
docker rm -f wordpress mariadb
```

Después elimina las redes:

```bash
docker network rm red_publica red_privada
```

Comprueba el resultado:

```bash
docker ps -a
docker network ls
```
