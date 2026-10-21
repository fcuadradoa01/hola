---
title: "SSH - Permisos ficheros de claves"

tags:
  - ssh
  - linux
  - permisos
  - admin
level: básico
---

# SSH - Permisos ficheros de claves

## carpeta ssh

1. La carpeta ~/.ssh (Permisos: 700)
- Qué significa: Solo tú (el propietario) puedes leer, escribir y entrar (execute) en la carpeta. Nadie más en el servidor puede ver lo que hay dentro.

- Por qué: Si otro usuario malintencionado o un servicio mal configurado en el servidor tuviera permisos para entrar en tu carpeta .ssh, podría cotillear o robar tus archivos.

## fichero clave privada

2. La clave privada `id_ed25519` (Permisos: `600`)

!!! info "Qué significa"
    Solo tú (el propietario) puedes leer y escribir en el archivo. Nadie más puede ni tocarlo ni leerlo.

- Por qué: Es tu secreto más valioso. Quien tenga tu clave privada puede suplantarte en GitHub y en cualquier servidor. Por eso Linux se pone muy estricto: si la clave privada tuviera permisos para que la leyeran otros usuarios (por ejemplo, permisos 644), el sistema SSH se negará a usarla por seguridad y te dará un error.

## fichero clave pública

3. La clave pública `id_ed25519.pub` (Permisos: `644`)

- Qué significa: Tú puedes leerla y escribirla, pero cualquier usuario del sistema puede leerla (aunque no modificarla).

- Por qué: Como su propio nombre indica, es pública. Su único propósito es ser compartida y mostrada al mundo (como hemos hecho al pegarla en GitHub). No representa ningún riesgo de seguridad que otros usuarios del servidor puedan verla, por lo que admite permisos más abiertos.

---

> ¿Por qué la claves no son `700` (sino `600`)?
> Darle permisos de ejecución (+x) a un archivo que es puramente de lectura/escritura es una mala práctica de seguridad.
> Por eso se usa `600` (4 + 2 = 6), que significa: *Solo el propietario puede leerlo y escribirlo, y punto*.

## Resumen rápido de permisos seguros:

Carpeta ~/.ssh/ $\rightarrow$ 700 (Privada total para entrar)

Clave privada (id_ed25519) $\rightarrow$ 600 (Solo lectura/escritura tuya)

Clave pública (id_ed25519.pub) $\rightarrow$ 644 (Lectura pública)

---
## Creditos

Elaborado por Paco Cuadrado `fcuadradoa01@educantabria.es`



