# Guía Didáctica: ¿Cómo Funciona TOTP (Time-based One-Time Password)?

TOTP es el estándar de autenticación de doble factor (2FA) utilizado por aplicaciones como Google Authenticator, Microsoft Authenticator o Authy. Está definido en el estándar **RFC 6238** y permite generar códigos numéricos temporales (normalmente de 6 dígitos) que cambian cada 30 segundos.

A continuación, analizaremos detalladamente su funcionamiento técnico, matemática subyacente, seguridad y comparativa frente a otras alternativas.

---

## 1. El "Secreto Compartido" (Shared Secret) y su Transmisión

El núcleo de TOTP se basa en la **criptografía simétrica**: tanto el cliente (tu teléfono) como el servidor (por ejemplo, Google o GitHub) deben poseer exactamente la misma clave secreta.

* **¿Qué es el Secreto Compartido?**
  Es una cadena de caracteres aleatorios generada por el servidor al momento de configurar la autenticación en dos pasos (por ejemplo, `JBSWY3DPEHPK3PXP`).
  
* **¿Cómo se transmite inicialmente?**
  El servidor suele codificar este secreto en formato **Base32** y mostrarlo mediante un **código QR**. La URL contenida en el código QR sigue un formato estándar (`keyuri`):
  
  $$\text{otpauth://totp/Servicio:usuario@email.com?secret=JBSWY3DPEHPK3PXP\&issuer=Servicio}$$

* **Analogía:** Imagina que tú y un amigo acuerdan un libro de claves secreto en persona. A partir de ese momento, ambos pueden enviarse mensajes cifrados basándose en la página y renglón correspondientes a cada minuto, sin necesidad de hablar previamente.

> **Punto Crítico de Seguridad:** El secreto solo se transmite **una vez** (durante el escaneo del QR). Si un atacante intercepta este QR, podrá generar los mismos códigos que tu teléfono de forma indefinida.

---

## 2. El Algoritmo Matemático: HMAC-SHA1 y Factor Tiempo

TOTP es una extensión de **HOTP** (HMAC-based One-Time Password), pero reemplaza un contador incremental por un **contador basado en el tiempo**.

### A. La Variable de Tiempo ($T$)
En lugar de enviar la hora exacta, el algoritmo convierte la hora UNIX (*Epoch Timestamp*, segundos transcurridos desde el 1 de enero de 1970) en un bloque discreto de tiempo de 30 segundos:

$$T = \left\lfloor \frac{\text{Tiempo UNIX Actual}}{\text{Paso de Tiempo (30s)}} \right\rfloor$$

* Si el tiempo actual son `1700000000` segundos:
  $$T = \left\lfloor \frac{1700000000}{30} \right\rfloor = 56666666$$
* Este valor $T$ no cambia durante un intervalo entero de 30 segundos.

### B. El Cifrado (HMAC-SHA1)
Se aplica un código de autenticación de mensajes basado en hash (**HMAC**) combinando la Clave Secreta ($K$) y el contador de tiempo ($T$):

$$\text{HMAC-SHA1}(K, T) \longrightarrow \text{Hash de 160 bits (20 bytes)}$$

### C. Truncamiento Dinámico (Dynamic Truncation)
Como un hash de 20 bytes es inviable para que un usuario lo escriba, se extrae un subconjunto de datos:
1. Se toma el último nibble (4 bits) del hash para usarlo como **offset** (índice de inicio).
2. Se leen 4 bytes a partir de esa posición.
3. Se aplica una máscara lógica `AND` para eliminar el bit de signo (evitando números negativos).
4. Se aplica la operación módulo ($10^6$) para obtener un número de 6 dígitos:

$$\text{Código TOTP} = (\text{Valor de 31 bits}) \pmod{10^6}$$

---

## 3. Autonomía Offline: ¿Por qué no requiere Internet?

Una duda común entre los usuarios es cómo la aplicación móvil genera el código correcto sin tener conexión a datos o Wi-Fi.

* **Sincronización Determinista:** El algoritmo es completamente **determinista**. Si dos dispositivos tienen el mismo secreto ($K$) y la misma hora ($T$), aplicar el algoritmo HMAC arrojará exactamente el mismo resultado.
* **Uso del Reloj Interno:** El smartphone utiliza su reloj interno para calcular $T$. Como la hora UNIX es universal y UTC (no se ve afectada por husos horarios ni cambio de hora de verano), ambos sistemas siempre calculan la misma variable $T$.

---

## 4. Margen de Tolerancia y Resincronización

A pesar de la precisión de los relojes modernos, puede existir cierta desincronización (*clock skew*) entre el móvil y el servidor.

* **Ventana de Validación (Window Size):**
  Para mitigar este desfase, el servidor no valida únicamente el código del intervalo actual ($T$). Comprueba también intervalos adyacentes:
  
  $$\text{Intervalos evaluados} = \{T - 1, T, T + 1\}$$

* **Margen de Tolerancia:**
  Una ventana de $\pm 1$ otorga un margen de $\pm 30$ segundos. Si marcas el código justo cuando caduca, el servidor aún lo considerará válido si cae dentro del rango $T-1$.
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

---

## Resumen Final para Alumnos

* **Fundamento:** TOTP = Secreto Criptográfico + Tiempo Unix actual.
* **Ventaja clave:** Funciona offline y no depende de la red móvil (a diferencia del SMS).
* **Principal límite:** Es susceptible al phishing interactivo, motivo por el cual la industria migra progresivamente hacia **Passkeys/FIDO2**.