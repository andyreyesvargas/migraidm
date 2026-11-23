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

El cliente IDM ya se encuentra instalado y configurado, ahora se podrán realizar las validaciones de las políticas de acceso definidas en el capitulo anterior, podemos validar la integración haciendo consultas de usuarios de ejercicios previos que no existen de forma local
```
[root@client ~]# id andy.reyes
uid=70003(andy.reyes) gid=70003(andy.reyes) groups=70003(andy.reyes),70007(operadores)
[root@client ~]# id jaime.zamudio
uid=70002(jaime.zamudio) gid=70002(editors) groups=70002(editors),70007(operadores)
```
<h3><br><strong>## Desinstalar cliente IDM</strong></h3>
El cliente IDM puede ser desinstalado con el siguiente procedimiento, el sistema debera reiniciar necesariamente

```
[root@client ~]# ipa-client-install --uninstall
Unenrolling client from IPA server
Removing Kerberos service principals from /etc/krb5.keytab
Disabling client Kerberos and LDAP configurations
Redundant SSSD configuration file /etc/sssd/sssd.conf was moved to /etc/sssd/sssd.conf.deleted
Restoring client configuration files
Unconfiguring the NIS domain.
nscd daemon is not installed, skip configuration
nslcd daemon is not installed, skip configuration
Systemwide CA database updated.
Client uninstall complete.
The original nsswitch.conf configuration has been restored.
You may need to restart services or reboot the machine.
Do you want to reboot the machine? [no]: yes
The ipa-client-install command was successful
```

Tambien borramos el equipo del GUI incluyendo los registros DNS que se crearon al registrarlo
<p align="center"><img src="../images/idm62.png?raw=true"></p>
<p align="center"><img src="../images/idm63.png?raw=true"></p>

<h3><br><strong>## Instalar cliente IDM con OTP</strong></h3>
Para registrar un cliente IDM usando una contraseña de un solo, debemos crear el registro DNS y el registro vacio con una contraseña de un solo desde el server

```
[root@server ~]# kinit admin
Password for admin@MIGRACIONES09.GOB.PE:
[root@server ~]# ipa dnsrecord-add migraciones09.gob.pe client --a-rec 192.168.109.11
ipa dnsrecord-add 109.168.192.in-addr.arpa. 11 --ptr-rec client.migraciones09.gob.pe.
  Record name: client
  A record: 192.168.109.11
[root@server ~]# ipa host-add client.migraciones09.gob.pe --random
----------------------------------------
Added host "client.migraciones09.gob.pe"
----------------------------------------
  Host name: client.migraciones09.gob.pe
  Random password: 7AlexHtraATJGunHqXjEoJi
  Password: True
  Keytab: False
  Managed by: client.migraciones09.gob.pe
```

En el ejemplo anterior se crea una contraseña como RAMDOM PASSWORD, extraemos eso para ejecutarlo en el cliente y tener una instalacion y configuración de cliente sin exponer la clave de admin

```
[root@client ~]# ipa-client-install --ntp-server=192.168.100.129 --mkhomedir --password='7AlexHtraATJGunHqXjEoJi'   --unattended
This program will set up IPA client.
Version 4.11.0

Discovery was successful!
Client hostname: client.migraciones09.gob.pe
Realm: MIGRACIONES09.GOB.PE
DNS Domain: migraciones09.gob.pe
IPA Server: server.migraciones09.gob.pe
BaseDN: dc=migraciones09,dc=gob,dc=pe
NTP server: 192.168.100.129

Synchronizing time
Configuration of chrony was changed by installer.
Attempting to sync time with chronyc.
Time synchronization was successful.
Downloading the CA certificate via HTTP, this is INSECURE
Successfully retrieved CA cert
    Subject:     CN=Certificate Authority,O=MIGRACIONES09.GOB.PE
    Issuer:      CN=Certificate Authority,O=MIGRACIONES09.GOB.PE
    Valid From:  2026-11-18 05:27:46+00:00
    Valid Until: 2046-11-18 05:27:46+00:00

Enrolled in IPA realm MIGRACIONES09.GOB.PE
Created /etc/ipa/default.conf
Configured /etc/sssd/sssd.conf
Systemwide CA database updated.
Adding SSH public key from /etc/ssh/ssh_host_ecdsa_key.pub
Adding SSH public key from /etc/ssh/ssh_host_ed25519_key.pub
Adding SSH public key from /etc/ssh/ssh_host_rsa_key.pub
SSSD enabled
Configured /etc/openldap/ldap.conf
Principal is not set when enrolling with OTP or PKINIT; using principal 'admin@migraciones09.gob.pe' for 'getent passwd'.
Configured /etc/ssh/ssh_config
Configured /etc/ssh/sshd_config.d/04-ipa.conf
Configuring migraciones09.gob.pe as NIS domain.
Configured /etc/krb5.conf for IPA realm MIGRACIONES09.GOB.PE
Client configuration complete.
The ipa-client-install command was successful
```

Se valida con algunos comandos que busquen la base de datos de idm
```
[root@client ~]# id andy.reyes
uid=70003(andy.reyes) gid=70003(andy.reyes) groups=70003(andy.reyes),70007(operadores)
[root@client ~]# id jaime.zamudio
uid=70002(jaime.zamudio) gid=70002(editors) groups=70002(editors),70007(operadores)
```

<h3><br><strong>## Gestión de equipos</strong></h3>

Considerando que los servicios informáticos están formados por grupos de servidores divididos en ambientes como producción, desarrollo y qa, es necesario gestionar los equipos en grupos para dividirlos por su función o ambiente.

Para crear un grupo de servidores ingresamos a la sección IDENTIDAD > GRUPOS > GRUPOS DE EQUIPOS
<p align="center"><img src="../images/idm64.png?raw=true"></p>

Considerando que los servidores SERVER y CLIENT son RHEL9, podemos agruparlo en un grupo RHEL9 para politicas de seguridad exclusivas de esa version
<p align="center"><img src="../images/idm65.png?raw=true"></p>

Ahora agregamos los server al grupo
<p align="center"><img src="../images/idm66.png?raw=true"></p>
<p align="center"><img src="../images/idm67.png?raw=true"></p>

<h3><br><strong>## Validacion de reglas SUDO</strong></h3>
Para realizar una gestion correcta de permisos, se trabajara solo con grupos, es decir, crearemos un grupo file_admins el cual integrara algunos usuarios que haran la gestion de permisos de archivos como los comandos chgrp, chwon entre otros

Agregamos un grupo
<p align="center"><img src="../images/idm68.png?raw=true"></p>

Creamos el grupo file_admins
<p align="center"><img src="../images/idm69.png?raw=true"></p>

Agregamos usuarios al grupo
<p align="center"><img src="../images/idm70.png?raw=true"></p>
<p align="center"><img src="../images/idm71.png?raw=true"></p>

Vamos a las reglas sudo y editamos la regla de gestion de archivos, ahora podemos reordenarla para que cumplan el siguiente requisito
QUIEN: grupo file_admins
EQUIPOS: grupo rhel9
COMANDOS: grupo gestionar archivos
<p align="center"><img src="../images/idm72.png?raw=true"></p>
<p align="center"><img src="../images/idm73.png?raw=true"></p>

A partir de ahora, la regla deberia permanecer y solo agregar integrantes a los grupos

<h3><br><strong>## Usuario super-admin</strong></h3>
Si se desea un usuario super-admin que tenga acceso tanto al IDM como a los clientes, podemos hacer la creacion via IDM pero se considera algunos escenarios para prevenir el uso indebido de este usuario

<br>- No replicar el procedimiento en un grupo, ya que es un rol tan especial que deberia ser unico(aplicarlo al usuario)
<br>- Considerar la seguridad del doble factor de autenticación o una contraseña compleja en extremo
<br>- Agregar el usuario al grupo admins
<br>- Reservar el usuario para labores de gestion y recuperacion ante eventos complejos

Se crea el usuario admin_nigra
<p align="center"><img src="../images/idm74.png?raw=true"></p>
<p align="center"><img src="../images/idm75.png?raw=true"></p>
<p align="center"><img src="../images/idm76.png?raw=true"></p>

Se crea la regla SUDO super_admin
<p align="center"><img src="../images/idm77.png?raw=true"></p>

Hacemos que la regla sudo tenga la opcion !authenticate para que no pida contraseña al momento de ejecutar los comandos sudo, en quien el usuario admin_migra como unico usuario con ese rol de super_admin y todas las demas opciones lo dejamos en CUALQUIERA
<p align="center"><img src="../images/idm78.png?raw=true"></p>
<p align="center"><img src="../images/idm79.png?raw=true"></p>

Podemos validar el usuario con un comando administrativo que no haya sido definido en el IDM, por ejemplo el fdisk
```
login as: admin_migra
Keyboard-interactive authentication prompts from server:
| Password: ***

[admin_migra@client ~]$ sudo fdisk -l /dev/sda
Disk /dev/sda: 100 GiB, 107374182400 bytes, 209715200 sectors
Disk model: Virtual disk
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 09876B94-DE4C-4BDF-8C20-7BC84AF2DFF9

Device       Start       End   Sectors  Size Type
/dev/sda1     2048   1230847   1228800  600M EFI System
/dev/sda2  1230848   3327999   2097152    1G Linux filesystem
/dev/sda3  3328000 209713151 206385152 98.4G Linux LVM
```

<h3><br><strong>## Reglas HBAC</strong></h3>
Las reglas HBAC son las politicas que permiten el acceso a los servidores registrados como clientes, estos servicios van desde el login, sesion ssh, sesion ftp, acceso web, entre otros, son muy aparte a las reglas sudo que se relacionan con comandos, estos se relacionan mas a servicios de sistema operativo, para identificar esto, vamos a la seccion de HBAC en POLITICAS y vemos 2 reglas por default que dan acceso a servicios de sistema operativo, estas reglas solo dan el acceso inicial, si el sistema tiene un firewall, ACL, o restricciones locales, estas restricciones se van a aplicar
<p align="center"><img src="../images/idm80.png?raw=true"></p>

Vamos a un terminal y ejecutamos un acceso remoto, siendo aplicado el acceso al servicio de login y ssh
```
login as: andy.reyes
Keyboard-interactive authentication prompts from server:
| Password:
End of keyboard-interactive prompts from server
Register this system with Red Hat Insights: insights-client --register
Create an account or view all your systems at https://red.ht/insights-dashboard
Last login: Sun Nov 22 23:34:59 2026 from 192.168.109.1
[andy.reyes@client ~]$
```

Ahora seleccionamos las reglas HBAC y las desactivamos
<p align="center"><img src="../images/idm81.png?raw=true"></p>

Ahora repetimos el logueo
```
login as: andy.reyes
Keyboard-interactive authentication prompts from server:
| Password:
End of keyboard-interactive prompts from server
Access denied
```

Los HBAC permite crear acceso a servicios, creando un nueva capa de seguridad, donde el usuario o grupo de usuarios primero debe tener un acceso permitido al servidor y luego tener acceso a recursos local con reglas SUDO.

Vamos a crear un grupo de usuarios rhel_admins que tengan acceso a los servidores, este HBAC no otorga permisos a comandos sudo, para ellos la regla sudo debe estar definida, este solo dará acceso al logueo via SSH
<p align="center"><img src="../images/idm82.png?raw=true"></p>

Ahora creamos la regla de HBAC que permita solo el servicio de autenticacion remota por ssh y los comandos sudo, para esta prueba las reglas default deben estar desactivadas para prevenir que den acceso completo a recursos
<p align="center"><img src="../images/idm83.png?raw=true"></p>
<p align="center"><img src="../images/idm84.png?raw=true"></p>
<p align="center"><img src="../images/idm85.png?raw=true"></p>
<p align="center"><img src="../images/idm86.png?raw=true"></p>

Ahora podemos loguearnos al sistema client con algun integrante del grupo rhel_admins para validar que pueda ejecutar las reglas sudo definidas en ejercicios previos
```
login as: jaime.zamudio
Keyboard-interactive authentication prompts from server:
| Password:
End of keyboard-interactive prompts from server
Register this system with Red Hat Insights: insights-client --register
Create an account or view all your systems at https://red.ht/insights-dashboard
Last login: Mon Nov 23 10:53:42 2026 from 192.168.109.1
[jaime.zamudio@client ~]$ sudo chown root.root a b c
[sudo] password for jaime.zamudio:
[jaime.zamudio@client ~]$ ll
total 0
-rw-r--r--. 1 root root 0 Nov 22 23:11 a
-rw-r--r--. 1 root root 0 Nov 22 23:11 b
-rw-r--r--. 1 root root 0 Nov 22 23:11 c
```

<h3><br><strong>## Reglas HBAC para servicios restringidos</strong></h3>
Se pueden crear reglas de HBAC para dar acceso a servicios pubicos que sean restringidos a usuarios del IDM, sin que suponga dar acceso al terminal SSH o las reglas SUDO, por ejemplo un servicio ftp o web da un acceso via cliente web o cliente ftp sin dar un terminal al linux ni tener reglas de comandos, para ellos vamos a crear un servicio el el cliente, el servicio vsftp
```
[root@client ~]# yum install -y vsftpd ftp
[root@client ~]# systemctl enable vsftpd --now
[root@client ~]# firewall-cmd --permanent --add-service=ftp
[root@client ~]# firewall-cmd --reload
```

Vamos a crear un grupo nuevo que incluya  a usuarios que no han sido definidos en reglas de acceso anteriores
<p align="center"><img src="../images/idm87.png?raw=true"></p>

Del grupo anterior seleccionamos un usuario para nuestras pruebas, a ese usuario le creamos un home en el client con algun contenido ftp
```
[root@client ~]# cd /home/
[root@client home]# ll
total 0
drwx------. 2 admin_migra   admin_migra  83 Nov 22 23:30 admin_migra
drwx------. 2 andy.reyes    andy.reyes   83 Nov 22 23:48 andy.reyes
drwx------. 2 jaime.zamudio editors     110 Nov 22 23:29 jaime.zamudio

[root@client home]# id julio.torres
uid=70021(julio.torres) gid=70021(julio.torres) groups=70021(julio.torres),70029(alpha),70034(ftp_users),70028(todos)
[root@client home]# mkdir julio.torres
[root@client home]# echo "Hola Julio" > julio.torres/test.txt
[root@client home]# chown -R 70021.70021 julio.torres
```

Con los recursos creados, procedemos con la regla HBAC
<p align="center"><img src="../images/idm88.png?raw=true"></p>
<p align="center"><img src="../images/idm89.png?raw=true"></p>

Ahora vamos a un cliente ftp y validamos el acceso a julio.torres
<p align="center"><img src="../images/idm90.png?raw=true"></p>
<p align="center"><img src="../images/idm91.png?raw=true"></p>

Como la idea es tener un acceso ftp sin dar privilegios de ssh o sudos, se valida tambien que no se pueda loguear por cliente ssh
```
login as: julio.torres
Keyboard-interactive authentication prompts from server:
| Password:
End of keyboard-interactive prompts from server
Access denied
```


<h3><br><strong>## Laboratorio 01</strong></h3>
<br>- Registrar el equipo client en el IDM (cualquier método)
<br>- Crear el grupo RHEL9 para que incluya a su client y server
<br>- Crear el grupo file_admins e ingrese un usuario en ese grupo
<br>- Modifique su regla SUDO para que ahora sea gestionado por los grupos
<br>      QUIEN: grupo file_admins
<br>      EQUIPOS: grupo rhel9
<br>      COMANDOS: grupo gestionar archivos

<h3><br><strong>## Laboratorio 02</strong></h3>
Cree un usuario super_admin con todos los permisos en todos los equipos sin contraseña de comandos sudo

<h3><br><strong>## Laboratorio 03</strong></h3>
Use las reglas HBAC para añadir la capa de servicios, solo SSH y SUDO a los grupos permitidos en sus reglas SUDO, como RHEL_admins

<h3><br><strong>## Laboratorio 04</strong></h3>
Implemente el servicio VSFTP para permitir el acceso al ftp instalado en su CLIENT, el cual debera estar disponible para un grupo de usuarios que no tengan ningún acceso SSH o SUDO, seleccione usuarios que no han sido usados aun, haga sus pruebas creando un home a uno de esos usuarios

<h3><br><strong>## Laboratorio 05</strong></h3>
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
<br>Ademas crear 2 grupos de equipos, TODOS con los servidores server y Client y el grupo LINUX que solo incluya a client

Defina los siguientes la siguiente estructura de permisos en su servidor IDM
<br>Politica-Quien-Comandos-Equipos
<br>AccesoRedes-soporte-Redes-TODOS
<br>AccesoServicios-soporte-Servicios-TODOS
<br>AccesoAlmacenamiento-soporte-Almacenamiento-LINUX
<br>AccesoProcesos-soporte-Procesos-LINUX
<br>AccesoSoftware-soporte-Software-LINUX


<p><br><a href="idm.md">volver</a></p>
