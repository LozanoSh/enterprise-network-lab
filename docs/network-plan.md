# Network Plan

## Objetivo

El laboratorio simula una pequeña infraestructura empresarial segmentada en tres zonas:

- DMZ
- SERVERS
- ADMIN

OPNsense funcionará como router y firewall entre todas las redes.

## Redes

| Zona | Red | Gateway | Uso |
|---|---|---|---|
| DMZ | 10.10.10.0/24 | 10.10.10.1 | Servicios expuestos |
| SERVERS | 10.10.20.0/24 | 10.10.20.1 | Backend y base de datos |
| ADMIN | 10.10.30.0/24 | 10.10.30.1 | Administración |

## Máquinas virtuales

| VM | Zona | IP prevista | Función |
|---|---|---|---|
| fw-opnsense | Todas | .1 | Router / Firewall |
| web-dmz | DMZ | 10.10.10.10 | Nginx / Reverse Proxy |
| backend-01 | SERVERS | 10.10.20.10 | Backend / API |
| db-01 | SERVERS | 10.10.20.20 | PostgreSQL |
| admin-01 | ADMIN | 10.10.30.10 | Administración |

## Puertos principales

- web-dmz: 80 / 443
- backend-01: 8080 / 22
- db-01: 5432 / 22
- admin-01: SSH, navegador y herramientas administrativas

## Flujo principal

admin-01
    ↓
web-dmz
    ↓ TCP/8080
backend-01
    ↓ TCP/5432
db-01