# Diagnóstico y aprendizajes

[Inicio](../README.md) · [Pruebas](plan-de-pruebas.md) · [Configuraciones](../configuraciones/README.md)

## Método de diagnóstico

Separar capas evita cambiar reglas a ciegas: comprobar dirección y ruta, resolución de nombres, socket en escucha, conexión TCP, autenticación y respuesta de aplicación. Registrar el origen de cada comando; una prueba ejecutada por SSH dentro de web-dmz tiene como origen web-dmz, aunque la terminal esté abierta en admin-01.

## DNS: consulta directa y resolución del sistema

Se probó inicialmente `lab.local`. La [captura histórica](../evidencias/dia-06/resolucion-dns-inicial.png) muestra un `NXDOMAIN` y, después, consultas directas a `10.10.30.1:53` que devuelven las IP correctas para `web.lab.local`, `api.lab.local` y `db.lab.local`.

Una consulta directa correcta no asegura que las aplicaciones resuelvan igual. En Linux, `.local` puede dirigirse a mDNS; consultar un servidor DNS explícitamente y usar el mecanismo habitual del sistema son caminos distintos. Se adoptó `lab.test` y se verificaron los tres nombres con `getent hosts`. La captura inicial no basta para atribuir aquel `NXDOMAIN` a mDNS. Ver la documentación de [resolución de systemd](https://www.freedesktop.org/software/systemd/man/247/org.freedesktop.resolve1.html) y el [registro de dominios de uso especial de IANA](https://www.iana.org/assignments/special-use-domain-names).

Comprobaciones de diagnóstico desde ADMIN con los nombres actuales:

```bash
nslookup web.lab.test 10.10.30.1
getent hosts web.lab.test
resolvectl status
```

La [evidencia final](../evidencias/dia-06/dns-interno-lab-test.png) corresponde a `getent hosts` para web, API y DB. Los nombres anteriores sólo se conservan como registro de esta etapa.

### Unbound, Dnsmasq y el puerto consultado

El resolver final del laboratorio es Unbound. Al diagnosticar, hay que identificar qué servicio responde en el puerto 53 y dónde se cargaron los registros. Tener una entrada en otro servicio no prueba que el cliente la consulte.

OPNsense documenta una disposición con Unbound en 53 y Dnsmasq en 53053, con reenvío entre ambos. El puerto 53053 no es el destino DNS habitual del cliente. Esto explica la distinción relevante durante el montaje; no hay una captura que permita afirmar que ese reenvío esté implementado aquí. Referencias: [Unbound](https://docs.opnsense.org/manual/unbound.html) y [ejemplos de Dnsmasq](https://docs.opnsense.org/manual/dnsmasq.html).

## PostgreSQL: escucha, autorización y credencial

La [consulta desde backend-01](../evidencias/dia-05/backend-01-conexion-a-postgresql.png) registra primero un fallo de autenticación y luego un `SELECT` correcto. La [prueba de Flask](../evidencias/dia-05/web-dmz-prueba-de-api.png) muestra `fe_sendauth: no password supplied` antes de la respuesta correcta. Son fallos de credencial; no prueban un bloqueo de firewall ni un error concreto de `pg_hba.conf`.

| Síntoma | Qué revisar |
|---|---|
| Timeout TCP | Origen, ruta, disponibilidad del host y logs del firewall |
| `Connection refused` | Servicio en escucha o rechazo activo; no asumir una regla PASS |
| `no pg_hba.conf entry` | Coincidencia de base, usuario, origen y tipo de conexión |
| `password authentication failed` | Credencial del usuario solicitado |
| `no password supplied` | Que el proceso de la aplicación reciba la credencial, sin imprimirla |

En db-01, una inspección de sólo lectura permite separar la escucha de la autorización:

```bash
sudo ss -ltnp 'sport = :5432'
sudo -u postgres psql -c 'SHOW listen_addresses;'
sudo -u postgres psql -c 'SHOW hba_file;'
```

`listen_addresses` controla dónde escucha PostgreSQL; `pg_hba.conf` selecciona el acceso por cliente, base y usuario. Se utiliza la primera regla coincidente, por lo que una entrada restrictiva no basta si hay permisos más amplios que también habilitan otros orígenes. Ver [autorización de clientes en PostgreSQL 18](https://www.postgresql.org/docs/18/auth-pg-hba-conf.html) y los [fragmentos sanitizados](../configuraciones/postgresql/README.md).

## Nginx: comprobar cada tramo

Primero probar Flask desde web-dmz y después la entrada por Nginx desde admin-01. Así se distingue una falla de la API o su DB de una falla del proxy. La evidencia del día 5 conserva errores de conexión a DB y la recuperación posterior; no contiene el texto de un error específico de sintaxis Nginx.

Para revisar una configuración Nginx, comprobar sintaxis, estado, escucha y logs antes de atribuir el problema a la red:

```bash
sudo nginx -t
systemctl status nginx --no-pager
sudo ss -ltnp 'sport = :80'
sudo journalctl -u nginx --no-pager -n 30
```

En el ejemplo versionado, `proxy_pass` apunta a `http://10.10.20.10:8080` y conserva `/health`. Errores de puerto, destino o sintaxis deben diagnosticarse por separado. La semántica de la URI está documentada en [proxy_pass](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass).

## SSH: verificar el resultado efectivo

Las capturas del día 6 muestran que forzar contraseña devuelve `Permission denied (publickey)` en los tres servidores. Para explicar ese resultado hay que revisar tanto `/etc/ssh/sshd_config` como sus archivos incluidos.

En general, OpenSSH conserva el primer valor leído de una opción. Un archivo llamado `99-endurecimiento.conf` no garantiza que su valor prevalezca sobre uno anterior. También pueden influir bloques `Match`. Ver [sshd_config](https://man.openbsd.org/sshd_config).

```bash
sudo sshd -t
sudo sshd -T | grep -E '^(pubkeyauthentication|passwordauthentication|kbdinteractiveauthentication) '
```

Si existen condiciones por usuario u origen, añadir a `sshd -T` la opción `-C user=USUARIO_ADMIN,addr=IP_ADMIN,host=NOMBRE_CLIENTE`, sustituyendo esos marcadores por datos reales. Esta comprobación está descrita en [sshd](https://man.openbsd.org/sshd#T). Mantener una sesión válida mientras se comprueba otra evita perder el acceso en futuras tareas de hardening.

## Leer las pruebas negativas con precisión

En el día 4, web-dmz obtuvo `Connection refused` al intentar llegar a backend:8080; esa captura no demuestra que Flask estuviera activo. La respuesta correcta de `/health` en el día 5 sí verifica ese camino funcional.

En el día 6, web-dmz no consiguió conectar a DB:5432. El timeout es coherente con el bloqueo confirmado. La escucha de PostgreSQL y la consulta exitosa desde backend aportan contexto, pero falta un registro de OPNsense de la misma sesión para vincular el intento a una regla concreta.
