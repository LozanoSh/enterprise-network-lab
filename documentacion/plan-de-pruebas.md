# Plan de pruebas y resultados

[Inicio](../README.md) · [Firewall](reglas-de-firewall.md) · [Evidencias](../evidencias/README.md) · [Diagnóstico](diagnostico.md)

## Alcance

Esta matriz registra las sesiones de los días 2 a 6. Los comandos permiten repetir las pruebas con el laboratorio encendido, los servicios activos y acceso autorizado desde cada origen.

**Observado** significa visible en una captura. **Declarado** identifica una configuración sin exportación independiente en el repositorio. **Sin evidencia** señala comprobaciones no documentadas. Una prueba negativa sólo acredita el protocolo y destino ensayados.

## Matriz principal

| ID | Origen | Destino | Protocolo / puerto | Resultado esperado | Resultado observado | Evidencia |
|---|---|---|---|---|---|---|
| T01 | admin-01 | web-dmz, 10.10.10.10 | HTTP TCP/80 | Acceso permitido a `/health` | Respuesta JSON del backend con DB conectada | [Día 5](../evidencias/dia-05/flujo-completo-nginx-api-postgresql.png) |
| T02 | admin-01 | web-dmz, backend-01, db-01 | SSH TCP/22 | Sesión administrativa permitida en cada host | Sesiones y `SSH_CONNECTION` desde 10.10.30.176 | Día 3: [web](../evidencias/dia-03/admin-01-ssh-a-web-dmz.png), [backend](../evidencias/dia-03/admin-01-ssh-a-backend-01.png), [DB](../evidencias/dia-03/admin-01-ssh-a-db-01.png) |
| T03 | web-dmz | backend-01, 10.10.20.10 | HTTP TCP/8080 | `/health` accesible desde el proxy | JSON final con `database: connected`; hubo errores previos de credenciales | [Día 5](../evidencias/dia-05/web-dmz-prueba-de-api.png) |
| T04 | web-dmz | db-01, 10.10.20.20 | TCP/5432 | Acceso directo bloqueado | `nc` termina por timeout; coherente con la política, sin log de firewall asociado | [Día 6](../evidencias/dia-06/dmz-a-postgresql-bloqueado.png) |
| T05 | web-dmz | Interfaz ADMIN de fw-opnsense, 10.10.30.1 | ICMP | Sin respuesta | 4 enviados, 0 recibidos | [Día 4](../evidencias/dia-04/web-dmz-pruebas-bloqueadas.png) |
| T06 | backend-01 | db-01, base labd | PostgreSQL TCP/5432 | Autenticación de labapp y consulta permitidas | Tras un fallo de autenticación, `SELECT` devuelve el mensaje de healthcheck | [Día 5](../evidencias/dia-05/backend-01-conexion-a-postgresql.png) |
| T07 | admin-01 | Nginx → Flask → PostgreSQL | TCP/80 → 8080 → 5432 | Respuesta extremo a extremo con dato de DB | `database: connected`, `db_message: PostgreSQL funcionando`, `status: ok` | [Día 5](../evidencias/dia-05/flujo-completo-nginx-api-postgresql.png) |
| T08 | backend-01 | Interfaz ADMIN de fw-opnsense, 10.10.30.1 | ICMP | Sin respuesta | 4 enviados, 0 recibidos | [Día 4](../evidencias/dia-04/backend-01-pruebas-de-segmentacion.png) |
| T09 | admin-01 | web.lab.test, api.lab.test, db.lab.test | Resolución del sistema; DNS 53 | Direcciones 10.10.10.10, 10.10.20.10, 10.10.20.20 | `getent hosts` devuelve las tres IP esperadas | [Día 6](../evidencias/dia-06/dns-interno-lab-test.png) |
| T10 | admin-01 | Los tres servidores | SSH TCP/22, clave pública | Acceso por clave ED25519 permitido | Configuración declarada; las capturas muestran el cierre de sesiones, no la negociación de la clave | Día 6: [web](../evidencias/dia-06/web-dmz-ssh-solo-clave.png), [backend](../evidencias/dia-06/backend-01-ssh-solo-clave.png), [DB](../evidencias/dia-06/db-01-ssh-solo-clave.png) |
| T11 | admin-01 | Los tres servidores | SSH TCP/22, contraseña forzada | Autenticación por contraseña rechazada | `Permission denied (publickey)` en los tres | [web](../evidencias/dia-06/web-dmz-ssh-solo-clave.png), [backend](../evidencias/dia-06/backend-01-ssh-solo-clave.png), [DB](../evidencias/dia-06/db-01-ssh-solo-clave.png) |
| T12 | db-01, inspección local | Sockets de PostgreSQL | TCP/5432 | Escucha sólo en 10.10.20.20 | `ss` muestra 10.10.20.20:5432; clúster 18 online | [Día 5](../evidencias/dia-05/postgresql-en-ejecucion.png) y [Día 6](../evidencias/dia-06/db-01-servicios-en-escucha.png) |
| T13 | web-dmz | db-01, 10.10.20.20 | ICMP | Sin respuesta | 4 enviados, 0 recibidos | [Día 4](../evidencias/dia-04/web-dmz-pruebas-bloqueadas.png) |
| T14 | web-dmz | example.com y resolución de google.com | HTTP TCP/80, HTTPS TCP/443; resolución DNS | Servicios de salida disponibles | HTTP 200 y resolución de nombre externo | [Día 4](../evidencias/dia-04/web-dmz-pruebas-permitidas.png) |
| T15 | backend-01 | example.com y resolución de google.com | HTTP TCP/80, HTTPS TCP/443; resolución DNS | Servicios de salida disponibles | HTTP 200 y `getent hosts` correcto | [Día 4](../evidencias/dia-04/backend-01-prueba-dns.png) |
| T16 | fw-opnsense y VirtualBox | Configuración de entrada WAN | Reglas de entrada / NAT | Sin publicación de la aplicación | Configuración declarada; sin captura ni exportación de reglas | Sin captura específica |
| T17 | web-dmz, inspección local | Sockets de SSH y Nginx | TCP/22 y TCP/80 | Servicios escuchando | `ss -tulpn` muestra ambos; también servicios locales auxiliares | [Día 6](../evidencias/dia-06/web-dmz-servicios-en-escucha.png) |
| T18 | backend-01, inspección local | Servicios base del host | TCP/22 | SSH escuchando; Flask puede estar apagado fuera de las pruebas de aplicación | `ss -tulpn` muestra SSH; no aparece un servicio en 8080 en esa inspección | [Día 6](../evidencias/dia-06/backend-01-servicios-en-escucha.png) |
| T19 | db-01, inspección local | SSH y PostgreSQL | TCP/22 y TCP/5432 | SSH disponible y PostgreSQL ligado a su IP privada | `ss -tulpn` muestra 22 y 10.10.20.20:5432 | [Día 6](../evidencias/dia-06/db-01-servicios-en-escucha.png) |

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

El éxito de T06 y la escucha de T12 ayudan a descartar una DB apagada al interpretar T04. La atribución a una regla concreta requiere registros de `fw-opnsense` de la misma sesión.

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

Se esperan `yes`, `no`, `no`, respectivamente. Si existen bloques `Match`, comprobar también el contexto real de usuario y origen, como se explica en [diagnóstico](diagnostico.md).

### Servicios en escucha

En `db-01` para T12, y en cada host para completar el inventario:

```bash
sudo ss -ltnp 'sport = :5432'
sudo ss -tulpn
```

Para PostgreSQL no se espera `0.0.0.0:5432` ni `[::]:5432`. Los sockets locales de otros servicios no deben confundirse con servicios expuestos a la red.

## Validaciones no documentadas

| ID | Origen / destino | Protocolo o control | Resultado esperado | Estado y evidencia faltante |
|---|---|---|---|---|
| P01 | backend-01 / sockets locales | TCP/8080 y TCP/22 | Flask activo y SSH escuchando | T18 cubre SSH sin un servicio en 8080 durante esa inspección; T03 demuestra 8080 funcional cuando la API está levantada, pero falta una captura de `ss` con Flask activo |
| P02 | db-01 / configuración local | pg_hba.conf | labd/labapp desde 10.10.20.10/32 con SCRAM, sin permisos remotos más amplios | Configuración declarada; sin exportación sanitizada ni prueba de rechazo desde otro origen con acceso al servicio |
| P03 | admin-01 → servidores | SSH TCP/22 | Negociación por clave y opciones efectivas correctas | Ampliar T10 con salida de autenticación sanitizada y `sshd -T` por host |
| P04 | DMZ / SERVERS → host de ADMIN | TCP, puerto a registrar | Nueva conexión no autorizada bloqueada | Pendiente; T05/T08 sólo cubren ICMP al gateway |
| P05 | admin-01 → web.lab.test | HTTP TCP/80 | `/health` correcto usando DNS | DNS y flujo por IP probados por separado; falta captura conjunta |
