---
title: "FIDO2: autenticación resistente al phishing"
tags:
  - autenticación
  - 2FA
  - doble factor
  - FIDO2
  - phising
  - WebAuthn
  - FaceID
  - TOTP
level: basico
---

# FIDO2: autenticación resistente al phishing

## ¿Qué es FIDO2?

**FIDO2** es un _estándar de autenticación_ que permite iniciar sesión sin depender únicamente de contraseñas y que está diseñado para resistir ataques de phishing.

Ha sido desarrollado por la _FIDO Alliance_ y es utilizado por servicios como Microsoft 365, Google, GitHub, Amazon o AWS.

!!! info "phising"
    Técnica de **ingeniería social** en la que un atacante suplanta la identidad de una entidad legítima para engañar a la víctima.

!!! info "FIDO Alliance"
    Asociación industrial abierta dedicada a crear estándares de autenticación globalmente interoperables. Su objetivo es eliminar el uso de contraseñas tradicionales reemplazándolas por métodos seguros basados en criptografía de llave pública, como la biometría o las llaves físicas de seguridad.

## El problema de las contraseñas

Las contraseñas presentan varios inconvenientes:

- 🎯 Se pueden adivinar.
- 🔁 Se pueden reutilizar en varios servicios.
- 📂 Se pueden filtrar en una brecha de seguridad.
- 🎣 Se pueden robar mediante phishing.

Por eso se empezaron a utilizar segundos factores como:

- SMS/correo electrónico.
- Aplicaciones TOTP (Google Authenticator, Microsoft Authenticator).
- Notificaciones Push.

!!! warning "vulnerable"
    Sin embargo, muchos de estos métodos siguen siendo vulnerables a ataques de _phishing_ avanzados.

## ¿Cómo funciona FIDO2?

FIDO2 utiliza criptografía de clave pública.

Cuando un usuario registra un dispositivo:

- Se genera un par de claves: `Clave privada` + `Clave pública`

- La `clave pública` se envía al servidor.

- La `clave privada` **nunca** abandona el dispositivo del usuario.

## Inicio de sesión con FIDO2

Cuando el usuario quiere autenticarse:

El servidor envía un desafío criptográfico.
El dispositivo lo firma con su clave privada.
El servidor verifica la firma con la clave pública almacenada.
Si la verificación es correcta, el acceso se concede.

## ¿Dónde se guarda la clave privada?

La clave privada suele almacenarse de forma segura entre otros:

- Una Passkey.

- Una YubiKey.

- Windows Hello.

- Ecosistema Apple: FaceID.

- Un teléfono móvil.

- Un TPM (Trusted Platform Module).

!!! tip "importante"
    La clave privada nunca se envía al servidor.

!!! info "Passkey"
    Es el término de la industria para referirse a una **credencial FIDO2**. Lo que cambia entre los diferentes métodos no es el concepto, sino dónde se almacena esa credencial y cómo se desbloquea.

!!! info "Yubikey"
    Es un dispositivo de **autenticación física** en formato de llave-USB (ahora tambien NFC) desarrollado por Yubico que implementa estándares avanzados de seguridad como FIDO2, U2F y OTP.

!!! info "Windows Hello"
    Es un sistema de autenticación biométrica y basada en PIN integrado en Windows. En nuestro caso, actúa de forma nativa como un autenticador FIDO2, utilizando la biometría o el PIN del dispositivo como credenciales criptográficas conformes con este estándar abierto.

!!! info "TPM"
    El chip TPM (Trusted Platform Module -Módulo de Plataforma Confiable) es un **componente físico integrado en la placa base** del equipo, por lo que a nivel de hardware es único y compartido por toda la máquina.

    Sin embargo, sí puede ser utilizado por diferentes usuarios que inicien sesión en el mismo ordenador, pero bajo un principio estricto de aislamiento criptográfico por cuenta.

## ¿Por qué FIDO2 es más seguro que TOTP?

Con **TOTP** el usuario escribe un código:

`Usuario -> 123456`

Si una web falsa solicita ese código, el atacante puede reutilizarlo inmediatamente.

Con **FIDO2** no existe ningún código que el usuario pueda copiar o entregar.

Además, la autenticación está vinculada al dominio real.

Por ejemplo:

```
✅ login.microsoftonline.com

❌ micr0soft-login.com
```

Si el dominio es falso, la autenticación no funcionará.

## FIDO2/WebAuthn

Desglosémoslo de forma técnica pero directa:

- **WebAuthn (Web Authentication API)**: Es el estándar oficial (de la W3C) integrado en los navegadores web. Es la interfaz que permite que una página web se comunique con tu sistema operativo para registrarte o iniciar sesión usando criptografía en lugar de contraseñas.

- **FIDO2**: Es el marco global creado por la FIDO Alliance que engloba a WebAuthn y añade el protocolo CTAP (Client-to-Authenticator Protocol). Este último es el que permite que un dispositivo o llave de seguridad (como tu móvil, una YubiKey o el chip TPM de tu PC) se comunique de forma segura con el navegador.

!!! tip "ejemplos cotidianos que probablemente ya utilices o conozcas"
    Al acceder a la app de tu banco, a la web de Amazon, eBay o Google desde el navegador móvil usando Touch ID, Face ID o la huella de Android.

    Por debajo: La web invoca **WebAuthn**, el sistema operativo utiliza la biometría local para autenticarte y **FIDO2** valida el acceso sin que introduzcas la contraseña.


## Ventajas de FIDO2

- ✅ Más seguro que las contraseñas.

- ✅ Resistente al phishing.

- ✅ No requiere escribir códigos.

- ✅ La clave privada nunca abandona el dispositivo.

- ✅ Compatible con múltiples servicios.

## Idea clave para recordar

- 🔑 TOTP demuestra que tienes el teléfono.

- 🔐 FIDO2 demuestra que tienes el dispositivo y que además estás hablando con el servidor legítimo.

!!! success "Objetivo"
    Esa es la razón por la que FIDO2 y las Passkeys se consideran actualmente uno de los métodos de autenticación más seguros para usuarios y organizaciones.

---

## 📜 Créditos y Licencia

* **Validación del Material:** Contenidos y prácticas validados por **Paco Cuadrado `fcuadradoa01@educantabria.es`**.
* **Licencia:** Licencia Creative Commons Atribución 4.0 Internacional ([CC BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.es)).
!!! note "CC BY 4.0"
    *Se permite compartir, copiar, redistribuir, adaptar y transformar este material para cualquier propósito (incluso comercial), siempre que se reconozca y mencione adecuadamente la autoría original.*