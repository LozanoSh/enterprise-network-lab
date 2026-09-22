# Arquitectura

## Descripción general

Este proyecto simula una pequeña infraestructura empresarial utilizando VirtualBox, OPNsense y máquinas virtuales Ubuntu.

OPNsense funciona como router y firewall central, conectando y aislando tres redes internas:

- DMZ
- SERVERS
- ADMIN

El objetivo es aplicar conceptos de segmentación de red, routing, firewall, administración remota y seguridad.

## Topología

Internet
   |
VirtualBox NAT
   |
OPNsense
   |
   +-- DMZ ------ web-dmz
   |
   +-- SERVERS -- backend-01
   |              db-01
   |
   +-- ADMIN ---- admin-01

## Redes

| Zona | Red | Gateway | Propósito |
|---|---|---|---|
| DMZ | 10.10.10.0/24 | 10.10.10.1 | Servicios expuestos |
| SERVERS | 10.10.20.0/24 | 10.10.20.1 | Backend y base de datos |
| ADMIN | 10.10.30.0/24 | 10.10.30.1 | Administración |

## Interfaces de OPNsense

| Interfaz | Red de VirtualBox | Dirección |
|---|---|---|
| WAN / em0 | NAT | DHCP |
| LAN / em3 | ADMIN | 10.10.30.1/24 |
| OPT1 / em1 | DMZ | 10.10.10.1/24 |
| OPT2 / em2 | SERVERS | 10.10.20.1/24 |

## Estado actual

- OPNsense instalado correctamente.
- WAN configurada mediante DHCP.
- Acceso a Internet verificado.
- Resolución DNS verificada.
- Redes internas configuradas.
- DHCP habilitado temporalmente en las redes internas.
- La red ADMIN puede comunicarse con OPNsense.