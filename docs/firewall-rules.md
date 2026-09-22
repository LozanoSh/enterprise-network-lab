# Reglas de Firewall

## Estado

Las siguientes reglas representan la política de seguridad planificada para el laboratorio.

La implementación y validación de estas reglas se realizará durante la etapa de configuración del firewall.

## Reglas planificadas

| Origen | Destino | Puerto / Servicio | Acción | Motivo |
|---|---|---|---|---|
| ADMIN | DMZ | 22, 80, 443 | PERMITIR | Administración y pruebas |
| ADMIN | SERVERS | 22, 8080 | PERMITIR | Administración de servidores |
| DMZ | backend-01 | 8080 | PERMITIR | Reverse proxy hacia la API |
| DMZ | SERVERS | Otros | BLOQUEAR | Evitar movimiento lateral |
| DMZ | ADMIN | Todos | BLOQUEAR | Proteger la red administrativa |
| backend-01 | db-01 | 5432 | PERMITIR | Acceso a PostgreSQL |
| SERVERS | ADMIN | Todos | BLOQUEAR | Evitar conexiones hacia administración |
| WAN | Redes internas | Todos | BLOQUEAR | No publicar servicios hacia Internet |

## Principio de seguridad

El laboratorio utiliza el principio de mínimo privilegio.

Solamente se habilitan las comunicaciones necesarias para el funcionamiento de los servicios. El resto del tráfico debe permanecer bloqueado.