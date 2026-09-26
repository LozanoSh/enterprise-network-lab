# Índice de evidencias

[Inicio](../README.md) · [Matriz de pruebas](../documentacion/plan-de-pruebas.md) · [Diagnóstico](../documentacion/diagnostico.md)

Las capturas registran etapas del montaje, no el estado en tiempo real de las VMs. Los nombres de los directorios identifican jornadas del proyecto; varias sesiones se subieron juntas en el mismo commit. Las pruebas con contraseña de los primeros días preceden al hardening del día 6.

## Día 2: OPNsense y conectividad inicial

| Captura | Qué permite comprobar |
|---|---|
| [Interfaces](dia-02/fw-opnsense-interfaces.png) | WAN por DHCP y gateways de ADMIN, DMZ y SERVERS |
| [Salida por IP](dia-02/fw-opnsense-prueba-ip-internet.png) | Ping desde OPNsense a 8.8.8.8 con respuesta |
| [Resolución externa](dia-02/fw-opnsense-prueba-dns.png) | Resolución de google.com y respuesta ICMP desde OPNsense |

## Día 3: estación de administración

| Captura | Qué permite comprobar |
|---|---|
| [Configuración de red](dia-03/admin-01-configuracion-de-red.png) | admin-01 con 10.10.30.176 por DHCP y gateway 10.10.30.1 |
| [Conectividad](dia-03/admin-01-pruebas-de-conectividad.png) | Acceso al gateway, salida por IP y resolución externa |
| SSH a [web-dmz](dia-03/admin-01-ssh-a-web-dmz.png), [backend-01](dia-03/admin-01-ssh-a-backend-01.png) y [db-01](dia-03/admin-01-ssh-a-db-01.png) | Sesiones con origen 10.10.30.176 y destino TCP/22, mostrado por SSH_CONNECTION |

## Día 4: segmentación

| Captura | Qué permite comprobar |
|---|---|
| SSH a [web-dmz](dia-04/admin-01-ssh-a-web-dmz.png), [backend-01](dia-04/admin-01-ssh-a-backend-01.png) y [db-01](dia-04/admin-01-ssh-a-db-01.png) | Administración antes de deshabilitar contraseña |
| [Salidas desde DMZ](dia-04/web-dmz-pruebas-permitidas.png) | HTTP/HTTPS externo y resolución; también un rechazo en backend:8080 y un timeout a backend:5432 |
| [Bloqueos desde DMZ](dia-04/web-dmz-pruebas-bloqueadas.png) | ICMP sin respuesta a 10.10.30.1 y db-01, con salida HTTP/HTTPS funcionando |
| [DNS y salida desde SERVERS](dia-04/backend-01-prueba-dns.png) | HTTP/HTTPS y getent hosts desde backend-01 |
| [Aislamiento de SERVERS](dia-04/backend-01-pruebas-de-segmentacion.png) | ICMP desde backend-01 a 10.10.30.1 sin respuesta |

El intento TCP/5432 de `web-dmz-pruebas-permitidas.png` apunta a **10.10.20.10 (backend)**, no a db-01. La prueba directa contra PostgreSQL se encuentra en el día 6.

## Día 5: aplicación y persistencia

| Captura | Qué permite comprobar |
|---|---|
| [Nginx activo](dia-05/nginx-en-ejecucion.png) | Servicio iniciado |
| [Nginx en TCP/80](dia-05/nginx-puerto-80.png) | Estado y socket en escucha |
| [PostgreSQL activo](dia-05/postgresql-en-ejecucion.png) | Clúster 18 online y escucha en 10.10.20.20:5432 |
| [Consulta desde backend](dia-05/backend-01-conexion-a-postgresql.png) | Acceso con labapp a labd y lectura de healthcheck tras un fallo de autenticación |
| [DMZ → API](dia-05/web-dmz-prueba-de-api.png) | `/health` en 8080; errores por credencial ausente y respuesta final correcta |
| [Flujo completo](dia-05/flujo-completo-nginx-api-postgresql.png) | Petición desde ADMIN a web-dmz:80 con resultado de la DB |

## Día 6: DNS y hardening

| Captura | Qué permite comprobar |
|---|---|
| [DNS actual](dia-06/dns-interno-lab-test.png) | Los tres nombres actuales resueltos por getent hosts desde ADMIN |
| [Diagnóstico DNS inicial](dia-06/resolucion-dns-inicial.png) | Consulta directa al resolver durante la etapa previa; contexto en diagnóstico |
| SSH en [web-dmz](dia-06/web-dmz-ssh-solo-clave.png), [backend-01](dia-06/backend-01-ssh-solo-clave.png) y [db-01](dia-06/db-01-ssh-solo-clave.png) | Cierre de sesiones y rechazo explícito al forzar contraseña |
| [Servicios de web-dmz](dia-06/web-dmz-servicios-en-escucha.png) | SSH y Nginx en escucha, además de servicios auxiliares locales |
| [DMZ → PostgreSQL bloqueado](dia-06/dmz-a-postgresql-bloqueado.png) | Timeout desde web-dmz hacia 10.10.20.20:5432 |

## Correcciones de organización

- Se corrigió la identificación de las capturas SSH de web-dmz y backend-01 del día 4 para que coincida con la IP y el host que muestran. Sus bytes no cambiaron.
- En el día 6, las capturas de servicios atribuidas a backend-01 y db-01 eran copias byte a byte de `web-dmz-servicios-en-escucha.png`. Se retiraron los dos duplicados y se conservó la captura original correcta. No existía evidencia independiente de sockets de esos otros dos hosts en esa jornada.
- Se conservaron las 28 imágenes únicas sin recortes, ediciones ni redacciones. Los prompts muestran nombres de usuario, IP privadas y rutas de sistema propias del laboratorio; no se identificaron contraseñas visibles, tokens ni claves privadas durante la inspección visual.

Las capturas incluyen intentos fallidos y mensajes del sistema que permiten reconstruir el diagnóstico. No se interpretan un prompt de contraseña como una contraseña publicada, ni una huella pública de certificado como una clave privada.
