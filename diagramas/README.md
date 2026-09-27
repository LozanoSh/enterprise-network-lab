# Diagramas del laboratorio

[Volver al proyecto](../README.md) · [Arquitectura](../documentacion/arquitectura.md)

| Diagrama | Contenido |
|---|---|
| [Topología principal](topologia-principal.png) | Cinco VMs, zonas ADMIN/DMZ/SERVERS, `fw-opnsense`, servicios y comunicaciones principales. |
| [Flujo de `/health`](flujo-health.png) | Ilustración original del recorrido de la petición y su respuesta. `fw-opnsense` representa el paso por el firewall; no recibe ni genera peticiones HTTP. |

Los PNG rotulan la VM con el nombre del producto, «OPNsense»; su hostname en el laboratorio es `fw-opnsense`. El [README principal](../README.md) muestra ambos diagramas directamente. Las [capturas de pruebas](../evidencias/README.md) respaldan los resultados observados. La IP `10.10.30.176` de `admin-01` fue asignada por DHCP durante las pruebas.
