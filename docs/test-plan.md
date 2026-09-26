# Plan de pruebas y resultados

[Inicio](../README.md) · [Firewall](firewall-rules.md) · [Evidencias](../evidence/README.md) · [Diagnóstico](troubleshooting.md)

## Alcance

Esta matriz registra las sesiones de los días 2 a 6. Los comandos son una guía para repetirlas, no nuevas ejecuciones realizadas durante la revisión del repositorio. Se necesita el laboratorio encendido, servicios activos y acceso autorizado desde el origen indicado.

**Observado** significa visible en una captura. **Confirmado por el autor** identifica datos del montaje sin evidencia independiente suficiente. **Pendiente** señala comprobaciones o capturas por completar. Una prueba negativa satisfactoria demuestra que ese intento no se permitió, dentro del alcance del protocolo y destino ensayados.

## Matriz principal

| ID | Origen | Destino | Protocolo / puerto | Resultado esperado | Resultado observado | Evidencia |
|---|---|---|---|---|---|---|
| T01 | admin-01 | web-dmz, 10.10.10.10 | HTTP TCP/80 | Acceso permitido a `/health` | Respuesta JSON del backend con DB conectada | [Día 5](../evidence/day-05/full-flow-nginx-health.png) |
| T02 | admin-01 | web-dmz, backend-01, db-01 | SSH TCP/22 | Sesión administrativa permitida en cada host | Sesiones y `SSH_CONNECTION` desde 10.10.30.176 | Día 3: [web](../evidence/day-03/admin-to-web-dmz-ssh.png), [backend](../evidence/day-03/admin-to-backend-ssh.png), [DB](../evidence/day-03/admin-to-db-ssh.png) |
| T03 | web-dmz | backend-01, 10.10.20.10 | HTTP TCP/8080 | `/health` accesible desde el proxy | JSON final con `database: connected`; hubo errores previos de credenciales | [Día 5](../evidence/day-05/dmz-to-backend-health.png) |
| T04 | web-dmz | db-01, 10.10.20.20 | TCP/5432 | Acceso directo bloqueado | `nc` termina por timeout; coherente con la política, sin log de firewall asociado | [Día 6](../evidence/day-06/dmz-to-postgresql-blocked.png) |
| T05 | web-dmz | Interfaz ADMIN de fw-opnsense, 10.10.30.1 | ICMP | Sin respuesta | 4 enviados, 0 recibidos | [Día 4](../evidence/day-04/dmz-blocked-tests.png) |
| T06 | backend-01 | db-01, base labd | PostgreSQL TCP/5432 | Autenticación de labapp y consulta permitidas | Tras un fallo de autenticación, `SELECT` devuelve el mensaje de healthcheck | [Día 5](../evidence/day-05/backend-to-postgresql.png) |
| T07 | admin-01 | Nginx → Flask → PostgreSQL | TCP/80 → 8080 → 5432 | Respuesta extremo a extremo con dato de DB | `database: connected`, `db_message: PostgreSQL funcionando`, `status: ok` | [Día 5](../evidence/day-05/full-flow-nginx-health.png) |
| T08 | backend-01 | Interfaz ADMIN de fw-opnsense, 10.10.30.1 | ICMP | Sin respuesta | 4 enviados, 0 recibidos | [Día 4](../evidence/day-04/servers-segmentation-tests.png) |
| T09 | admin-01 | web.lab.test, api.lab.test, db.lab.test | Resolución del sistema; DNS 53 | Direcciones 10.10.10.10, 10.10.20.10, 10.10.20.20 | `getent hosts` devuelve las tres IP esperadas | [Día 6](../evidence/day-06/internal-dns-lab-test.png) |
| T10 | admin-01 | Los tres servidores | SSH TCP/22, clave pública | Acceso por clave ED25519 permitido | Confirmado por el autor; capturas muestran el cierre de sesiones, no la negociación detallada de la clave | Día 6: [web](../evidence/day-06/web-dmz-ssh-key-only.png), [backend](../evidence/day-06/backend-ssh-key-only.png), [DB](../evidence/day-06/db-ssh-key-only.png) |
| T11 | admin-01 | Los tres servidores | SSH TCP/22, contraseña forzada | Autenticación por contraseña rechazada | `Permission denied (publickey)` en los tres | [web](../evidence/day-06/web-dmz-ssh-key-only.png), [backend](../evidence/day-06/backend-ssh-key-only.png), [DB](../evidence/day-06/db-ssh-key-only.png) |
| T12 | db-01, inspección local | Sockets de PostgreSQL | TCP/5432 | Escucha sólo en 10.10.20.20 | `ss` muestra 10.10.20.20:5432; clúster 18 online | [Día 5](../evidence/day-05/postgresql-running.png) |
| T13 | web-dmz | db-01, 10.10.20.20 | ICMP | Sin respuesta | 4 enviados, 0 recibidos | [Día 4](../evidence/day-04/dmz-blocked-tests.png) |
| T14 | web-dmz | example.com y resolución de google.com | HTTP TCP/80, HTTPS TCP/443; resolución DNS | Servicios de salida disponibles | HTTP 200 y resolución de nombre externo | [Día 4](../evidence/day-04/dmz-allowed-tests.png) |
| T15 | backend-01 | example.com y resolución de google.com | HTTP TCP/80, HTTPS TCP/443; resolución DNS | Servicios de salida disponibles | HTTP 200 y `getent hosts` correcto | [Día 4](../evidence/day-04/servers-dns-test.png) |
| T16 | Revisión de OPNsense y VirtualBox | Configuración de entrada WAN | Reglas de entrada / NAT | Sin publicación de la aplicación | Confirmado por el autor; pendiente de captura o exportación sanitizada | Sin captura específica |
| T17 | web-dmz, inspección local | Sockets de SSH y Nginx | TCP/22 y TCP/80 | Servicios escuchando | `ss -tulpn` muestra ambos; también servicios locales auxiliares | [Día 6](../evidence/day-06/web-dmz-listening-services.png) |

T01 y T07 utilizan una misma captura para dos criterios: disponibilidad HTTP y recorrido completo. T05 y T08 cubren el gateway de ADMIN, no todas las máquinas de esa zona. T09 prueba resolución mediante el sistema; por sí solo `getent` no identifica el servidor que respondió.

## Comandos de repetición

### HTTP y consulta a datos

Desde `admin-01`, para T01/T07 por IP y, como extensión, por el nombre actual:

```bash
curl --max-time 5 http://10.10.10.10/health
curl --max-time 5 http://web.lab.test/health
```

La captura existente corresponde a la primera variante. Además de recibir JSON, comprobar los valores `status`, `database` y `db_message`.

Desde `web-dmz`, para T03 y T04:

```bash
curl --max-time 5 http://10.10.20.10:8080/health
nc -vz -w 3 10.10.20.20 5432
```

Desde `backend-01`, para T06, introducir la contraseña sólo en el prompt interactivo:

```bash
psql -h 10.10.20.20 -U labapp -d labd -W -c 'SELECT * FROM healthcheck;'
```

El éxito de T06 y la escucha de T12 ayudan a descartar una DB apagada al interpretar T04. Para atribuir el bloqueo a una regla concreta, falta correlacionarlo con logs de OPNsense en la misma sesión.

### Aislamiento y DNS

Desde `web-dmz`, T05/T13; desde `backend-01`, sólo el primer comando para T08:

```bash
ping -c 4 10.10.30.1
ping -c 4 10.10.20.20
```

Desde `admin-01`, T09:

```bash
getent hosts web.lab.test
getent hosts api.lab.test
getent hosts db.lab.test
```

### SSH: probar ambos métodos

Desde `admin-01`, sustituir `USUARIO_ADMIN` por la cuenta del laboratorio y repetir con las IP `10.10.10.10`, `10.10.20.10` y `10.10.20.20`:

```bash
# T10: ofrecer únicamente la clave indicada; no publicar el archivo privado.
ssh -i ~/.ssh/id_ed25519 -o IdentitiesOnly=yes -o PreferredAuthentications=publickey -o PasswordAuthentication=no USUARIO_ADMIN@10.10.10.10

# T11: deshabilitar clave pública y forzar contraseña.
ssh -o PubkeyAuthentication=no -o PreferredAuthentications=password USUARIO_ADMIN@10.10.10.10
```

T10 debe abrir una sesión; T11 debe rechazarla con `Permission denied (publickey)`. Una eventual solicitud de passphrase de la clave privada es distinta de la contraseña remota. No se desactiva la verificación de identidad del servidor.

En cada servidor, revisar sintaxis y configuración efectiva:

```bash
sudo sshd -t
sudo sshd -T | grep -E '^(pubkeyauthentication|passwordauthentication|kbdinteractiveauthentication) '
```

Se esperan `yes`, `no`, `no`, respectivamente. Si existen bloques `Match`, comprobar también el contexto real de usuario y origen, como se explica en [diagnóstico](troubleshooting.md).

### Servicios en escucha

En `db-01` para T12, y en cada host para completar el inventario:

```bash
sudo ss -ltnp 'sport = :5432'
sudo ss -tulpn
```

Para PostgreSQL no se espera `0.0.0.0:5432` ni `[::]:5432`. Los sockets locales de otros servicios no deben confundirse con servicios expuestos a la red.

## Evidencia pendiente de ampliar

| ID | Origen / destino | Protocolo o control | Resultado esperado | Estado y evidencia faltante |
|---|---|---|---|---|
| P01 | backend-01 / sockets locales | TCP/8080 y TCP/22 | Flask activo y SSH escuchando | Falta captura independiente de `ss`; T03 demuestra respuesta HTTP |
| P02 | db-01 / sockets locales | TCP/5432 y TCP/22 | DB restringida y SSH disponible | T12 cubre DB en día 5; falta inventario completo del día 6 |
| P03 | db-01 / configuración local | pg_hba.conf | labd/labapp desde 10.10.20.10/32 con SCRAM, sin permisos remotos más amplios | Configuración confirmada; falta exportación sanitizada y prueba de rechazo de otro origen que alcance el servicio |
| P04 | admin-01 → servidores | SSH TCP/22 | Negociación por clave y opciones efectivas correctas | Ampliar T10 con salida de autenticación sanitizada y `sshd -T` por host |
| P05 | DMZ / SERVERS → host de ADMIN | TCP, puerto a registrar | Nueva conexión no autorizada bloqueada | Pendiente; T05/T08 sólo cubren ICMP al gateway |
| P06 | admin-01 → web.lab.test | HTTP TCP/80 | `/health` correcto usando DNS | DNS y flujo por IP probados por separado; falta captura conjunta |

Los archivos originalmente llamados `backend-listening-services.png` y `db-listening-services.png` eran copias exactas de la captura de web-dmz. Se conserva la imagen correcta; no se utilizan esos duplicados para acreditar P01/P02.
