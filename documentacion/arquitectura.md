# Arquitectura

[Inicio](../README.md) · [Diagramas](../diagramas/README.md) · [Red](plan-de-red.md) · [Firewall](reglas-de-firewall.md) · [Pruebas](plan-de-pruebas.md)

## Alcance y estado

El laboratorio contiene cinco VMs en VirtualBox y tres zonas internas: DMZ, SERVERS y ADMIN. La documentación corresponde al estado registrado hasta el día 6. No hay exportaciones completas de las VMs, del firewall ni del código de aplicación.

La [topología principal](../diagramas/topologia-principal.png) muestra las zonas y los servicios; el [flujo de `/health`](../diagramas/flujo-health.png) representa la petición y su respuesta.

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
| admin-01 → web-dmz:80 | ADMIN → fw-opnsense → DMZ | Permiso de acceso HTTP desde administración |
| web-dmz → backend-01:8080 | DMZ → fw-opnsense → SERVERS | Excepción concreta para el proxy inverso |
| backend-01 → db-01:5432 | Dentro de SERVERS | PostgreSQL: escucha, pg_hba.conf y autenticación |
| admin-01 → servidores:22 | ADMIN → fw-opnsense → zona del servidor | Regla de administración y autenticación SSH |
| DMZ / SERVERS → Internet | fw-opnsense → WAN → NAT de VirtualBox | Restricciones de salida por servicio |

Las respuestas de conexiones autorizadas forman parte de esos flujos. Permitir una sesión iniciada desde ADMIN no implica autorizar nuevas conexiones en sentido contrario.

## Fronteras de confianza

**WAN y laboratorio.** La WAN usa NAT de VirtualBox. La aplicación no tiene redirecciones de puertos ni reglas manuales de entrada en WAN. La DMZ es la zona de entrada HTTP interna del ejercicio; no significa que el sitio esté publicado en Internet.

**DMZ y SERVERS.** La separación reduce los destinos a los que puede acceder el servidor web. Su necesidad funcional es backend-01:8080. Una conexión directa desde web-dmz a db-01:5432 debe quedar bloqueada, aunque la DB esté funcionando.

**ADMIN y servicios.** ADMIN concentra la administración. DMZ y SERVERS no deben iniciar libremente tráfico hacia ella. Tener acceso administrativo es una capacidad distinta de atender HTTP o consultar datos.

**Aplicación y datos.** Es una separación entre procesos y hosts, pero no entre subredes. La autorización del usuario `labapp` se evalúa en PostgreSQL, no en `fw-opnsense`.

## Decisión actual: backend y DB en SERVERS

`10.10.20.10/24` y `10.10.20.20/24` se comunican directamente en la misma red virtual. El gateway `10.10.20.1` no está en ese recorrido. Por tanto, una regla de `fw-opnsense` para «backend → DB:5432» no controlaría esa conexión.

Los controles confirmados son:

- `listen_addresses = '10.10.20.20'`: limita la dirección de escucha, no los clientes que pueden alcanzarla.
- `pg_hba.conf`: limita el acceso remoto de `labapp` a `labd` desde `10.10.20.10/32` y exige SCRAM.
- Bloqueo entre DMZ y DB en el firewall de red.

Estos controles cubren funciones diferentes. `pg_hba.conf` autoriza conexiones al servicio; no sustituye un firewall de host. Los permisos de `labapp` también quedan disponibles para la aplicación. Los privilegios SQL efectivos no están documentados por completo.

Una subred independiente para `db-01` permitiría filtrar también el tramo backend → DB. No está implementada; requeriría cambiar direcciones, reglas, DNS y pruebas.

## Configuración frente a evidencia

El [plan de pruebas](plan-de-pruebas.md) enlaza resultados observados del recorrido HTTP, consultas SQL, resolución de nombres y rechazo de contraseña SSH. La política exacta de `pg_hba.conf`, la configuración efectiva completa de SSH y la ausencia de publicación WAN son configuraciones declaradas sin exportación independiente en este repositorio.

Los archivos de [configuraciones/](../configuraciones/README.md) son ejemplos sanitizados de los controles descritos. No representan un sistema de aprovisionamiento reproducible ni una copia literal de la configuración instalada.
