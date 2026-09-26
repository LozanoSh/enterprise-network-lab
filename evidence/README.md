# Índice de evidencias

[Inicio](../README.md) · [Matriz de pruebas](../docs/test-plan.md) · [Diagnóstico](../docs/troubleshooting.md)

Las capturas registran etapas del montaje, no el estado en tiempo real de las VMs. Los nombres de los directorios identifican jornadas del proyecto; varias sesiones se subieron juntas en el mismo commit. Las pruebas con contraseña de los primeros días preceden al hardening del día 6.

## Día 2: OPNsense y conectividad inicial

| Captura | Qué permite comprobar |
|---|---|
| [Interfaces](day-02/opnsense-interfaces.png) | WAN por DHCP y gateways de ADMIN, DMZ y SERVERS |
| [Salida por IP](day-02/opnsense-internet-ip-test.png) | Ping desde OPNsense a 8.8.8.8 con respuesta |
| [Resolución externa](day-02/opnsense-dns-test.png) | Resolución de google.com y respuesta ICMP desde OPNsense |

## Día 3: estación de administración

| Captura | Qué permite comprobar |
|---|---|
| [Configuración de red](day-03/admin-network-config.png) | admin-01 con 10.10.30.176 por DHCP y gateway 10.10.30.1 |
| [Conectividad](day-03/admin-connectivity-tests.png) | Acceso al gateway, salida por IP y resolución externa |
| SSH a [web-dmz](day-03/admin-to-web-dmz-ssh.png), [backend-01](day-03/admin-to-backend-ssh.png) y [db-01](day-03/admin-to-db-ssh.png) | Sesiones con origen 10.10.30.176 y destino TCP/22, mostrado por SSH_CONNECTION |

## Día 4: segmentación

| Captura | Qué permite comprobar |
|---|---|
| SSH a [web-dmz](day-04/admin-to-web-dmz-ssh.png), [backend-01](day-04/admin-to-backend-ssh.png) y [db-01](day-04/admin-to-db-ssh.png) | Administración antes de deshabilitar contraseña |
| [Salidas desde DMZ](day-04/dmz-allowed-tests.png) | HTTP/HTTPS externo y resolución; también un rechazo en backend:8080 y un timeout a backend:5432 |
| [Bloqueos desde DMZ](day-04/dmz-blocked-tests.png) | ICMP sin respuesta a 10.10.30.1 y db-01, con salida HTTP/HTTPS funcionando |
| [DNS y salida desde SERVERS](day-04/servers-dns-test.png) | HTTP/HTTPS y getent hosts desde backend-01 |
| [Aislamiento de SERVERS](day-04/servers-segmentation-tests.png) | ICMP desde backend-01 a 10.10.30.1 sin respuesta |

El intento TCP/5432 de `dmz-allowed-tests.png` apunta a **10.10.20.10 (backend)**, no a db-01. La prueba directa contra PostgreSQL se encuentra en el día 6.

## Día 5: aplicación y persistencia

| Captura | Qué permite comprobar |
|---|---|
| [Nginx activo](day-05/nginx-running.png) | Servicio iniciado |
| [Nginx en TCP/80](day-05/nginx-port-80.png) | Estado y socket en escucha |
| [PostgreSQL activo](day-05/postgresql-running.png) | Clúster 18 online y escucha en 10.10.20.20:5432 |
| [Consulta desde backend](day-05/backend-to-postgresql.png) | Acceso con labapp a labd y lectura de healthcheck tras un fallo de autenticación |
| [DMZ → API](day-05/dmz-to-backend-health.png) | `/health` en 8080; errores por credencial ausente y respuesta final correcta |
| [Flujo completo](day-05/full-flow-nginx-health.png) | Petición desde ADMIN a web-dmz:80 con resultado de la DB |

## Día 6: DNS y hardening

| Captura | Qué permite comprobar |
|---|---|
| [DNS actual](day-06/internal-dns-lab-test.png) | Los tres nombres actuales resueltos por getent hosts desde ADMIN |
| [Diagnóstico DNS inicial](day-06/internal-dns-resolution.png) | Consulta directa al resolver durante la etapa previa; contexto en troubleshooting |
| SSH en [web-dmz](day-06/web-dmz-ssh-key-only.png), [backend-01](day-06/backend-ssh-key-only.png) y [db-01](day-06/db-ssh-key-only.png) | Cierre de sesiones y rechazo explícito al forzar contraseña |
| [Servicios de web-dmz](day-06/web-dmz-listening-services.png) | SSH y Nginx en escucha, además de servicios auxiliares locales |
| [DMZ → PostgreSQL bloqueado](day-06/dmz-to-postgresql-blocked.png) | Timeout desde web-dmz hacia 10.10.20.20:5432 |

## Correcciones de organización

- Se intercambiaron los nombres `admin-to-web-dmz-ssh.png` y `admin-to-backend-ssh.png` del día 4 para que coincidan con la IP y el host que muestran. Sus bytes no cambiaron.
- En el día 6, los archivos llamados `backend-listening-services.png` y `db-listening-services.png` eran copias byte a byte de `web-dmz-listening-services.png`. Se retiraron los dos duplicados y se conservó la captura original correcta. No existía evidencia independiente de sockets de esos otros dos hosts en esa jornada.
- Se conservaron las 28 imágenes únicas sin recortes, ediciones ni redacciones. Los prompts muestran nombres de usuario, IP privadas y rutas de sistema propias del laboratorio; no se identificaron contraseñas visibles, tokens ni claves privadas durante la inspección visual.

Las capturas incluyen intentos fallidos y mensajes del sistema que permiten reconstruir el diagnóstico. No se interpretan un prompt de contraseña como una contraseña publicada, ni una huella pública de certificado como una clave privada.
