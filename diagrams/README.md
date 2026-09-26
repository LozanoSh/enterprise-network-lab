# Diagrama de topología pendiente

[Volver al proyecto](../README.md)

El README contiene un esquema ASCII temporal. Este directorio queda reservado para el diagrama final y su archivo fuente editable.

El diagrama debe representar:

- Las cinco VMs con sus nombres, zonas, subredes y gateways del [plan de red](../docs/network-plan.md).
- WAN por NAT de VirtualBox y ausencia de publicación de la aplicación.
- El recorrido HTTP/80 → HTTP/8080 → PostgreSQL/5432.
- OPNsense entre ADMIN, DMZ y SERVERS; el tramo backend → DB dentro de SERVERS, sin atravesarlo.
- El bloqueo directo DMZ → DB, la administración SSH y DNS en Unbound, con una leyenda que distinga funciones y flujos.

Al incorporarlo, enlazarlo desde la sección «Topología» del README y comprobar que siga siendo legible en GitHub. No representar TLS web, una VLAN de DB ni tecnologías de la lista de mejoras futuras como parte de la arquitectura actual.
