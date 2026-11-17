---
title: "Laboratorio: Acceso ssh mediante claves asimétricas"
tags:
  - autenticación
  - ssh
  - rsa
  - clave publica
  - clave privada
level: basico
---
# 🛡️ Laboratorio: Acceso Seguro a Servidores Linux mediante SSH y Claves Asimétricas

## 🎯 Objetivos de Aprendizaje
Al finalizar esta práctica, serás capaz de:
1. Comprender la diferencia fundamental entre la autenticación tradicional por contraseña y la autenticación basada en criptografía asimétrica (clave pública / clave privada).
2. Generar un par de claves criptográficas modernas en tu equipo cliente.
3. Configurar un servidor **Ubuntu Server** para permitir el acceso seguro mediante claves sin depender de contraseñas.
4. Identificar y gestionar los principales archivos de configuración y seguridad de SSH, tanto en el cliente como en el servidor.

---

## 🧠 Parte 1: Conceptos Fundamentales (Antes de tocar la terminal)

### Contraseñas vs. Claves Asimétricas
Cuando te conectas a un servidor usando una **contraseña**, estás enviando un secreto compartido a través de la red (aunque el túnel SSH esté cifrado). Si un atacante logra interceptar esa contraseña mediante ingeniería social o fuerza bruta, podrá entrar al servidor desde cualquier lugar del mundo.

La **autenticación asimétrica** soluciona este problema dividiendo la credencial en dos archivos matemáticamente vinculados:

| Característica | Clave Privada (`id_ed25519`) | Clave Pública (`id_ed25519.pub`) |
| :--- | :--- | :--- |
| **Analogía** | La llave física de tu casa. | El candado o cerradura que instalas en la puerta. |
| **¿Dónde vive?** | **Únicamente** en tu equipo cliente (PC/portátil). | En todos los servidores a los que quieras conectarte. |
| **Visibilidad** | **Secreta.** Nunca se comparte ni se envía por red. | **Pública.** Puede copiarse o enviarse sin compromiso. |
| **Función** | Demuestra tu identidad resolviendo un reto matemático. | Verifica que posees la clave privada correspondiente. |

> 💡 **¿Por qué es más seguro?** Porque tu clave privada **nunca viaja por la red** durante el inicio de sesión. El servidor utiliza tu clave pública para enviarte un mensaje cifrado; solo tu equipo, al poseer la clave privada, puede descifrarlo y demostrar quién eres.

---

## 🛠️ Parte 2: Desarrollo Práctico

### Escenario del Laboratorio
* **Equipo Cliente:** Tu ordenador personal (Windows con PowerShell/Git Bash, macOS o Linux).
* **Equipo Servidor:** Una máquina virtual o física con **Ubuntu Server** accesible por red.

---

### Paso 1: Generar el par de claves en el equipo cliente
En tu equipo local, abre una terminal y ejecuta el siguiente comando para crear tus claves utilizando el algoritmo **Ed25519** (más rápido y seguro que el antiguo RSA):

```bash
ssh-keygen -t ed25519 -C "alumno@laboratorio"
```

1. **Ruta de guardado:** Pulsa `Enter` para aceptar la ubicación por defecto (`~/.ssh/id_ed25519`).
2. **Frase de contraseña (*Passphrase*):** Puedes escribir una contraseña extra para proteger el archivo de tu clave privada en tu disco duro, o pulsar `Enter` dos veces para dejarla sin frase de paso.

**Comprobación:** Verifica que se han creado los dos archivos en tu carpeta oculta `.ssh`:
```bash
ls -la ~/.ssh/
```
*(Deberías ver `id_ed25519` e `id_ed25519.pub`)*.

---

### Paso 2: Llevar tu clave pública al servidor Ubuntu
Ahora debemos instalar el "candado" (tu clave pública) en el servidor Ubuntu. Para esta primera vez, todavía necesitarás usar tu contraseña de usuario del servidor.

#### Opción A: Si estás en Linux o macOS (Método automático)
Utiliza la herramienta dedicada para copiar la clave:
```bash
ssh-copy-id usuario@ip_del_servidor
```

#### Opción B: Si estás en Windows o quieres hacerlo manualmente
1. Muestra el contenido de tu clave pública en tu terminal local y cópialo:
   ```bash
   cat ~/.ssh/id_ed25519.pub
   ```
2. Conéctate de forma tradicional al servidor Ubuntu:
   ```bash
   ssh usuario@ip_del_servidor
   ```
3. Una vez dentro de Ubuntu, crea la carpeta `.ssh` (si no existe), ajusta sus permisos y pega tu clave dentro del archivo `authorized_keys`:
   ```bash
   mkdir -p ~/.ssh
   chmod 700 ~/.ssh
   echo "PEGA_AQUI_EL_CONTENIDO_DE_TU_CLAVE_PUBLICA" >> ~/.ssh/authorized_keys
   chmod 600 ~/.ssh/authorized_keys
   ```

> ⚠️ **Nota crítica sobre permisos:** Linux es muy estricto con la seguridad de SSH. Si la carpeta `~/.ssh` no tiene permisos `700` o el archivo `authorized_keys` no tiene permisos `600`, el servidor ignorará tus claves por precaución.

---

### Paso 3: Probar la conexión sin contraseña
Cierra la sesión en el servidor (escribiendo `exit`) y vuelve a conectarte desde tu equipo cliente:

```bash
ssh usuario@ip_del_servidor
```

Si todo está bien configurado, entrarás directamente a la terminal de Ubuntu **sin que te solicite la contraseña del usuario del servidor**.

---

### Paso 4: Simplificar la conexión (Archivo `config` del cliente)
Recordar direcciones IP y usuarios es tedioso. Podemos crear un "alias" en nuestro equipo cliente.

En tu **equipo local**, crea o edita el archivo `~/.ssh/config`:
```bash
nano ~/.ssh/config
```

Añade la siguiente estructura adaptada a tus datos:
```text
Host ubuntu-lab
    HostName 192.168.1.50
    User alumno
    IdentityFile ~/.ssh/id_ed25519
```

Guarda los cambios (`Ctrl + O`, `Enter`) y sal (`Ctrl + X`). A partir de ahora, para conectarte solo tendrás que escribir:
```bash
ssh ubuntu-lab
```

---

### Paso 5: Endurecer el servidor (Desactivar contraseñas)
Una vez comprobado que el acceso por clave funciona, el paso final en entornos reales es prohibir el inicio de sesión mediante contraseña para evitar ataques de fuerza bruta.

1. Dentro del **servidor Ubuntu**, edita el archivo de configuración del servicio SSH:
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```
2. Busca y asegúrate de que estas directivas estén activas (sin el símbolo `#` delante) y con los siguientes valores:
   ```text
   PubkeyAuthentication yes
   PasswordAuthentication no
   ```
3. Guarda el archivo y reinicia el servicio para aplicar los cambios:
   ```bash
   sudo systemctl restart ssh
   ```

---

## 📂 Parte 3: Mapa de Archivos Clave en SSH

Para dominar la administración de sistemas, debes saber qué función cumple cada archivo y en qué máquina se encuentra:

### 🖥️ En tu Equipo Cliente (`~/.ssh/`)
* **`id_ed25519`**: Tu clave privada. Debe tener permisos restrictivos (`600`). Nunca debe salir de tu equipo.
* **`id_ed25519.pub`**: Tu clave pública. El texto que contiene es lo que se copia a los servidores.
* **`known_hosts`**: Base de datos con las "huellas dactilares" (*fingerprints*) de los servidores a los que te has conectado. Sirve para evitar ataques *Man-in-the-Middle* (suplantación del servidor).
* **`config`**: Archivo opcional donde defines alias, puertos personalizados o qué clave usar para cada servidor.

### 🖧 En el Servidor Ubuntu
* **`~/.ssh/authorized_keys`**: Lista de claves públicas (una por línea) que tienen permiso para iniciar sesión en esa cuenta de usuario específica.
* **`/etc/ssh/sshd_config`**: El archivo principal de configuración del **demonio (servidor) SSH**. Controla el puerto de escucha, si se permite el acceso como `root`, si se aceptan contraseñas, etc.
* **`/var/log/auth.log`**: Archivo de auditoría donde Ubuntu registra todos los intentos de conexión (exitosos y fallidos). Es el primer lugar donde debes mirar si algo falla.

---

## 📝 Cuestionario de Repaso para Entregar
Responde brevemente a las siguientes cuestiones para validar tu práctica:
1. Si formateas tu ordenador cliente y pierdes el archivo `id_ed25519`, pero conservas `id_ed25519.pub`, ¿podrás volver a conectarte al servidor? ¿Por qué?
2. ¿Qué ocurre en el cliente si reinstalamos el sistema operativo del servidor Ubuntu desde cero (manteniendo la misma IP) e intentamos conectarnos por SSH? ¿Qué archivo local interviene y por qué lanza una advertencia?
3. Ejecuta el comando `sudo tail -n 10 /var/log/auth.log` en tu servidor Ubuntu tras iniciar sesión y pega la línea donde se evidencia que entraste usando `publickey`.