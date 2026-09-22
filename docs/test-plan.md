# Plan de Pruebas

## Pruebas de conectividad y seguridad

| ID | Prueba | Resultado esperado | Estado |
|---|---|---|---|
| T01 | ADMIN -> web-dmz:80 | Permitido | Pendiente |
| T02 | ADMIN -> backend-01:22 | Permitido | Pendiente |
| T03 | web-dmz -> backend-01:8080 | Permitido | Pendiente |
| T04 | web-dmz -> db-01:5432 | Bloqueado | Pendiente |
| T05 | web-dmz -> ADMIN | Bloqueado | Pendiente |
| T06 | backend-01 -> db-01:5432 | Permitido | Pendiente |
| T07 | ADMIN -> aplicación | Funciona extremo a extremo | Pendiente |
| T08 | VMs -> Internet | Permitido | Pendiente |
| T09 | WAN -> redes internas | Bloqueado | Pendiente |

## Pruebas realizadas

### T-D02-01 — Conectividad WAN de OPNsense

Comando:

```text
ping 8.8.8.8