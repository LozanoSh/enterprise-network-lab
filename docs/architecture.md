# Arquitectura

[Inicio](../README.md) · [Red](network-plan.md) · [Firewall](firewall-rules.md) · [Pruebas](test-plan.md)

## Alcance y estado

El laboratorio contiene cinco VMs en VirtualBox y tres zonas internas: DMZ, SERVERS y ADMIN. Esta descripción corresponde al cierre del día 6 y combina las capturas conservadas con la configuración confirmada por el autor. No hay una exportación completa de las VMs, del firewall ni del código de aplicación.

## Responsabilidades por capa

| Capa | Componente | Responsabilidad | Dependencia principal |
|---|---|---|---|
| Red | fw-opnsense | Enrutar y filtrar tráfico entre zonas; salida WAN mediante NAT de VirtualBox | Interfaces virtuales de las tres redes |
| Nombres | Unbound en fw-opnsense | Resolver web.lab.test, api.lab.test y db.lab.test | Registros internos y acceso DNS desde clientes |
| Administración | admin-01 | Gestionar servidores por SSH y validar servicios | Acceso permitido desde ADMIN y clave ED25519 |
| Entrada HTTP | web-dmz / Nginx | Recibir TCP/80 y reenviar peticiones al backend | backend-01:8080 |
| Aplicación | backend-01 / Flask | Consultar la DB mediante psycopg y responder `/health` | db-01:5432, base labd, usuario labapp |
| Datos | db-01 / PostgreSQL 18 | Persistir y devolver el mensaje de healthcheck | Escucha restringida y autenticación SCRAM |

Nginx conoce el destino HTTP de la API; no necesita una conexión a PostgreSQL. La aplicación es la capa que utiliza el usuario de base de datos.

## Recorrido del tráfico

| Tramo | Camino | Control relevante |
|---|---|---|
| admin-01 → web-dmz:80 | ADMIN → OPNsense → DMZ | Permiso de acceso HTTP desde administración |
| web-dmz → backend-01:8080 | DMZ → OPNsense → SERVERS | Excepción concreta para el reverse proxy |
| backend-01 → db-01:5432 | Dentro de SERVERS | PostgreSQL: escucha, pg_hba.conf y autenticación |
| admin-01 → servidores:22 | ADMIN → OPNsense → zona del servidor | Regla de administración y autenticación SSH |
| DMZ / SERVERS → Internet | OPNsense → WAN → NAT de VirtualBox | Restricciones de salida por servicio |

Las respuestas de conexiones autorizadas forman parte de esos flujos. Permitir una sesión iniciada desde ADMIN no implica autorizar nuevas conexiones en sentido contrario.

## Fronteras de confianza

**WAN y laboratorio.** La WAN usa NAT de VirtualBox. La aplicación no tiene port forwards ni reglas manuales de entrada en WAN. La DMZ es la zona de entrada HTTP interna del ejercicio; no significa que el sitio esté publicado en Internet.

**DMZ y SERVERS.** La separación reduce los destinos a los que puede acceder el servidor web. Su necesidad funcional es backend-01:8080. Una conexión directa desde web-dmz a db-01:5432 debe quedar bloqueada, aunque la DB esté funcionando.

**ADMIN y servicios.** ADMIN concentra la administración. DMZ y SERVERS no deben iniciar libremente tráfico hacia ella. Tener acceso administrativo es una capacidad distinta de atender HTTP o consultar datos.

**Aplicación y datos.** Es una separación entre procesos y hosts, pero todavía no entre subredes. La autorización del usuario `labapp` se evalúa en PostgreSQL, no en OPNsense.

## Decisión actual: backend y DB en SERVERS

`10.10.20.10/24` y `10.10.20.20/24` se comunican directamente en la misma red virtual. El gateway `10.10.20.1` no está en ese recorrido. Por tanto, una supuesta regla OPNsense «backend → DB:5432» no controla la conexión actual.

Los controles confirmados son:

- `listen_addresses = '10.10.20.20'`: limita la dirección de escucha, no los clientes que pueden alcanzarla.
- `pg_hba.conf`: limita el acceso remoto de `labapp` a `labd` desde `10.10.20.10/32` y exige SCRAM.
- Bloqueo entre DMZ y DB en el firewall de red.

Estos controles cubren funciones diferentes. `pg_hba.conf` autoriza conexiones al servicio; no sustituye un firewall de host. Tampoco evita que una aplicación comprometida use las credenciales y permisos que legítimamente posee. No hay una revisión de privilegios SQL completa versionada.

Separar la DB en una nueva VLAN/subred permitiría aplicar filtrado de red también a ese tramo. Es una evolución futura: cambiaría el direccionamiento y requeriría actualizar reglas, DNS y pruebas. No forma parte del estado actual.

## Configuración frente a evidencia

El [plan de pruebas](test-plan.md) enlaza resultados observados. Las capturas prueban el recorrido HTTP, consultas SQL, resolución de nombres y rechazo de contraseña SSH. La restricción exacta de `pg_hba.conf`, la configuración efectiva completa de SSH y la ausencia de publicación WAN están confirmadas por el autor, pero todavía no tienen una captura o exportación independiente completa en este repositorio.

Los archivos de [configs/](../configs/README.md) son ejemplos sanitizados de los controles descritos. No representan un sistema de aprovisionamiento reproducible ni una copia literal de la configuración instalada.
