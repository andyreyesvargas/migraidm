<h1>Entendimiento y visualización del proceso de instalación</h1>

<p>
<strong>Meta:</strong>
<br>Entender el proceso de instalación de la solución Red Hat Identity Management así como el diseño básico para una correcta implementación.
</p>
<p>
<strong>Objetivos:</strong>
<br>- Implementar Red Hat Identity Management
<br>- Actualizar la instalación de Red Hat Identity Management
</p>
<p>
<strong>Secciones:</strong>
<br>- Instalación de Red Hat IDM
<br>- Instalación de Red Hat IDM avanzada
</p>
<p>
<strong>Laboratorios:</strong>
<br><strong>- Instalación del servidor Red Hat IDM</strong>
<br><strong>- Instalación de Red Hat IDM</strong>
<br><strong>- Desinstalación de Red Hat IDM</strong>
<br><strong>- Instalación de Red Hat IDM avanzada</strong>

</p>

<strong>Requisitos:</strong>
<br><strong>- Sistema Operativo RHEL 9.4</strong>

<strong>## Instalación del servidor Red Hat IDM</strong>

El proceso de instalación inicia con la habilitación de los repositorios de sistema operativo `rhel-9-for-x86_64-baseos-rpms` y `rhel-9-for-x86_64-appstream-rpms` con los comandos:

```
# subscription-manager register
# subscription-manager repos --enable=rhel-9-for-x86_64-baseos-rpms
# subscription-manager repos --enable=rhel-9-for-x86_64-appstream-rpms
```


En caso de no contar con una suscripción activa, se puede utilizar un repositorio creado a partir del ISO de RHEL 9.4 editando el archivo de configuración de repositorio en la ruta `/etc/yum.repos.d/rhel.repo` con el siguiente contenido:
```
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

El comando `dnf repolist` nos permitirá validar la disponibilidad de los repositorios ya sean por suscripción o por ISO

```
[root@server ~]# yum repolist
Updating Subscription Management repositories.
Unable to read consumer identity

This system is not registered with an entitlement server. You can use "rhc" or "subscription-manager" to register.

repo id                                           repo name
appstream                                         appstream
baseos                                            baseos
```

Como buena practica, el sistema IDM debe contar con una hora sincronizada, para ello habilitar la sincronización con el servidor ntp del salón 192.168.100.200

```
[root@server ~]# yum install -y chrony
[root@server ~]# vi /etc/chrony.conf
pool ntp.migraciones.gob.pe iburst
[root@server ~]# systemctl enable chronyd --now
[root@server ~]# chronyc sources
MS Name/IP address         Stratum Poll Reach LastRx Last sample
===============================================================================
^? ntp.migraciones.gob.pe        0   6     0     -     +0ns[   +0ns] +/-    0ns
```
En caso de realizar la conexión remota por una red inestable o que tiende a desconexiones, podemos proteger la conexión con el aplicativo screen

```
[root@server ~]# yum install -y ftp://ftp.migraciones.gob.pe/screen-4.8.0-6.el9.x86_64.rpm
```

Crear conexión
```
# screen -S andy
```

Listar conexión screen, identificar el id ejemplo: 3081**
```
# screen -ls
There is a screen on:
        3081.andy      (Attached)
```

En caso de des-conexión abrir otro terminal y ejecutar el comando indicado para restablecer la conexión, recordar poner el id que se genere en su terminal**
```
# screen -r 3081
```

Para instalar el servidor de IDM usamos
```
[root@server ~]# yum install ipa-server
```
Como vamos a implementar el servidor IDM con su propio DNS, debemos instalar adicionalmente
```
[root@server ~]# yum install ipa-server-dns
```

En caso se requiera instalar complementos de IDM como integracion con Active Directory o Ansible, se podrán instalar a posterior

<strong>## Configuración del servidor Red Hat IDM</strong>

Una vez instalado los paquetes necesarios para el servidor Red Hat IDM procedemos a configurar el servicio, para ello primero debemos validar que el servidor cumpla una resolución de nombres básica con el comando `dig`
```
[root@server ~]# dig +short server.migraciones09.gob.pe
```

En caso de no contar un registro DNS podemos utilizar el registro local en el archivo `/etc/hosts` para cumplir con el proceso de instalación hasta que el administrador de red cree el registro en el DNS corporativo
```
[root@server ~]# echo "192.168.109.10 server.migraciones09.gob.pe" >> /etc/hosts
```

Validamos el nuevo registro
```
[root@server ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.109.10 server.migraciones09.gob.pe
```

Para configurar el sistema usamos el tool `ipa-server-install` el cual realiza las preguntas necesarias para generar la configuracion del sistema

`# ipa-server-install`

En caso el dominio ya exista en la red o internet, el instalador nos mostrara la alerta y no permitirá configurar el sistema, en caso se de el error debemos indicarle al sistema que fuerce la configuración a pesar de existir un dominio similar

`# ipa-server-install  --allow-zone-overlap`

```
[root@server ~]# ipa-server-install --allow-zone-overlap

The log file for this installation can be found in /var/log/ipaserver-install.log
==============================================================================
This program will set up the IPA Server.
Version 4.11.0

This includes:
  * Configure a stand-alone CA (dogtag) for certificate management
  * Configure the NTP client (chronyd)
  * Create and configure an instance of Directory Server
  * Create and configure a Kerberos Key Distribution Center (KDC)
  * Configure Apache (httpd)
  * Configure SID generation
  * Configure the KDC to enable PKINIT

To accept the default shown in brackets, press the Enter key.

Do you want to configure integrated DNS (BIND)? [no]: yes

Enter the fully qualified domain name of the computer
on which you're setting up server software. Using the form
<hostname>.<domainname>
Example: master.example.com


Server host name [server.migraciones09.gob.pe]:

Warning: skipping DNS resolution of host server.migraciones09.gob.pe
The domain name has been determined based on the host name.

Please confirm the domain name [migraciones09.gob.pe]:

The kerberos protocol requires a Realm name to be defined.
This is typically the domain name converted to uppercase.

Please provide a realm name [MIGRACIONES09.GOB.PE]:
Certain directory server operations require an administrative user.
This user is referred to as the Directory Manager and has full access
to the Directory for system management tasks and will be added to the
instance of directory server created for IPA.
The password must be at least 8 characters long.

Directory Manager password:
Password (confirm):

The IPA server requires an administrative user, named 'admin'.
This user is a regular system account used for IPA server administration.

IPA admin password:
Password (confirm):

Checking DNS domain migraciones09.gob.pe., please wait ...
DNS zone migraciones09.gob.pe. already exists in DNS and is handled by server(s): idm.migraciones.gob.pe. Please make sure that the domain is properly delegated to this IPA server.
Do you want to configure DNS forwarders? [yes]:
Following DNS servers are configured in /etc/resolv.conf: 192.168.100.129
Do you want to configure these servers as DNS forwarders? [yes]:
All detected DNS servers were added. You can enter additional addresses now:
Enter an IP address for a DNS forwarder, or press Enter to skip:
DNS forwarders: 192.168.100.129
Checking DNS forwarders, please wait ...
DNS server 192.168.100.129 does not support DNSSEC: answer to query '. SOA' is missing DNSSEC signatures (no RRSIG data)
Please fix forwarder configuration to enable DNSSEC support.

DNS server 192.168.100.129: answer to query '. SOA' is missing DNSSEC signatures (no RRSIG data)
Please fix forwarder configuration to enable DNSSEC support.
WARNING: DNSSEC validation will be disabled
Do you want to search for missing reverse zones? [yes]:
Reverse record for IP address 192.168.109.10 already exists
Trust is configured but no NetBIOS domain name found, setting it now.
Enter the NetBIOS name for the IPA domain.
Only up to 15 uppercase ASCII letters, digits and dashes are allowed.
Example: EXAMPLE.


NetBIOS domain name [MIGRACIONES09]:

Do you want to configure chrony with NTP server or pool address? [no]:

The IPA Master Server will be configured with:
Hostname:       server.migraciones09.gob.pe
IP address(es): 192.168.109.10
Domain name:    migraciones09.gob.pe
Realm name:     MIGRACIONES09.GOB.PE

The CA will be configured with:
Subject DN:   CN=Certificate Authority,O=MIGRACIONES09.GOB.PE
Subject base: O=MIGRACIONES09.GOB.PE
Chaining:     self-signed

BIND DNS server will be configured to serve IPA domain with:
Forwarders:       192.168.100.129
Forward policy:   only
Reverse zone(s):  No reverse zone

Continue to configure the system with these values? [no]: yes
```

El sistema deberia terminar de instalar
```
Setup complete

Next steps:
        1. You must make sure these network ports are open:
                TCP Ports:
                  * 80, 443: HTTP/HTTPS
                  * 389, 636: LDAP/LDAPS
                  * 88, 464: kerberos
                  * 53: bind
                UDP Ports:
                  * 88, 464: kerberos
                  * 53: bind
                  * 123: ntp

        2. You can now obtain a kerberos ticket using the command: 'kinit admin'
           This ticket will allow you to use the IPA tools (e.g., ipa user-add)
           and the web user interface.

Be sure to back up the CA certificates stored in /root/cacert.p12
These files are required to create replicas. The password for these
files is the Directory Manager password
The ipa-server-install command was successful
```


<strong>## Desinstalación del servidor Red Hat IDM</strong>

El proceso de desinstalación permite recuperar el sistema operativo para su uso en otras aplicaciones, para detener los servicios y desinstalarlos del sistema se debe usar el comando:

```
[root@server ~]# ipa-server-install --uninstall
This is a NON REVERSIBLE operation and will delete all data and configuration!
It is highly recommended to take a backup of existing data and configuration using ipa-backup utility before proceeding.

Are you sure you want to continue with the uninstall procedure? [no]: yes
```

Posterior la desinstalación, se recomienda reiniciar el sistema para depurar cualquier información en memoria del sistema anterior.
```
[root@server ~]# reboot
```


<strong>## Instalación del servidor Red Hat IDM avanzada</strong>

Conociendo el proceso de instalación, podemos simplificarlo con parámetros avanzados desde la misma instalación, por ejemplo instalemos el sistema con la siguiente linea de comandos analizando las mejoras propuestas por el instructor

```
[root@server ~]# ipa-server-install --realm=MIGRACIONES09.GOB.PE --netbios-name=MIGRACIONES09 --domain=migraciones09.gob.pe --ds-password=Passw0rd$ --admin-password=Passw0rd$ --hostname=server.migraciones09.gob.pe --ip-address=192.168.109.10 --setup-dns --forwarder 192.168.109.1 --reverse-zone=109.168.192.in-addr.arpa --idstart=60000 --idmax=10000000 --allow-zone-overlap --no-ntp

The log file for this installation can be found in /var/log/ipaserver-install.log
==============================================================================
This program will set up the IPA Server.
Version 4.11.0

This includes:
  * Configure a stand-alone CA (dogtag) for certificate management
  * Create and configure an instance of Directory Server
  * Create and configure a Kerberos Key Distribution Center (KDC)
  * Configure Apache (httpd)
  * Configure DNS (bind)
  * Configure SID generation
  * Configure the KDC to enable PKINIT

Excluded by options:
  * Configure the NTP client (chronyd)

To accept the default shown in brackets, press the Enter key.

Warning: skipping DNS resolution of host server.migraciones09.gob.pe
Checking DNS domain migraciones09.gob.pe., please wait ...
Checking DNS domain 109.168.192.in-addr.arpa, please wait ...
Checking DNS forwarders, please wait ...
DNS server 192.168.109.1 does not support DNSSEC: answer to query '. SOA' is missing DNSSEC signatures (no RRSIG data)
Please fix forwarder configuration to enable DNSSEC support.

DNS server 192.168.109.1: answer to query '. SOA' is missing DNSSEC signatures (no RRSIG data)
Please fix forwarder configuration to enable DNSSEC support.
WARNING: DNSSEC validation will be disabled
Using reverse zone(s) 109.168.192.in-addr.arpa.

The IPA Master Server will be configured with:
Hostname:       server.migraciones09.gob.pe
IP address(es): 192.168.109.10
Domain name:    migraciones09.gob.pe
Realm name:     MIGRACIONES09.GOB.PE

The CA will be configured with:
Subject DN:   CN=Certificate Authority,O=MIGRACIONES09.GOB.PE
Subject base: O=MIGRACIONES09.GOB.PE
Chaining:     self-signed

BIND DNS server will be configured to serve IPA domain with:
Forwarders:       192.168.109.1
Forward policy:   only
Reverse zone(s):  109.168.192.in-addr.arpa.

Continue to configure the system with these values? [no]: yes
```

La instalación deberia culminar con exito
```
Setup complete

Next steps:
        1. You must make sure these network ports are open:
                TCP Ports:
                  * 80, 443: HTTP/HTTPS
                  * 389, 636: LDAP/LDAPS
                  * 88, 464: kerberos
                  * 53: bind
                UDP Ports:
                  * 88, 464: kerberos
                  * 53: bind

        2. You can now obtain a kerberos ticket using the command: 'kinit admin'
           This ticket will allow you to use the IPA tools (e.g., ipa user-add)
           and the web user interface.

Be sure to back up the CA certificates stored in /root/cacert.p12
These files are required to create replicas. The password for these
files is the Directory Manager password
The ipa-server-install command was successful
```

Tambien se debe abrir los puertos del firewall
```
[root@server ~]# firewall-cmd --permanent --add-service=http
success
[root@server ~]# firewall-cmd --permanent --add-service=https
success
[root@server ~]# firewall-cmd --permanent --add-service=ldap
success
[root@server ~]# firewall-cmd --permanent --add-service=ldaps
success
[root@server ~]# firewall-cmd --permanent --add-service=kerberos
success
[root@server ~]# firewall-cmd --permanent --add-service=dns
success
[root@server ~]# firewall-cmd --permanent --add-service=ntp
success
[root@server ~]# firewall-cmd --reload
success
```

Una vez instalado el sistema debería realizar respaldos de la infraestructura, podría apoyarse en soluciones de respaldo sobre plataforma de virtualización o en los comandos `ipa-backup` e `ipa-restore` y realizar actualizaciones con el comando `ipa-server-upgrade`

<h3><strong>Laboratorio</strong></h3>
<br>- Instale su propio servidor IDM en su dominio asignado de forma interactiva
<br>- Desinstale el servicio IDM para recuperar el sistema operativo
<br>- Reinicie el sistema
<br>- Instale el servidor IDM de forma automatizada con integración con el servidor DNS del salon(192.168.100.129) y los usuarios tenga el ID desde 70000 hasta 80000



<p><br><a href="idm">volver</a></p>
