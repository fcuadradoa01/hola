# Portainer
Portainer sobre Docker en Ubuntu es muy sencillo y te va a dar una interfaz web para gestionar contenedores, imágenes, volúmenes, redes, logs, stacks de Docker Compose, etc.
## 1. Crear el volumen da datos Portainer

```bash
docker volume create portainer_data
```` 
---

## 2. Lanzar el contenedor

```bash
 docker run -d \
  --name portainer \
  --restart=always \
  -p 8000:8000 \
  -p 9443:9443 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest
```
Comprueba que está funcionando:

```bash
docker ps
```

Deberías ver el contenedor ```portainer``` en estado ```Up```.


## 3. Abrir la interfaz web
Desde tu navegador: ```https://IP_DEL_SERVIDOR:9443 ```


⚠️ La primera vez aparecerá una advertencia del certificado SSL autofirmado. Es normal.


## 4. Crear usuario administrador
En el primer acceso:

- Usuario: el que quieras (por ejemplo ```admin```)
- Contraseña: una robusta
- Selecciona ```Get Started```
- Después elige: ```Local Environment```