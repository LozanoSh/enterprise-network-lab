# Configuraciones de referencia

[Inicio](../README.md) · [Arquitectura](../docs/architecture.md) · [Diagnóstico](../docs/troubleshooting.md)

Estos archivos son **ejemplos sanitizados reconstruidos a partir del comportamiento y los controles confirmados del laboratorio**. No son exportaciones literales de las VMs ni se han aplicado durante esta revisión del repositorio. Su finalidad es explicar las partes relevantes de la configuración.

| Archivo | Contenido | Alcance |
|---|---|---|
| [nginx/default.conf](nginx/default.conf) | Entrada HTTP y proxy a backend-01:8080 | Bloque mínimo ilustrativo, sin TLS |
| [ssh/99-hardening.conf](ssh/99-hardening.conf) | Autenticación por clave y desactivación de contraseña | Fragmento; requiere revisar precedencia y configuración efectiva |
| [postgresql/README.md](postgresql/README.md) | Escucha y autorización de labd/labapp | Parámetros y explicación, sin credenciales |

El prefijo `99-` del ejemplo SSH no garantiza prioridad: OpenSSH suele utilizar el primer valor leído. Antes de incorporar cualquier fragmento a una VM, revisar sus archivos existentes y comprobar `nginx -t` o `sshd -t`/`sshd -T`, según corresponda. No hay una validación de estos ejemplos contra los servicios instalados.

Las credenciales se proporcionan fuera de Git. Si se documenta una variable secreta, usar un marcador explícito como `DB_PASSWORD=<secret>`; no representa un valor utilizable ni confirma el nombre de variable usado por el código actual. No copiar claves privadas, archivos `.env` reales ni exportaciones completas de OPNsense sin revisión.
