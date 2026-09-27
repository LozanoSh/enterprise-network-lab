# PostgreSQL: controles de acceso

[Configuraciones](../README.md) · [Arquitectura](../../documentacion/arquitectura.md) · [Pruebas](../../documentacion/plan-de-pruebas.md)

PostgreSQL 18 se ejecuta en `db-01` (`10.10.20.20`). La aplicación usa la base `labd`, el usuario `labapp` y la tabla `healthcheck`. Los fragmentos siguientes describen la configuración declarada; no son archivos exportados de la VM.

## Dirección de escucha

Fragmento de `postgresql.conf`:

```ini
listen_addresses = '10.10.20.20'
port = 5432
```

La [captura del día 5](../../evidencias/dia-05/postgresql-en-ejecucion.png) muestra el clúster 18 online y un socket TCP en `10.10.20.20:5432`. Limitar la escucha no selecciona los clientes autorizados. Los cambios de `listen_addresses` requieren reinicio del servicio; ver la [referencia de conexiones de PostgreSQL 18](https://www.postgresql.org/docs/18/runtime-config-connection.html).

## Autorización de la aplicación

Entrada remota representativa en `pg_hba.conf`:

```text
# TIPO  BASE  USUARIO  ORIGEN          AUTENTICACIÓN
host    labd  labapp   10.10.20.10/32  scram-sha-256
```

Esta entrada limita esa autorización a la base, el usuario y el origen indicados. Debe revisarse el archivo completo para descartar reglas más amplias: PostgreSQL utiliza la primera coincidencia y rechaza conexiones sin coincidencia. No reemplazar con este fragmento las reglas locales necesarias para administración. Ver [pg_hba.conf](https://www.postgresql.org/docs/18/auth-pg-hba-conf.html).

SCRAM autentica al usuario; esta línea `host` no exige TLS. Aunque la sesión `psql` capturada negoció TLS, no hay evidencia suficiente para afirmar que todas las conexiones de Flask lo exijan o validen certificados.

## Validación disponible

- [backend-01 → DB](../../evidencias/dia-05/backend-01-conexion-a-postgresql.png): consulta de `healthcheck` con resultado `PostgreSQL funcionando`.
- [web-dmz → DB](../../evidencias/dia-06/dmz-a-postgresql-bloqueado.png): intento TCP que termina por timeout entre zonas.
- Sin evidencia: exportación sanitizada de las reglas efectivas y prueba de rechazo desde otro origen que alcance PostgreSQL. El bloqueo de la DMZ por `fw-opnsense` no prueba por sí solo `pg_hba.conf`.

No se incluyen sentencias de creación de usuarios con contraseña ni credenciales. Tampoco se presupone un conjunto de privilegios SQL que no haya sido documentado.
