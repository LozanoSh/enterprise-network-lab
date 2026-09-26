# Política de firewall

[Inicio](../README.md) · [Arquitectura](arquitectura.md) · [Pruebas](plan-de-pruebas.md)

## Cómo leer esta política

OPNsense filtra entre zonas con un criterio de mínimo privilegio. **PASS** permite el flujo indicado; **BLOCK** expresa el tráfico que debe denegarse, por regla explícita o por bloqueo por defecto. Estas tablas describen la política confirmada por el autor, no una exportación literal ni el orden de las reglas instaladas.

La columna de respaldo distingue un resultado visible de una configuración confirmada sin captura específica. Los IDs remiten al [plan de pruebas](plan-de-pruebas.md). Las pruebas cubren IPv4 y destinos concretos; no demuestran todas las combinaciones de origen, puerto y protocolo.

## ADMIN

| Origen | Destino | Puerto / protocolo | Acción | Justificación | Respaldo |
|---|---|---|---|---|---|
| admin-01 | web-dmz, backend-01, db-01 | TCP/22 | PASS | Administración por SSH | T02; autenticación en T10 y T11 |
| admin-01 | web-dmz | TCP/80 | PASS | Entrada a la aplicación | T01, T07 |
| ADMIN | fw-opnsense | Interfaz administrativa; puerto exacto sin documentar | PASS | Gestión del firewall | Confirmado; interfaz visible detrás de algunas terminales |
| admin-01 | fw-opnsense / Unbound | DNS, puerto 53 | PASS | Resolución interna | T09; consulta directa histórica en diagnóstico |

Se habilitaron además accesos específicos para pruebas. El plan inicial mencionaba ADMIN → backend:8080, pero no hay inventario completo de esas excepciones ni captura dedicada que justifique presentarlas como permisos permanentes. Falta registrar destinos y puertos efectivos; no se asume una regla «ADMIN → cualquiera».

## DMZ

| Origen | Destino | Puerto / protocolo | Acción | Justificación | Respaldo |
|---|---|---|---|---|---|
| web-dmz (10.10.10.10) | backend-01 (10.10.20.10) | TCP/8080 | PASS | Reverse proxy hacia Flask | T03, T07 |
| DMZ | SERVERS | Resto del tráfico no autorizado | BLOCK | Limitar movimiento lateral | Política confirmada; T04 y T13 cubren destinos puntuales |
| web-dmz | db-01 (10.10.20.20) | TCP/5432 | BLOCK | La DMZ no debe consultar datos directamente | T04: timeout |
| DMZ | ADMIN | Nuevas conexiones, cualquier protocolo | BLOCK | Proteger administración | T05 sólo comprueba ICMP a 10.10.30.1 |
| DMZ | Servicios web externos necesarios | TCP/80 y TCP/443 | PASS | Descargas y mantenimiento | T14 prueba example.com; lista exacta de destinos sin exportar |
| DMZ | Resolutores y servidores horarios autorizados | DNS 53 / NTP UDP/123 | PASS | Resolución y sincronización | DNS externo observado en T14; destinos y prueba NTP pendientes de captura |

El TCP/443 de esta tabla corresponde a **salida HTTPS a Internet**, no a un servicio HTTPS en web-dmz. La excepción TCP/8080 debe prevalecer sobre el bloqueo general hacia SERVERS; sin una exportación no se audita aquí el orden real.

## SERVERS

| Origen | Destino | Puerto / protocolo | Acción | Justificación | Respaldo |
|---|---|---|---|---|---|
| SERVERS | ADMIN | Nuevas conexiones no autorizadas | BLOCK | Evitar acceso iniciado desde servidores a administración | T08 sólo comprueba ICMP a 10.10.30.1 |
| SERVERS | Servicios web externos necesarios | TCP/80 y TCP/443 | PASS | Mantenimiento | T15 desde backend-01; no prueba todos los hosts |
| SERVERS | Resolutores y servicios necesarios de salida | DNS 53; demás excepciones sin detalle exportado | PASS limitado | Resolver nombres y mantener servicios | T15 muestra DNS; inventario de salida pendiente |

**backend-01 → db-01:5432 no es una regla efectiva de OPNsense.** Ambos hosts están en `10.10.20.0/24` y se comunican directamente. El acceso a `labd` con `labapp` se restringe en PostgreSQL mediante `pg_hba.conf`, origen `10.10.20.10/32` y SCRAM. La escucha está limitada a `10.10.20.20`. Ver los [controles del servicio](../configuraciones/postgresql/README.md).

## WAN

| Origen | Destino | Puerto / protocolo | Acción | Justificación | Respaldo |
|---|---|---|---|---|---|
| WAN | Servidores internos / aplicación | Nuevas conexiones entrantes | BLOCK por defecto; sin PASS manual ni port forwards | No publicar el laboratorio en Internet | Confirmado por el autor; T16 pendiente de evidencia de configuración |

La WAN recibe configuración por DHCP en el NAT de VirtualBox. La [conectividad de OPNsense a Internet](../evidencias/dia-02/fw-opnsense-prueba-ip-internet.png) prueba salida, no ausencia de exposición. Para cerrar T16 se deben revisar tanto OPNsense como los reenvíos del NAT de VirtualBox.

## Qué falta para auditar las reglas completas

- Capturas o exportación sanitizada con interfaces, orden, aliases y destinos de salida.
- Logs que correlacionen el timeout DMZ → DB con el bloqueo en OPNsense.
- Pruebas adicionales hacia un host de ADMIN y por TCP, además del ICMP al gateway.
- Revisión de IPv6 y de las excepciones administrativas temporales.

Un `Connection refused` indica rechazo de la conexión y no basta para acreditar una regla PASS. Un timeout es compatible con un bloqueo, pero también puede aparecer por falta de ruta o indisponibilidad del host. El diagnóstico debe combinar pruebas de cliente, escucha del servicio y logs.
