---
title: "Cómo funciona una conexión SSH paso a paso"
tags:
  - ssh
  - cifrado
  - conexion
  - simetrico
  - asimetrico
level: basico
---

# Cómo funciona una conexión SSH paso a paso

## Tres preguntas muy simples:
- 🖥️ **¿Con quién estoy hablando?** → Identidad del servidor.
- 🔒 **¿Cómo protegemos la conversación?** → Cifrado.
- 👤 **¿Quién eres tú?** → Autenticación del usuario

## 1. El cliente contacta con el servidor

Por ejemplo:

```Shell
ssh fcuadradoa01@192.168.1.10
```

En este momento todavía:

- No sabemos si el servidor es auténtico.
- No hay sesión establecida.
- No se ha autenticado el usuario.

Simplemente existe una conexión de red.

```
Cliente -------------> Servidor
```

## 2. El servidor presenta su `Host Key`

El servidor tiene una pareja de claves propia:

```
Clave privada del servidor (Host Key privada)
Clave pública del servidor (Host Key pública)
```

Por ejemplo:

```
/etc/ssh/ssh_host_ed25519_key
/etc/ssh/ssh_host_ed25519_key.pub
```

La clave pública se envía al cliente.

!!! tip "importante"
    Esta clave identifica al SERVIDOR, no al usuario.

Aquí suele aparecer la primera confusión. No es:

```
Soy Francisco
```

Sino:

```
Soy el servidor 192.168.1.10"
```

## 3. El cliente comprueba `known_hosts`

El cliente revisa:

```
~/.ssh/known_hosts
```

para comprobar si ya conoce esa `Host Key`.

### Primera conexión

No encuentra ninguna coincidencia.

Entonces aparece el famoso mensaje:

```
The authenticity of host '192.168.1.10' can't be established.
Are you sure you want to continue connecting?
```

Si aceptamos: `yes`

la `Host Key` queda almacenada en: `~/.ssh/known_hosts`

!!! info "a saber"
    La `Host Key` es en realidad la clave pública del servidor, que se genero con la instalación del servicio `ssh`.


### Conexiones posteriores
SSH compara: `Host Key recibida` vs `Host Key almacenada`

- **Si coinciden:**
    
    `Servidor verificado` ✅

- **Si son distintas:**

    SSH sospecha un problema:

    `WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!` ⚠️

    Puede deberse a:

    - Un ataque MITM.
    - Reinstalación del servidor.
    - Regeneración de claves SSH.

    Por seguridad, `SSH` detiene el proceso. 🛑

## 4. Negociación de una clave temporal

Ahora cliente y servidor crean una clave secreta compartida (`simétrica`).

!!! tip "idea"
    Cliente y  Servidor - Generan la misma clave - ¡Sin enviarla por la red!
    
    Esa clave se utilizará solamente durante esa sesión.

## 5. Se crea el canal cifrado

Desde ese momento **TODO** viaja cifrado:

- 👤 Identidad del usuario.
- 🔑 Credenciales de acceso.
- ⌨️ Comandos enviados al servidor.
- 📦 Archivos transferidos.
- 📺 Resultados y salidas por pantalla.

Aunque alguien caputure los paquetes verá datos ilegibles.

## 6. El usuario demuestra quién es

Ahora viene otra comprobación completamente diferente.

El servidor ya sabe: `Soy el servidor legítimo`

Ahora pregunta: `¿Y tú quién eres?`

Existen varios métodos.

### Contraseña (simétrico)

```
Usuario: fcuadradoa01
Contraseña: ********
``` 

### Clave pública (asimétrico)

El cliente tiene: `id_ed25519` (clave privada) y `id_ed25519.pub` (clave pública).

El servidor almacena la pública en: `~/.ssh/authorized_keys`

Si la prueba criptográfica es válida: `Usuario autenticado` y ya podemos ejecutar comandos.

!!! success "resumen"
    `known_hosts` sirve para verificar la identidad del servidor. 
    
    `authorized_keys` sirve para verificar la identidad del usuario.
    
     Son problemas distintos y SSH los resuelve por separado.

---

## 📜 Créditos y Licencia

* **Validación del Material:** Contenidos y prácticas validados por **Paco Cuadrado `fcuadradoa01@educantabria.es`**.
* **Licencia:** Licencia Creative Commons Atribución 4.0 Internacional ([CC BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.es)).
!!! note "CC BY 4.0"
    *Se permite compartir, copiar, redistribuir, adaptar y transformar este material para cualquier propósito (incluso comercial), siempre que se reconozca y mencione adecuadamente la autoría original.*