<h1>Registrar una maquina cliente al IDM y valida su funcionamiento</h1>

<p>
<strong>Meta:</strong>
<br>Entender el proceso de instalación del cliente IDM para ejecutar operaciones en el.
</p>
<p>
<strong>Objetivos:</strong>
<br>- Instalar el cliente IDM
<br>- Aplicar políticas de acceso al cliente IDM
</p>
<p>
<strong>Secciones:</strong>
<br>- Implementación y uso del cliente IDM
</p>
<p>
<strong>Laboratorios:</strong>
<br><strong>- Instalación del cliente Red Hat IDM</strong>
<br><strong>- Configuración del cliente Red Hat IDM</strong>
<br><strong>- Validación de la regla SUDO</strong>
<br><strong>- Gestión de equipos</strong>

</p>

<strong>Requisitos:</strong>
<br><strong>- Sistema Operativo RHEL 9.4</strong>
</p>

<h3><strong>## Instalación del cliente Red Hat IDM</strong></h3>

El proceso de instalación inicia con la habilitación de los repositorios de sistema operativo `rhel-9-for-x86_64-baseos-rpms` y `rhel-9-for-x86_64-appstream-rpms` con los comandos:

`# subscription-manager repos --enable=rhel-9-for-x86_64-baseos-rpms`

`# subscription-manager repos --enable=rhel-9-for-x86_64-appstream-rpms`


En caso de no contar con una suscripción activa, se puede utilizar un repositorio creado a partir del ISO de RHEL 8.2 editando el archivo de configuración de repositorio en la ruta `/etc/yum.repos.d/rhel.repo` con el siguiente contenido:
```text
[root@server ~]# vi /etc/yum.repos.d/rhel.repo
[baseos]
name=baseos
baseurl=ftp://ftp.migraciones.gob.pe/rhel94/BaseOS
enabled=1
gpgheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
[appstream]
name=appstream
baseurl=ftp://ftp.migraciones.gob.pe/rhel94/AppStream
enabled=1
gpgheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
```


Finalmente, el proceso de instalación se podrá iniciar con el comando:
```
[root@client ~]# yum install ipa-client -y
Updating Subscription Management repositories.

Complete!
```

<h3><strong>## Configuración del cliente Red Hat IDM</strong></h3>

Una vez instalado los paquetes necesarios para el servidor Red Hat IDM procedemos a configurar el servicio, para ello primero debemos validar que el servidor cumpla una resolución de nombres básica con el comando `dig`
```
[root@client ~]# dig +short server.migraciones09.gob.pe
192.168.109.10
```

A diferencia del servidor IDM el cliente debe integrarse por DNS completamente, por eso debemos validar la configuración del adaptador de red de la siguiente manera
<br>`# cat /etc/NetworkManager/system-connections/ens*  | grep dns`
<br>`dns=192.168.109.10;`

Ahora ya podemos configurar el cliente IDM referenciando al servidos ntp 192.168.100.129
```
[root@client ~]# ipa-client-install --ntp-server=192.168.100.129 --mkhomedir
This program will set up IPA client.
Version 4.11.0

Discovery was successful!
Client hostname: client.migraciones09.gob.pe
Realm: MIGRACIONES09.GOB.PE
DNS Domain: migraciones09.gob.pe
IPA Server: server.migraciones09.gob.pe
BaseDN: dc=migraciones09,dc=gob,dc=pe
NTP server: 192.168.100.129

Continue to configure the system with these values? [no]: yes
Synchronizing time
Configuration of chrony was changed by installer.
Attempting to sync time with chronyc.
Time synchronization was successful.
User authorized to enroll computers: admin
Password for admin@MIGRACIONES09.GOB.PE: ******

Client configuration complete.
The ipa-client-install command was successful
```

El cliente IDM ya se encuentra instalado y configurado, ahora se podrán realizar las validaciones de las políticas de acceso definidas en el capitulo anterior


<h3><br><strong>## Validación de la regla SUDO</strong></h3>

Durante la creación de la regla SUDO respondimos a la sección de QUIEN y QUE pueden ejecutar la política, ahora podemos responder a la pregunta DONDE, para ellos modificamos la regla en la siguiente sección
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm601.png?raw=true"></p>

Buscamos nuestro cliente recién registrado y lo desplazamos de la izquierda a la derecha
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm602.png?raw=true"></p>

Aceptamos los cambios en la regla<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm603.png?raw=true"></p>


<h3><br><strong>Laboratorio</strong></h3>
<br>Modifique su regla de acceso del capitulo anterior para que sea ejecutable desde su cliente
<br>-

<h3><br><strong>Laboratorio</strong></h3>
<br>Valide la constraseña de su usuario personal con el siguiente procedimiento
<br>- Inicie sesion via ssh con su usuario personal del idm en el servidor idm
<br>- Ingrese la contraseña actual
<br>- De ser necesario actualice la constraseña
<br>-

<h3><br><strong>Laboratorio</strong></h3>
<br>Con la regla terminada podemos ingresar a un terminal y validar el acceso desde el servidor `client`
<br>`# su - andy.reyes`
<br>`$ cd /tmp`
<br>`$ touch file`
<br>`$ sudo chown root file`
<br>`$ sudo chgrp root file`
<br>`$ ll file`



<h3><br><strong>## Gestión de equipos</strong></h3>


Considerando que los servicios informáticos están formados por grupos de servidores divididos en ambientes como producción, desarrollo y qa, es necesario gestionar los equipos en grupos para dividirlos por su función o ambiente.

Para crear un grupo de servidores ingresamos a la sección IDENTIDAD > GRUPOS > EQUIPOS
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm604.png?raw=true"></p>

Agregamos un nuevo grupo
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm605.png?raw=true"></p>

Ingresamos los datos del nuevo grupo
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm606.png?raw=true"></p>

Seleccionamos el nuevo grupo
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm607.png?raw=true"></p>

Agregamos un equipo al nuevo grupo
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm608.png?raw=true"></p>

Desplazamos un equipo de la parte izquierda a la derecha
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm609.png?raw=true"></p>

Aceptamos los cambios
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm610.png?raw=true"></p>

Para validar los integrantes de un grupo de equipos podemos usar la linea de comandos
<br>`# kinit admin@DOMAIN01.NET`
<br>`# ipa hostgroup-show test`

<strong>Ejercicio Propuesto</strong>

A continuación se presenta una estructura de grupos de comandos SUDO

<br>REDES
<br>/sbin/route
<br>/sbin/ifconfig
<br>/bin/ping
<br>/sbin/dhclient
<br>/usr/bin/net
<br>/sbin/iptables
<br>/usr/bin/rfcomm
<br>/usr/bin/wvdial
<br>/sbin/iwconfig
<br>/sbin/mii-tool

<br>SERVICIOS
<br>/sbin/service
<br>/sbin/chkconfig
<br>/usr/bin/systemctl start
<br>/usr/bin/systemctl stop
<br>/usr/bin/systemctl reload
<br>/usr/bin/systemctl restart
<br>/usr/bin/systemctl status
<br>/usr/bin/systemctl enable
<br>/usr/bin/systemctl disable

<br>ALMACENAMIENTO
<br>/sbin/fdisk
<br>/sbin/sfdisk
<br>/sbin/parted
<br>/sbin/partprobe
<br>/bin/mount
<br>/bin/umount

<br>PROCESOS
<br>/bin/nice
<br>/bin/kill
<br>/usr/bin/kill
<br>/usr/bin/killall

<br>SOFTWARE
<br>/bin/rpm
<br>/usr/bin/up2date
<br>/usr/bin/yum
<br>/usr/bin/dnf

<br>Donde se creara el grupo soporte que incluya a su usuario personal
<br>Ademas crear 2 grupos de equipos, Todos con los servidores IDM y Client y el grupo LinuxServers que solo incluya a client

Defina los siguientes la siguiente estructura de permisos en su servidor IDM
<br>Politica-Quien-Que-Donde
<br>AccesoRedes-soporte-Redes-TODOS
<br>AccesoServicios-soporte-Servicios-TODOS
<br>AccesoAlmacenamiento-soporte-Almacenamiento-LinuxServers
<br>AccesoProcesos-soporte-Procesos-LinuxServers
<br>AccesoSoftware-soporte-Software-LinuxServers


<p><br><a href="idm">volver</a></p>
