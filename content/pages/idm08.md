<h1>Gestión de auditoria</h1>

<p>
<strong>Meta:</strong>
<br>Implementar soluciones de auditoria
</p>
<p>
<strong>Objetivos:</strong>
<br>- Implementar AIDE
<br>- Implementar AUDIT
<br>- Implementar SYSLOG
</p>
<p>
<strong>Secciones:</strong>
<br>- Gestión de eventos
</p>
<p>
<strong>Laboratorios:</strong>
<br><strong>- Instalar AIDE</strong>
<br><strong>- Configurar AUDIT</strong>
<br><strong>- Configurar syslog</strong>
<br><strong>- Configurar syslog como servidor</strong>

</p>

<strong>Requisitos:</strong>
<br><strong>- Servidor IDM</strong>
<br><strong>- Cliente IDM</strong><br>
</p>

<h3><strong>## Instalar AIDE</strong></h3>

AIDE ( Advanced Intrusion Detection Environment ) es una herramienta de detección de intrusos de código abierto, pequeña pero potente, que utiliza reglas predefinidas para verificar la integridad de archivos y directorios en sistemas operativos tipo Unix como Linux

Para configurar AIDE instalamos el rpm
```
[root@client ~]# yum install -y aide
```
En caso quisiéramos modificar la configuración editamos el archivo `/etc/aide.conf`

Para inicializar la base de datos 
```
[root@client ~]# aide --init
[root@client ~]# cp -p /var/lib/aide/aide.db.new.gz /var/lib/aide/aide.db.gz
```

Para verificar la auditorio
```
[root@client ~]# aide --check | grep etc
```

Intentamos generar cambios en el sistema
```
[root@client ~]# touch /etc/file01
[root@client ~]# touch /etc/file02
[root@client ~]# touch /etc/file03
```

Volvemos a verificar
```
[root@client ~]# aide --check | grep etc
f++++++++++++++++: /etc/file01
f++++++++++++++++: /etc/file02
f++++++++++++++++: /etc/file03

[root@client ~]# aide --check
Start timestamp: 2026-11-26 10:46:58 -0500 (AIDE 0.16)
AIDE found differences between database and filesystem!!

Summary:
  Total number of entries:      46350
  Added entries:                3
  Removed entries:              0
  Changed entries:              1

---------------------------------------------------
Added entries:
---------------------------------------------------

f++++++++++++++++: /etc/file01
f++++++++++++++++: /etc/file02
f++++++++++++++++: /etc/file03

---------------------------------------------------
Changed entries:
---------------------------------------------------

f   ...     C... : /var/log/audit/audit.log

---------------------------------------------------
Detailed information about changes:
---------------------------------------------------

File: /var/log/audit/audit.log
  SHA512   : WCyjpwt5WvhNQysqpUGmcQN9t+YjhKJ0 | NyNUA5EufvB/BpWLQfqHnj4Y/lU0bBuk
             Hto0NoMozwphUN5SMp+GInMZRZCQm3i6 | f5G+3m92prsimqjPhdKNBqsV4yeAwRw7
             S2TR+XEREzen08S4eDODNA==         | pbtPN4JYBZWwaen7S1Koxg==


---------------------------------------------------
The attributes of the (uncompressed) database(s):
---------------------------------------------------

/var/lib/aide/aide.db.gz
  MD5      : jxaam5jMnCMl7XqWMhdomQ==
  SHA1     : 2N2ZpTDSbdFy2tPVvn+JKmScG5w=
  RMD160   : ekJ5ODrSzcMvDT8DiEgA7k++bgk=
  TIGER    : yrwKriyMlv9aj5P2hb70N5JT/IDvZOby
  SHA256   : psjNkcl+YUjlJ8YtC+vWT6cOsSyQQbNp
             EvoHNjjoaTA=
  SHA512   : pqoywdZgu9K7NZjfYsMoXWx9KjrqckvQ
             0Zye88FaV2T4d9diTQqogXVBLonwCosK
             GZr/5AJMnZyf6DFI3wDZpw==


End timestamp: 2026-11-26 10:47:04 -0500 (run time: 0m 6s)
```

En caso no veamos cambios, podemos forzar una actualización con
```
[root@client ~]# aide --update
```

Es una buena practica actualizar la informacion de forma programada
```
[root@client ~]# vi /etc/cron.d/aide
00 00 * * * root /usr/sbin/aide --update
```

Vamos a crear un recurso de monitoreo para la carpeta /was el cual debera ser montoreado excluyendo la carpeta /was/log, para ello agregamos los recursos a aide.conf
```
[root@client ~]# mkdir /was
[root@client ~]# mkdir /was/log
[root@client ~]# vi /etc/aide.conf
/was   CONTENT_EX
!/was/log
```

Actualizamos la base de datos y generamos contenido en todos los recursos
```
[root@client ~]# aide --update
[root@client ~]# touch /was/file01
[root@client ~]# touch /was/file02
[root@client ~]# touch /was/file03
[root@client ~]# touch /was/file04
[root@client ~]# touch /was/file05
[root@client ~]# touch /was/file06
[root@client ~]# touch /was/log/log01
[root@client ~]# touch /was/log/log02
[root@client ~]# touch /was/log/log03
[root@client ~]# aide --check | grep was
d++++++++++++++++: /was
f++++++++++++++++: /was/file01
f++++++++++++++++: /was/file02
f++++++++++++++++: /was/file03
f++++++++++++++++: /was/file04
f++++++++++++++++: /was/file05
f++++++++++++++++: /was/file06
```


<h3><strong>## Configurar AUDIT</strong></h3>

El servicio de auditoria de linux (audit) permite vigilar el acceso a ciertos ficheros y directorios sobre los que deseamos tener control.

Verificamos las reglas de auditoria activadas
```
[root@client ~]# auditctl -l
No rules
```

Creamos un archivo para nuestra auditoria
```
[root@client ~]# touch /root/file01.txt
```

Creamos una regla de auditoria para el archivo
```
[root@client ~]# auditctl -w /root/file01.txt -p wa -k file01_changes
```
```
• -p <permisos>: Especifica qué acciones registrar. Se pueden combinar:
	• r: Lectura (read).
	• w: Escritura (write).
	• x: Ejecución (execute).
	• a: Cambio de atributos o metadatos (attribute change, como chmod o chown).
```


Volvemos a validar la lista de reglas
```
[root@client ~]# auditctl -l
-w /root/file01.txt -p wa -k file01_changes
```

Realizamos algunos cambios en el archivo y verificamos la auditoria con:
```
[root@client ~]# echo "hola Andy" > /root/file01.txt
[root@client ~]# ausearch -k file01_changes
----
time->Thu Nov 26 15:17:28 2026
type=PROCTITLE msg=audit(1795724248.713:163): proctitle=617564697463746C002D77002F726F6F742F66696C6530312E747874002D70007761002D6B0066696C6530315F6368616E676573
type=SYSCALL msg=audit(1795724248.713:163): arch=c000003e syscall=44 success=yes exit=1088 a0=4 a1=7ffff2d1e4d0 a2=440 a3=0 items=0 ppid=1922 pid=1950 auid=0 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=pts0 ses=3 comm="auditctl" exe="/usr/sbin/auditctl" subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 key=(null)
type=CONFIG_CHANGE msg=audit(1795724248.713:163): auid=0 ses=3 subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 op=add_rule key="file01_changes" list=4 res=1
----
time->Thu Nov 26 15:18:05 2026
type=PROCTITLE msg=audit(1795724285.021:164): proctitle="-bash"
type=PATH msg=audit(1795724285.021:164): item=1 name="/root/file01.txt" inode=17505168 dev=fd:00 mode=0100644 ouid=0 ogid=0 rdev=00:00 obj=unconfined_u:object_r:admin_home_t:s0 nametype=NORMAL cap_fp=0 cap_fi=0 cap_fe=0 cap_fver=0 cap_frootid=0
type=PATH msg=audit(1795724285.021:164): item=0 name="/root/" inode=16777351 dev=fd:00 mode=040550 ouid=0 ogid=0 rdev=00:00 obj=system_u:object_r:admin_home_t:s0 nametype=PARENT cap_fp=0 cap_fi=0 cap_fe=0 cap_fver=0 cap_frootid=0
type=CWD msg=audit(1795724285.021:164): cwd="/root"
type=SYSCALL msg=audit(1795724285.021:164): arch=c000003e syscall=257 success=yes exit=3 a0=ffffff9c a1=55d9258899a0 a2=241 a3=1b6 items=2 ppid=1921 pid=1922 auid=0 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=pts0 ses=3 comm="bash" exe="/usr/bin/bash" subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 key="file01_changes"
```

Podemos realizar el mismo procedimiento para carpetas
```
[root@client ~]# mkdir /folder01
[root@client ~]# auditctl -w /folder01 -p wa -k folder01_changes
[root@client ~]# auditctl -l
-w /root/file01.txt -p wa -k file01_changes
-w /folder01 -p wa -k folder01_changes
```

Generamos cambios en la carpeta y analizamos la salida
```
[root@client ~]#  touch /folder01/file01
[root@client ~]#  touch /folder01/file02
[root@client ~]#  touch /folder01/file03
[root@client ~]# ausearch -k folder01_changes
----
time->Thu Nov 26 15:22:34 2026
type=PROCTITLE msg=audit(1795724554.080:165): proctitle=617564697463746C002D77002F666F6C6465723031002D70007761002D6B00666F6C64657230315F6368616E676573
type=PATH msg=audit(1795724554.080:165): item=0 name="/folder01" inode=34502797 dev=fd:00 mode=040755 ouid=0 ogid=0 rdev=00:00 obj=unconfined_u:object_r:default_t:s0 nametype=NORMAL cap_fp=0 cap_fi=0 cap_fe=0 cap_fver=0 cap_frootid=0
type=CWD msg=audit(1795724554.080:165): cwd="/root"
type=SOCKADDR msg=audit(1795724554.080:165): saddr=100000000000000000000000
type=SYSCALL msg=audit(1795724554.080:165): arch=c000003e syscall=44 success=yes exit=1084 a0=4 a1=7ffec20d2910 a2=43c a3=0 items=1 ppid=1922 pid=1958 auid=0 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=pts0 ses=3 comm="auditctl" exe="/usr/sbin/auditctl" subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 key=(null)
type=CONFIG_CHANGE msg=audit(1795724554.080:165): auid=0 ses=3 subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 op=add_rule key="folder01_changes" list=4 res=1
----
time->Thu Nov 26 15:23:12 2026
type=PROCTITLE msg=audit(1795724592.999:166): proctitle=746F756368002F666F6C64657230312F66696C653031
type=PATH msg=audit(1795724592.999:166): item=1 name="/folder01/file01" inode=34502798 dev=fd:00 mode=0100644 ouid=0 ogid=0 rdev=00:00 obj=unconfined_u:object_r:default_t:s0 nametype=CREATE cap_fp=0 cap_fi=0 cap_fe=0 cap_fver=0 cap_frootid=0
type=PATH msg=audit(1795724592.999:166): item=0 name="/folder01/" inode=34502797 dev=fd:00 mode=040755 ouid=0 ogid=0 rdev=00:00 obj=unconfined_u:object_r:default_t:s0 nametype=PARENT cap_fp=0 cap_fi=0 cap_fe=0 cap_fver=0 cap_frootid=0
type=CWD msg=audit(1795724592.999:166): cwd="/root"
type=SYSCALL msg=audit(1795724592.999:166): arch=c000003e syscall=257 success=yes exit=3 a0=ffffff9c a1=7fff6201669e a2=941 a3=1b6 items=2 ppid=1922 pid=1962 auid=0 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=pts0 ses=3 comm="touch" exe="/usr/bin/touch" subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 key="folder01_changes"
----
time->Thu Nov 26 15:23:15 2026
type=PROCTITLE msg=audit(1795724595.567:167): proctitle=746F756368002F666F6C64657230312F66696C653032
type=PATH msg=audit(1795724595.567:167): item=1 name="/folder01/file02" inode=34502799 dev=fd:00 mode=0100644 ouid=0 ogid=0 rdev=00:00 obj=unconfined_u:object_r:default_t:s0 nametype=CREATE cap_fp=0 cap_fi=0 cap_fe=0 cap_fver=0 cap_frootid=0
type=PATH msg=audit(1795724595.567:167): item=0 name="/folder01/" inode=34502797 dev=fd:00 mode=040755 ouid=0 ogid=0 rdev=00:00 obj=unconfined_u:object_r:default_t:s0 nametype=PARENT cap_fp=0 cap_fi=0 cap_fe=0 cap_fver=0 cap_frootid=0
type=CWD msg=audit(1795724595.567:167): cwd="/root"
type=SYSCALL msg=audit(1795724595.567:167): arch=c000003e syscall=257 success=yes exit=3 a0=ffffff9c a1=7ffcc867269e a2=941 a3=1b6 items=2 ppid=1922 pid=1963 auid=0 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=pts0 ses=3 comm="touch" exe="/usr/bin/touch" subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 key="folder01_changes"
----
time->Thu Nov 26 15:23:16 2026
type=PROCTITLE msg=audit(1795724596.543:168): proctitle=746F756368002F666F6C64657230312F66696C653033
type=PATH msg=audit(1795724596.543:168): item=1 name="/folder01/file03" inode=34502800 dev=fd:00 mode=0100644 ouid=0 ogid=0 rdev=00:00 obj=unconfined_u:object_r:default_t:s0 nametype=CREATE cap_fp=0 cap_fi=0 cap_fe=0 cap_fver=0 cap_frootid=0
type=PATH msg=audit(1795724596.543:168): item=0 name="/folder01/" inode=34502797 dev=fd:00 mode=040755 ouid=0 ogid=0 rdev=00:00 obj=unconfined_u:object_r:default_t:s0 nametype=PARENT cap_fp=0 cap_fi=0 cap_fe=0 cap_fver=0 cap_frootid=0
type=CWD msg=audit(1795724596.543:168): cwd="/root"
type=SYSCALL msg=audit(1795724596.543:168): arch=c000003e syscall=257 success=yes exit=3 a0=ffffff9c a1=7ffc4c12869e a2=941 a3=1b6 items=2 ppid=1922 pid=1964 auid=0 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=pts0 ses=3 comm="touch" exe="/usr/bin/touch" subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 key="folder01_changes"
```

<h3><strong>## Configurar syslog</strong></h3>

Por default los logs de los sistemas RHEL se depuran en la 4ta semana, sin embargo si el administrador desea cambiar esta configuración puede modificar el archivo `/etc/logrotate.conf` y reiniciar el servicio
`# systemctl restart rsyslog`

A pesar que se configura la rotación de logs para la depuratorio, existe la necesidad de mantener persistencia de estos eventos en algún lugar, por eso podemos apoyarnos en el cliente syslog, para redireccionar los logs del sistema al equipo del instructor donde @ para UDP y @@ para TCP
<br>`# vi /etc/rsyslog.conf`
<br>`*.* @@192.168.0.120:514`
<br>`# systemctl restart rsyslog`


<h3><strong>## Configurar syslog como servidor</strong></h3>

Para implementar su propio servidor syslog use el mismo paquete rsyslog, tanto client como server usan el mismo servicio, para el ejercicio las funciones seran las siguientes:

Syslog Server = client de IDM
Syslog Client = server de IDM

Considerando que solo queremos recibir los siguientes LOGs de los posible clients
```
/var/log/dirsrv/slapd-MIGRACIONES09-GOB-PE/access
/var/log/secure
/var/log/messages
```

Ahora configuramos el syslogserver
```
[root@client ~]# vi /etc/rsyslog.d/remote-server.conf
# --- 1. Módulos de entrada (TCP recomendado para no perder logs de auditoría/LDAP) ---
module(load="imtcp" MaxSessions="500")
module(load="imudp")

# --- 2. Plantillas de destino por tipo de log ---
template(name="DirsrvAccessLog" type="string" string="/var/log/remote/%HOSTNAME%/dirsrv-access.log")
template(name="SecureLog"       type="string" string="/var/log/remote/%HOSTNAME%/secure.log")
template(name="MessagesLog"     type="string" string="/var/log/remote/%HOSTNAME%/messages.log")

# --- 3. Reglas de clasificación (Ruleset) ---
ruleset(name="RecepcionRemota") {

    # A) Logs de 389-DS / FreeIPA (identificados por el Tag enviado desde el cliente)
    if ($programname == "dirsrv-access" or $syslogtag contains "dirsrv-access") then {
        action(type="omfile" dynaFile="DirsrvAccessLog" dirCreateMode="0750" fileCreateMode="0640")
        stop
    }

    # B) Logs de seguridad (/var/log/secure -> facility authpriv o auth)
    if ($syslogfacility-text == "authpriv" or $syslogfacility-text == "auth") then {
        action(type="omfile" dynaFile="SecureLog" dirCreateMode="0750" fileCreateMode="0640")
        stop
    }

    # C) Resto de logs generales del sistema (/var/log/messages)
    action(type="omfile" dynaFile="MessagesLog" dirCreateMode="0750" fileCreateMode="0640")
    stop
}

# --- 4. Vincular puertos al Ruleset ---
input(type="imtcp" port="514" ruleset="RecepcionRemota")
input(type="imudp" port="514" ruleset="RecepcionRemota")
```

Creamos la carpeta de logs remotos para distinguirlos de los logs locales y habilitamos el servicio
```
[root@client ~]# mkdir -p /var/log/remote
[root@client ~]# chmod 750 /var/log/remote
[root@client ~]# systemctl restart rsyslog
[root@client ~]# rsyslogd -N1
rsyslogd: version 8.2310.0-4.el9, config validation run (level 1), master config /etc/rsyslog.conf
rsyslogd: End of config validation run. Bye.
[root@client ~]# firewall-cmd --permanent --add-port=514/tcp
[root@client ~]# firewall-cmd --permanent --add-port=514/udp
[root@client ~]# firewall-cmd --reload
```


Ahora configuramos el cliente syslog para que los envie al syslog server, reemplazar la IP del syslog server al final del archivo
```
[root@server ~]# vi /etc/rsyslog.d/remote-client.conf
# Cargar módulo para leer archivos de texto plano (dirsrv)
module(load="imfile" PollingInterval="2")

# Leer el log de acceso de 389-DS / FreeIPA y etiquetarlo como "dirsrv-access"
input(
    type="imfile"
    File="/var/log/dirsrv/slapd-MIGRACIONES09-GOB-PE/access"
    Tag="dirsrv-access"
    Severity="info"
    Facility="local0"
)

# Reenviar todo (dirsrv-access, authpriv/secure y messages) por TCP (@@) al servidor Syslog
*.* @@192.168.109.11:514
```

Reiniciarnos el servicio
```
[root@server ~]# systemctl restart rsyslog
[root@server ~]# rsyslogd -N1
rsyslogd: version 8.2510.0-2.el9, config validation run (level 1), master config /etc/rsyslog.conf
rsyslogd: End of config validation run. Bye.
```

Podemos generar eventos como reiniciar el idm, cambiar a un usuario y ejecutar sudos
```
[root@server ~]# ipactl restart
[root@server ~]# su - jaime.zamudio
[jaime.zamudio@server ~]$ touch test
[jaime.zamudio@server ~]$ sudo chown root.root test
[jaime.zamudio@server ~]$ exit
```
Verificamos los nuevos logs en el syslog server
```
[root@client ~]# ll /var/log/remote/
drwx--x---. 2 root root 69 Nov 26 18:05 server
[root@client ~]# ll /var/log/remote/server/
-rw-------. 1 root root 315345 Nov 26 18:08 dirsrv-access.log
-rw-------. 1 root root 104479 Nov 26 18:08 messages.log
-rw-------. 1 root root   3053 Nov 26 18:08 secure.log
```

<h3><strong>Laboratorio 1</strong></h3>
<br>Implemente el servicio de AIDE usando como procedimiento el ejemplo del instructor, creando una carpeta /oracle excluyendo el monitoreo de la subcarpeta /oracle/log, genere contenido en ambas carpetas para validar

<h3><strong>Laboratorio 2</strong></h3>
<br>Implemente el servicio de auditoria de un archivo y de una carpeta usando como procedimiento el ejemplo del instructor

<h3><strong>Laboratorio 3</strong></h3>
<br>Implemente el syslog server en su estación IDM client y el cliente syslog en su estación IDM Server, debe audir al menos los siguientes logs
<br>/var/log/dirsrv/slapd-<INSTANCIA DE IDM SERVER>/access
<br>/var/log/secure
<br>/var/log/messages


<p><br><a href="idm.md">volver</a></p>
