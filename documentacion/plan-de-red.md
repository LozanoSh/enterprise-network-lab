# Plan de red

[Inicio](../README.md) · [Arquitectura](arquitectura.md) · [Firewall](reglas-de-firewall.md)

## Redes y gateways

| Zona | Red | Gateway de los hosts | Uso |
|---|---|---|---|
| WAN | NAT de VirtualBox, DHCP | Proporcionado por VirtualBox a fw-opnsense | Salida a Internet |
| DMZ | 10.10.10.0/24 | 10.10.10.1 | Entrada HTTP y reverse proxy |
| SERVERS | 10.10.20.0/24 | 10.10.20.1 | API y base de datos |
| ADMIN | 10.10.30.0/24 | 10.10.30.1 | Administración y pruebas |

## Inventario de hosts

| Hostname | Rol | Zona | IP | Gateway | Servicios / puertos relevantes |
|---|---|---|---|---|---|
| fw-opnsense | Router, firewall y DNS | WAN, DMZ, SERVERS, ADMIN | WAN por DHCP; 10.10.10.1, 10.10.20.1, 10.10.30.1 | WAN según DHCP | DNS 53; interfaz administrativa desde ADMIN, puerto exacto sin captura |
| web-dmz | Reverse proxy Nginx | DMZ | 10.10.10.10 | 10.10.10.1 | SSH TCP/22, HTTP TCP/80 |
| backend-01 | API Flask con psycopg | SERVERS | 10.10.20.10 | 10.10.20.1 | SSH TCP/22; HTTP TCP/8080 mientras la aplicación está levantada |
| db-01 | PostgreSQL 18 | SERVERS | 10.10.20.20 | 10.10.20.1 | SSH TCP/22; PostgreSQL TCP/5432 en 10.10.20.20 |
| admin-01 | Estación Ubuntu Desktop | ADMIN | DHCP; 10.10.30.176 observado | 10.10.30.1 | Cliente SSH, HTTP y DNS; no se documenta un servicio entrante |

La [captura de red de admin-01](../evidencias/dia-03/admin-01-configuracion-de-red.png) muestra DHCP y la ruta por `10.10.30.1`. El plan inicial proponía `10.10.30.10`, pero esa dirección no es la observada. Las capturas posteriores siguen mostrando `10.10.30.176`; no se asume una reserva permanente. Tampoco se infiere de estas capturas el mecanismo de asignación de todos los servidores.

Los puertos de la tabla son servicios relevantes para el ejercicio, no un inventario exhaustivo de sockets locales. Por ejemplo, la [captura de web-dmz](../evidencias/dia-06/web-dmz-servicios-en-escucha.png) también muestra resolución local, DHCP y sincronización horaria. No existe HTTPS de la aplicación en TCP/443.

## Interfaces de fw-opnsense

Asignación visible en la [consola del día 2](../evidencias/dia-02/fw-opnsense-interfaces.png):

| Interfaz OPNsense | Adaptador | Red de VirtualBox | Dirección |
|---|---|---|---|
| WAN | em0 | NAT | DHCP; 10.0.2.15/24 observado |
| LAN | em3 | ADMIN | 10.10.30.1/24 |
| OPT1 | em1 | DMZ | 10.10.10.1/24 |
| OPT2 | em2 | SERVERS | 10.10.20.1/24 |

La política interna documentada y sus pruebas se centran en IPv4. La presencia de direcciones o sockets IPv6 en capturas no demuestra filtrado equivalente en IPv6.

## Nombres internos

| Registro en Unbound | Dirección |
|---|---|
| web.lab.test | 10.10.10.10 |
| api.lab.test | 10.10.20.10 |
| db.lab.test | 10.10.20.20 |

La [validación con getent](../evidencias/dia-06/dns-interno-lab-test.png) confirma los tres resultados desde ADMIN. No se han documentado alias DNS para `admin-01` o `fw-opnsense`.
