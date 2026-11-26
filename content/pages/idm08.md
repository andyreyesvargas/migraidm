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
<br>`# auditctl -l`

Creamos un archivo para nuestra auditoria
<br>`# touch /root/file01.txt`

Creamos una regla de auditoria para el archivo
<br>`# auditctl -w /root/file01.txt -p wa -k file01_changes`

Volvemos a validar la lista de reglas
<br>`# auditctl -l`

Realizamos algunos cambios en el archivo y verificamos la auditoria con:
<br>`# ausearch -k file01_changes`

Podemos realizar el mismo procedimiento para carpetas
<br>`# mkdir /folder01`
<br>`# auditctl -w /folder01 -p wa -k folder01_changes`
<br>`# ausearch -k folder01_changes`
<br>`# touch /folder01/file01`
<br>`# ausearch -k folder01_changes`


<h3><strong>Laboratorio</strong></h3>
<br>Implemente el servicio de auditoria de un archivo y de una carpeta usando como procedimiento el ejemplo del instructor


<h3><strong>## Configurar syslog</strong></h3>

Por default los logs de los sistemas RHEL se depuran en la 4ta semana, sin embargo si el administrador desea cambiar esta configuración puede modificar el archivo `/etc/logrotate.conf` y reiniciar el servicio
`# systemctl restart rsyslog`

A pesar que se configura la rotación de logs para la depuratorio, existe la necesidad de mantener persistencia de estos eventos en algún lugar, por eso podemos apoyarnos en el cliente syslog, para redireccionar los logs del sistema al equipo del instructor donde @ para UDP y @@ para TCP
<br>`# vi /etc/rsyslog.conf`
<br>`*.* @@192.168.0.120:514`
<br>`# systemctl restart rsyslog`


<h3><strong>## Configurar syslog como servidor</strong></h3>

Para implementar su propio servidor syslog use el mismo paquete rsyslog y el mismo archivo de configuración en el servidor IDM
<br>`# vi /etc/rsyslog.conf`

Descomente las linea de UDP
<br>`module(load="imudp") # needs to be done just once`
<br>`input(type="imudp" port="514")`

Abra los puertos de firewall
<br>`# firewall-cmd --permanent --add-por=514/udp`
<br>`# firewall-cmd --reload`

Reinicie el servicio
<br>`# systemctl restart rsyslog`

Ahora en el client IDM configure el servicio rsyslog para que apunte a su propio servidor syslog
<br>`# vi /etc/rsyslog.conf`
<br>`*.* @192.168.0.171:514`
<br>`# systemctl restart rsyslog`
<br>`# logger prueba`

Validemos el archivo `/var/log/messages` del servidor IDM y debe incluir también los eventos del cliente IDM

<h3><strong>Laboratorio</strong></h3>
<br>Implemente el syslog server en su estación IDM Server y el cliente syslog en su estación IDM Cliente, genere acciones que genere errores o denegacion de permisos para auditarlos




<h3><strong>Laboratorio 1</strong></h3>
<br>Implemente el servicio de AIDE usando como procedimiento el ejemplo del instructor

<p><br><a href="idm.md">volver</a></p>
