---
title: "Apuntes de Redes: TCP vs UDP, Puertos y Sockets"

tags:
  - tcp
  - udp
  - puertos
  - tcp/ip
level: básico
---

# Apuntes de Redes: TCP vs UDP, Puertos y Sockets

## 1. TCP vs UDP

### TCP (Transmission Control Protocol)

* **Orientado a conexión:** Se establece una conexión sólida entre el cliente y el servidor (*handshake* o saludo en tres vías) antes de transmitir cualquier dato.

* **Garantía:** Asegura que los datos lleguen completos, sin errores y en el orden correcto.

!!! note "analogía"
    Analogía de la llamada telefónica. TCP es como llamar por teléfono: marcas, la otra persona descuelga, dices "hola", te responden "hola" y recién ahí empiezas a hablar. Si se corta una palabra, pides que te la repitan.

### UDP (User Datagram Protocol)

* **Sin conexión:** Permite el envío de datagramas de forma rápida en redes IP sin establecer previamente una conexión.

* **Autocontenido:** El propio datagrama incorpora suficiente información sobre el destinatario en su cabecera. No garantiza recepción ni orden, pero prioriza la velocidad.


!!! note "analogía"
    Analogía de la carta o el streaming. UDP es como enviar una postal por correo o gritar en una habitación concurrida: lo lanzas y confías en que llegue. Se usa en videojuegos online y llamadas de Discord/Zoom; si se pierde un paquete, prefieres un pequeño "lag" momentáneo antes que congelar la pantalla esperando recuperar ese frame antiguo.

## 2. Categorías de Puertos TCP y UDP

Los puertos permiten a un sistema operativo saber a qué aplicación o servicio específico va dirigida la información que llega por la red. 

Se organizan en tres categorías según la IANA (Internet Assigned Numbers Authority):

1. **Puertos Bien Conocidos (*Well-Known Ports*):**

    - **Rango:** `0` al `1023`

    - Son números fijos y estandarizados para servicios fundamentales de red.

2. **Puertos Registrados (*Registered Ports*):**

    - **Rango:** `1024` al `49151`

    - Reservados por organizaciones o empresas de software ante la IANA para aplicaciones específicas.

3. **Puertos Dinámicos / Privados (*Dynamic / Ephemeral Ports*):**

    - **Rango:** `49152` al `65535`

    - Se asignan de forma temporal y dinámica por el sistema operativo.

    - *Ejemplo:* Cuando abres tu navegador para visitar una web, tu PC usa un puerto de este tipo como origen. Al cerrar la pestaña o terminar la sesión, el número vuelve a quedar libre.

!!! note "analogía"
    La analogía del edificio. La dirección IP es la dirección de un edificio de pisos. El número de puerto es el número del apartamento. ¡Sin el número de puerto, el cartero (tu tarjeta de red) no sabe a qué vecino entregarle la carta!

## 3. Tabla: Puertos y Aplicaciones Frecuentes

| Puerto | Acrónimo | Servicio / Aplicación | Protocolo | 
| ----- | ----- | ----- | ----- | 
| **20 / 21** | FTP | Transferencia de archivos | TCP | 
| **22** | SSH | Shell seguro (administración remota) | TCP | 
| **23** | Telnet | Administración remota en texto plano *(en desuso por inseguro)* | TCP | 
| **53** | DNS | Servicio de nombres de dominio | UDP / TCP | 
| **67 / 68** | DHCP | Asignación dinámica de direcciones IP | UDP | 
| **69** | TFTP | Transferencia trivial de archivos (sin autenticación) | UDP | 
| **80** | HTTP | Transferencia de hipertexto (Web no segura) | TCP | 
| **443** | HTTPS | Transferencia de hipertexto seguro (Web cifrada) | TCP | 
| **3306** | \- | Conexión a bases de datos MySQL / MariaDB | TCP | 
| **3389** | RDP | Escritorio remoto de Windows (*Remote Desktop Protocol*) | TCP | 
| **10000** | \- | Panel de administración Webmin | TCP | 

### 🔍 Dosis de realidad / Curiosidades para la clase:

* **¿Por qué Telnet (23) está en desuso?** Porque enviaba el usuario y la contraseña en texto plano por la red. Cualquiera haciendo *sniffing* (escuchando la red) podía ver tus credenciales. Por eso nació **SSH (22)**, que cifra todo el tráfico.

* **El caso de DNS (53):** Usa **UDP** para consultas habituales por ser rápido y ligero, pero cambia a **TCP** cuando las respuestas son muy grandes o para transferencias de zona entre servidores DNS.

* **DHCP (67/68):** Utiliza dos puertos distintos para separar el tráfico entre el servidor (puerto 67) y el cliente que solicita la IP (puerto 68).

* **HTTPS (443):** Es el rey actual de la web. Si entras a un sitio web y ves el candadito en el navegador, estás comunicándote a través del puerto 443 cifrado con TLS/SSL.