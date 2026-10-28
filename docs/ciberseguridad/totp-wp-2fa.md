---
title: "Caso práctico: TOTP en WordPress con WP 2FA"
tags:
  - autenticación
  - 2FA
  - doble factor
  - TOTP
  - Google Authenticador
  - Microsoft Authenticador
  - wordpress
level: basico
---

# Caso práctico: TOTP en WordPress con WP 2FA

## 1. ¿Qué ocurre cuando un usuario activa el 2FA?

Cuando activa TOTP desde WP 2FA:

- El plugin genera un secreto aleatorio.
- Construye un QR.
- El usuario lo escanea con _Google/Microsoft Authenticator_.
- Tanto WordPress como el móvil pasan a conocer el **mismo secreto**.

## 2. ¿Qué contiene realmente el QR?

Contiene algo parecido a:

```
otpauth://totp/WordPress:paco?
secret=JBSWY3DPEHPK3PXP
&issuer=WordPress
```

Lo importante es: `secret=JBSWY3DPEHPK3PXP`

Esa es la **clave compartida**.

A partir de ella se pueden **generar infinitos códigos** TOTP.

---

## 3. ¿Dónde Guarda WP 2FA el Secreto de Cada Usuario?

!!! tip "La Regla de Oro de la Criptografía"
    *Nunca guardes la llave en el mismo sitio que el cofre.* Si un atacante entra en la habitación, no debe encontrar la cerradura y la llave en el mismo cajón.

Para que la autenticación de doble factor (TOTP) funcione, el **secreto compartido** debe estar almacenado de forma segura en **dos lugares distintos**: el dispositivo del usuario y el servidor web.

### 📱 Lado del Cliente (El Dispositivo Móvil)

El secreto se guarda localmente en el almacenamiento seguro de la aplicación de autenticación que elija el usuario:

* **Google Authenticator**
* **Microsoft Authenticator**
* **Authy**
* **Aegis** *(Open Source)*

### 🖥️ Lado del Servidor (WordPress & WP 2FA)

En el servidor, el plugin **WP 2FA** aplica la regla de oro para evitar que un hackeo a la base de datos exponga los secretos de todos los usuarios.

!!! info "El Algoritmo de Cifrado"
    Los secretos no se guardan en texto plano. WP 2FA almacena cada secreto cifrado mediante el algoritmo **AES-256-CTR**. Cuando un usuario intenta iniciar sesión, el servidor descifra temporalmente el secreto en memoria para validar el código introducido.

### 🗝️ La Arquitectura: El Cofre y la Llave Maestra

El sistema divide la información en dos componentes independientes:

| Componente | Qué representa | Dónde se almacena |
| :--- | :--- | :--- |
| **📦 El Cofre** | Los **secretos TOTP** cifrados de los usuarios | Base de datos de WordPress (Tabla `wp_usermeta` / `wp_options`) |
| **🔑 La Llave Maestra** | La **clave de cifrado** (`WP2FA_ENCRYPT_KEY`) | Archivo de configuración `wp-config.php` del servidor |


!!! success "¿Por qué se usa `wp-config.php` para la Llave Maestra?"
    Por defecto, al instalar el plugin, este genera una clave privada única y la escribe directamente en el archivo `wp-config.php`.

!!! warning "¿Por qué es esto crítico para la seguridad?"
    1. El archivo `wp-config.php` vive en el sistema de archivos del servidor (accesible solo por FTP/SSH), **fuera de la base de datos SQL**.
    2. Si un atacante consigue explotar una vulnerabilidad de inyección SQL (SQLi) y descarga toda la base de datos de tu WordPress, **solo obtendrá datos cifrados e inútiles**.
    3. Sin acceso al sistema de archivos para leer `wp-config.php`, el atacante no podrá descifrar los secretos de autenticación.

---

## Resumen
```
┌─────────────────────────────────────────────────────────┐
│                      MÓVIL DEL USUARIO               │
│   App 2FA (Google/MS Auth) ──> Guarda el Secreto     │
└─────────────────────────────────────────────────────────┘
                                ▲
                                │ (Validación en login)
                                ▼
┌─────────────────────────────────────────────────────────┐
│                    SERVIDOR WORDPRESS                │
│                                                      │
│  📦 BASE DE DATOS  ──> Guarda los Secretos CIFRADOS │
│             ▲                                        │
│             │ (Requiere para descifrar)              │
│             ▼                                        │
│  🔑 wp-config.php  ──> Guarda la LLAVE MAESTRA       │
└─────────────────────────────────────────────────────────┘
```

¿Qué sabe WordPress? → El secreto TOTP del usuario.

¿Qué sabe el móvil? → El mismo secreto TOTP.

¿Quién genera el código? → Ambos, de forma independiente.

¿Viaja el código por Internet? → No.

¿Viaja el secreto por Internet en cada login? → No.

¿Puede un robo de contraseña saltarse el 2FA? → No.

¿Puede un compromiso total del servidor afectar al 2FA? → Sí.

!!! abstract "Resumen"
    TOTP mejora muchísimo la seguridad frente al robo de credenciales, pero sigue basándose en un secreto compartido entre el usuario y el servidor. Precisamente para eliminar esa dependencia nacieron `FIDO2` y las `Passkeys`.

---

## 📜 Créditos y Licencia

* **Validación del Material:** Contenidos y prácticas validados por **Paco Cuadrado `fcuadradoa01@educantabria.es`**.
* **Licencia:** Licencia Creative Commons Atribución 4.0 Internacional ([CC BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.es)).
!!! note "CC BY 4.0"
    *Se permite compartir, copiar, redistribuir, adaptar y transformar este material para cualquier propósito (incluso comercial), siempre que se reconozca y mencione adecuadamente la autoría original.*