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
<br>`# dnf -y install aide`

En caso quisieramosmodificar la configuracion editamos el archivo `/etc/aide.conf`

Para inicializar la base de datos 
<br>`# aide --init`
<br>`# cp -p /var/lib/aide/aide.db.new.gz /var/lib/aide/aide.db.gz`

Para verificar la auditorio
<br>`# aide --check`

Intentamos generar cambios en el sistema
<br>`# touch /etc/file01`
<br>`# touch /etc/file02`
<br>`# touch /etc/file03`

Volvemos a verificar
<br>`# aide --check`

En caso no veamos cambios, podemos forzar una actualización con
<br>`# aide --update`

Es una buena practica actualizar la informacion de forma programada
<br>`# vi /etc/cron.d/aide`
<br>`00 00 * * * root /usr/sbin/aide --update`

<h3><strong>Laboratorio</strong></h3>
<br>Implemente el servicio de AIDE usando como procedimiento el ejemplo del instructor


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
<p><br><a href="idm">volver</a></p>
