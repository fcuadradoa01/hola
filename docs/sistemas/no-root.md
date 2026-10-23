# ¿Por qué no debemos trabajar siempre como root?

En los sistemas Unix/Linux, la cuenta `root` posee control total sobre el sistema. Puede modificar cualquier archivo, instalar o eliminar software, cambiar configuraciones críticas e incluso dejar el sistema inoperativo con un solo comando.

Por este motivo, **no se recomienda trabajar de forma habitual como root**.

## 🔏 Principio de mínimo privilegio

Una de las normas fundamentales de la ciberseguridad indica que un usuario solo debe disponer de los permisos estrictamente necesarios para realizar su trabajo.

Trabajar habitualmente como usuario normal y emplear `sudo` únicamente cuando sea necesario es una aplicación práctica de este principio.

## 🙈 Errores con consecuencias graves

Un simple error tipográfico puede provocar daños importantes:

```bash
rm -rf /datos
```
o incluso...

```bash
rm -rf /
```

Ejecutados como root pueden eliminar información crítica del sistema.

## ✒️ Trazabilidad, logs y auditoría

Otro motivo para evitar trabajar directamente como `root` es la necesidad de mantener un registro claro de las acciones realizadas sobre el sistema.

Cuando varios administradores comparten la cuenta `root`:

- Resulta difícil saber quién realizó un cambio.
- La responsabilidad de las acciones queda difuminada.
- Se dificulta la investigación de incidentes y errores.

Por el contrario, cuando cada administrador utiliza su propia cuenta y emplea `sudo`, el sistema puede registrar qué usuario ejecutó cada acción privilegiada.

Por ejemplo, en los logs del sistema aparecerá información similar a:

```text
Oct 20 10:15:32 servidor sudo: fjcuadrado : TTY=pts/0 ; PWD=/home/fjcuadrado ; COMMAND=/usr/bin/systemctl restart apache2
```

💡 Por este motivo, en empresas y centros de datos es habitual deshabilitar el acceso directo de `root` por `SSH` y exigir el uso de cuentas personales con privilegios mediante sudo.

---

!!! tip "consejo"
    Si necesitas root todo el tiempo, probablemente estás haciendo algo mal. Si solo lo utilizas cuando realmente es necesario, estás administrando el sistema correctamente.

---

## 📜 Créditos y Licencia

* **Validación del Material:** Contenidos y prácticas validados por **Paco Cuadrado `fcuadradoa01@educantabria.es`**.
* **Licencia:** Licencia Creative Commons Atribución 4.0 Internacional ([CC BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.es)).
!!! note "CC BY 4.0"
    *Se permite compartir, copiar, redistribuir, adaptar y transformar este material para cualquier propósito (incluso comercial), siempre que se reconozca y mencione adecuadamente la autoría original.*
