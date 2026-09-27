# Configuraciones de referencia

[Inicio](../README.md) · [Arquitectura](../documentacion/arquitectura.md) · [Diagnóstico](../documentacion/diagnostico.md)

Estos archivos son **ejemplos sanitizados**, no exportaciones de la configuración instalada en las VMs. No contienen credenciales.

| Archivo | Contenido | Estado |
|---|---|---|
| [nginx/sitio-laboratorio.conf](nginx/sitio-laboratorio.conf) | Entrada HTTP y proxy inverso hacia `backend-01:8080` | Bloque ilustrativo sin TLS; no validado contra la VM |
| [ssh/99-endurecimiento.conf](ssh/99-endurecimiento.conf) | Clave pública habilitada y autenticación por contraseña deshabilitada | Fragmento ilustrativo; falta verificar precedencia con `sshd -T` |
| [postgresql/README.md](postgresql/README.md) | Escucha restringida y autorización de `labd` / `labapp` | Controles comprobados parcialmente mediante conexión y escucha; falta exportación de `pg_hba.conf` |

## Criterio de seguridad

Las credenciales se mantienen fuera de Git. Si una configuración necesitara referenciar un secreto, utilizar un marcador explícito como:

```text
DB_PASSWORD=<secret>
```

No versionar claves privadas, `.env` reales, `.pgpass`, tokens ni exportaciones completas que puedan contener secretos.

## Validación antes de aplicar cambios

Los fragmentos de configuración deben verificarse en el host correspondiente antes de recargar un servicio:

El prefijo `99-` del ejemplo SSH no garantiza prioridad: OpenSSH suele utilizar el primer valor leído. Revisar los archivos incluidos y cualquier bloque `Match` al comprobar la configuración efectiva.

```bash
sudo nginx -t
sudo sshd -t
sudo sshd -T
```

Para PostgreSQL, comprobar la configuración efectiva y repetir las pruebas de conectividad del [plan de pruebas](../documentacion/plan-de-pruebas.md).
