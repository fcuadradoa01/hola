---
title: "Guía Didáctica: ¿Cómo Funciona TOTP (Time-based One-Time Password)?"
tags:
  - autenticación
  - 2FA
  - doble factor
  - TOTP
  - Google Authenticador
  - Microsoft Authenticador
level: basico
---

# ¿Cómo Funciona TOTP (Time-based One-Time Password)?

TOTP es el estándar de autenticación de doble factor (2FA) utilizado por aplicaciones como Google Authenticator, Microsoft Authenticator o Authy. Está definido en el estándar **RFC 6238** y permite generar códigos numéricos temporales (normalmente de 6 dígitos) que cambian cada 30 segundos.

---

## 1. El "Secreto Compartido" (Shared Secret) y su Transmisión

El núcleo de TOTP se basa en la **criptografía simétrica**: tanto el cliente (tu teléfono) como el servidor (por ejemplo, Google o GitHub) deben poseer exactamente la misma clave secreta.

* **¿Qué es el Secreto Compartido?**
  Es una cadena de caracteres aleatorios generada por el servidor al momento de configurar la autenticación en dos pasos (por ejemplo, `JBSWY3DPEHPK3PXP`).
  
* **¿Cómo se transmite inicialmente?**
  El servidor suele codificar este secreto en formato **Base32** y mostrarlo mediante un **código QR**. 

!!! info "Punto Crítico de Seguridad"
    El secreto solo se transmite **una vez** (durante el escaneo del QR). Si un atacante intercepta este QR, podrá generar los mismos códigos que tu teléfono de forma indefinida.

---

## 2. El Algoritmo Matemático y Factor Tiempo

TOTP es una extensión de **HOTP** (HMAC-based One-Time Password), pero reemplaza un contador incremental por un **contador basado en el tiempo**.

La app toma dos ingredientes: la clave secreta que compartiste al principio y el bloque de tiempo actual (los 30 segundos en los que estamos).

El cálculo lo hacen los dos por separado:

- En tu **móvil**: La app _Google Authenticator_ (o similar) calcula el código usando su reloj y el secreto.

- En la **nube**: El servidor de la web (el de Google, GitHub, tu banco...) hace exactamente el mismo cálculo usando su propio reloj y el secreto que tiene guardado en su base de datos.

!!! info "reloj"
    La sincronización de los relojes es la clave absoluta de todo el sistema, aunque un desfase de segundo no es relevante.

---

## 3. Autonomía Offline: ¿Por qué no requiere Internet?

El servidor nunca te tiene que enviar el código por internet.

!!! tip "duda"
    Común entre los usuarios es cómo la aplicación móvil genera el código correcto sin tener conexión a datos o Wi-Fi.

* **Sincronización Determinista:** El algoritmo es completamente **determinista**. Si dos dispositivos tienen el mismo secreto `($K$)` y la misma hora `($T$)`, aplicar el algoritmo `HMAC` arrojará exactamente el mismo resultado.
* **Uso del Reloj Interno:** El smartphone utiliza su reloj interno para calcular `$T$`. Como la hora UNIX es universal y UTC (no se ve afectada por husos horarios ni cambio de hora de verano), ambos sistemas siempre calculan la misma variable `$T$`.

---

## 4. Margen de Tolerancia y Resincronización

A pesar de la precisión de los relojes modernos, puede existir cierta desincronización (*clock skew*) entre el móvil y el servidor.

* **Ventana de tolerancia:**
      
      - Para mitigar este desfase, lo habitual es que el servidor compruebe no solo el intervalo de tiempo actual, sino también el intervalo anterior y el posterior (es decir, $-1$, $0$ y $+1$).
  
      - Traducido a los bloques de 30 segundos, significa que el servidor acepta códigos en una franja de 90 segundos en total

* **Estrategia anti-replay:** Una vez que un código en un intervalo específico ($T$) ha sido aceptado por el servidor, este se marca como consumido para evitar ataques de retransmisión (*replay attacks*) dentro de los mismos 30 segundos.

---

## 5. Vectores de Ataque y Comparativa frente a SMS

### Vectores de Ataque contra TOTP
1. **Phishing en Tiempo Real (Adversary-in-the-Middle):** Plataformas como Evilginx capturan las credenciales del usuario y la cookie/código TOTP en tiempo real, reenviándolo al servidor legítimo antes de que expire.
2. **Robo del Secreto:** Malware en el móvil o vulnerabilidades en el servidor que almacena la clave compartida $K$.
3. **Ataques de Desincronización del Reloj:** Alterar la hora del servidor o dispositivo para invalidar la generación de códigos.

### ¿Por qué TOTP es superior al SMS?
* **Susceptibilidad a SIM Swapping:** Un atacante puede duplicar tu tarjeta SIM mediante ingeniería social con la operadora telefónica y recibir tus mensajes SMS. TOTP no depende de redes telefónicas.
* **Interceptación de Red:** Los SMS se transmiten en texto plano sobre protocolos de telefonía vulnerables como SS7/Diameter.
* **Malware de Lectura:** El permiso para leer SMS en sistemas operativos suele ser más explotado por troyanos bancarios que el acceso al almacenamiento seguro del llavero de una app TOTP.

---

## 6. Debilidades de TOTP frente a FIDO2 / Passkeys

Aunque TOTP es una mejora considerable respecto a las contraseñas simples y SMS, las **Passkeys (FIDO2 / WebAuthn)** representan el estándar moderno más seguro por varias razones:

| Característica | TOTP | Passkeys (FIDO2 / WebAuthn) |
| :--- | :--- | :--- |
| **Resistencia al Phishing** | ❌ **No**. El usuario puede copiar manualmente el código en una web falsa. | ✅ **Sí**. Criptográficamente vinculado al dominio web legítimo (*Domain Binding*). |
| **Secreto Compartido** | ❌ **Sí**. El servidor guarda la clave $K$. Si el servidor es hackeado, el secreto se filtra. | ✅ **No**. Usa criptografía de clave pública/privada. El servidor solo guarda la clave pública. |
| **Intervención Humana** | ❌ Requiere leer e introducir 6 dígitos. | ✅ Flujo transparente con biometría (FaceID, huella). |
| **Ataque AiTM** | ❌ Vulnerable a proxying en tiempo real. | ✅ Inmune (el navegador valida el origen del dominio). |

!!! info "AiTM"
    AiTM (Adversary in the Middle), una evolución del clásico Man-in-the-Middle.
    Imagina que un atacante monta una página de inicio de sesión **falsa** que en realidad funciona como un proxy en tiempo real. Cuando el usuario introduce: _Usuario_, _Contraseña_, _Código TOTP_, el atacante los reenvía instantáneamente al servicio legítimo y suplante identidad.

---

## Resumen Final para Alumnos

* **Fundamento:** TOTP = Secreto Criptográfico + Tiempo Unix actual.
* **Ventaja clave:** Funciona offline y no depende de la red móvil (a diferencia del SMS).
* **Principal límite:** Es susceptible al phishing interactivo, motivo por el cual la industria migra progresivamente hacia **Passkeys/FIDO2**.

## 📜 Créditos y Licencia

* **Validación del Material:** Contenidos y prácticas validados por **Paco Cuadrado `fcuadradoa01@educantabria.es`**.
* **Licencia:** Licencia Creative Commons Atribución 4.0 Internacional ([CC BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.es)).
!!! note "CC BY 4.0"
    *Se permite compartir, copiar, redistribuir, adaptar y transformar este material para cualquier propósito (incluso comercial), siempre que se reconozca y mencione adecuadamente la autoría original.*