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
<br><strong>- Sistema Operativo RHEL 8.2</strong>
<br><strong>- Alumno 01:</strong><br>  - idm: 192.168.100.11 client: 192.168.100.21 dominio: domain01.net
<br><strong>- Alumno 02:</strong><br>  - idm: 192.168.100.12 client: 192.168.100.22 dominio: domain02.net
<br><strong>- Alumno 03:</strong><br>  - idm: 192.168.100.13 client: 192.168.100.23 dominio: domain03.net
<br><strong>- Alumno 04:</strong><br>  - idm: 192.168.100.14 client: 192.168.100.24 dominio: domain04.net
<br><strong>- Alumno 05:</strong><br>  - idm: 192.168.100.15 client: 192.168.100.25 dominio: domain05.net
<br><strong>- Alumno 06:</strong><br>  - idm: 192.168.100.16 client: 192.168.100.26 dominio: domain06.net
<br><strong>- Alumno 07:</strong><br>  - idm: 192.168.100.17 client: 192.168.100.27 dominio: domain07.net
</p>

<strong>## Instalación del servidor Red Hat IDM</strong>

El proceso de instalación inicia con la habilitación de los repositorios de sistema operativo `rhel-8-for-x86_64-baseos-rpms` y `rhel-8-for-x86_64-appstream-rpms` con los comandos:

```
# subscription-manager register
# subscription-manager list --available
# subscription-manager attach --pool=**************
# subscription-manager repos --enable=rhel-8-for-x86_64-baseos-rpms
# subscription-manager repos --enable=rhel-8-for-x86_64-appstream-rpms
```


En caso de no contar con una suscripción activa, se puede utilizar un repositorio creado a partir del ISO de RHEL 8.2 editando el archivo de configuración de repositorio en la ruta `/etc/yum.repos.d/rhel.repo` con el siguiente contenido:
```text
[BaseOS]
name=BaseOS
enabled=1
baseurl=ftp://192.168.100.203/rhel82/BaseOS
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
###
[AppStream]
name=AppStream
enabled=1
baseurl=ftp://192.168.100.203/rhel82/AppStream
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
###
```


El comando `dnf repolist` nos permitirá validar la disponibilidad de los repositorios ya sean por suscripción o por ISO

`# dnf repolist`

Como buena practica, el sistema IDM debe contar con una hora sincronizada, para ello habilitar la sincronización con el servidor ntp del salón 192.168.100.200

```
# yum install chrony -y
# vi /etc/chrony.conf
server 192.168.100.200 iburst
# systemctl restart chronyd
# systemctl enable chronyd
# chronyc sources
```

En caso de realizar la conexión remota por una red inestable o que tiende a desconexiones, podemos proteger la conexión con el aplicativo screen

```
# yum install -y ftp://ftp.opennova.pe/instaladores/screen-4.6.2-12.el8.x86_64.rpm
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

Al validar la disponibilidad de los repositorios se podrá activar el módulo de IDM con:

`# dnf module enable idm:DL1`

`# dnf distro-sync`

Finalmente, el proceso de instalación se podrá iniciar con el comando:

`# dnf module install idm:DL1/server`

Considerar que el proceso de instalación puede realizarse con diferentes módulos además del servidor base, también existen otros módulos:
<br>- server
<br>- dns
<br>- adtrust
<br>- client

Ahora podemos instalar los modulos de dns y el cliente idm
<br>`# dnf module install idm:DL1/dns`
<br>`# dnf module install idm:DL1/client`

<strong>## Configuración del servidor Red Hat IDM</strong>

Una vez instalado los paquetes necesarios para el servidor Red Hat IDM procedemos a configurar el servicio, para ello primero debemos validar que el servidor cumpla una resolución de nombres básica con el comando `dig`

`# dig +short idm.domain08.net

En caso de no contar un registro DNS podemos utilizar el registro local en el archivo `/etc/hosts` para cumplir con el proceso de instalación hasta que el administrador de red cree el registro en el DNS corporativo

`# echo "192.168.100.18 idm.domain08.net" >> /etc/hosts`

Validamos el nuevo registro

`# cat /etc/hosts`

Para configurar el sistema usamos el tool `ipa-server-install` el cual realiza las preguntas necesarias para generar la configuracion del sistema

`# ipa-server-install`

En caso el dominio ya exista en la red o internet, el instalador nos mostrara la alerta y no permitirá configurar el sistema, en caso se de el error debemos indicarle al sistema que fuerce la configuración a pesar de existir un dominio similar

`# ipa-server-install  --allow-zone-overlap`

Debemos responder las preguntas de instalar con la siguiente configuración sugerida
<br>Do you want to configure integrated DNS (BIND)?: <strong>yes</strong>
<br>Server host name: <strong>idm.domain08.net</strong>
<br>Please confirm the domain name: <strong>domain08.net</strong>
<br>Please provide a realm name: <strong>DOMAIN08.NET</strong>
<br>Directory Manager password: <strong>Passw0rd$</strong>
<br>Password (confirm): <strong>Passw0rd$</strong>
<br>IPA admin password: <strong>Passw0rd$</strong>
<br>Password (confirm): <strong>Passw0rd$</strong>
<br>Do you want to configure DNS forwarders?: <strong>no</strong>
<br>Do you want to search for missing reverse zones?: <strong>no</strong>
<br>Do you want to configure chrony with NTP server or pool address?: <strong>no</strong>
<br>
<br>Finalmente se confirman los cambios
<br>Continue to configure the system with these values?: <strong>yes</strong>

Con la instalación del sistema se debe también aperturar los servicios a través del firewall, para ello configuramos las reglas de firewall de acceso

<br>`# firewall-cmd --permanent --add-service=http`
<br>`# firewall-cmd --permanent --add-service=https`
<br>`# firewall-cmd --permanent --add-service=ldap`
<br>`# firewall-cmd --permanent --add-service=ldaps`
<br>`# firewall-cmd --permanent --add-service=kerberos`
<br>`# firewall-cmd --permanent --add-service=dns`
<br>`# firewall-cmd --permanent --add-service=ntp`
<br>`# firewall-cmd --reload`

Ahora podemos examinar la plataforma ingresando a la ruta `https://idm.domain08.net`, de ser necesario agregar `192.168.100.18 idm.domain08.net` al archivo hosts de su estación de trabajo.

Cambien podemos examinar el estado de los servicios idm con el comando
```
# ipactl status
```

<strong>## Desinstalación del servidor Red Hat IDM</strong>

El proceso de desinstalación permite recuperar el sistema operativo para su uso en otras aplicaciones, para detener los servicios y desinstalarlos del sistema se debe usar el comando:

```
# ipa-server-install --uninstall
```

Posterior la desinstalación, se recomienda reiniciar el sistema para depurar cualquier información en memoria del sistema anterior.

`# init 6`


<strong>## Instalación del servidor Red Hat IDM avanzada</strong>

Conociendo el proceso de instalación, podemos simplificarlo con parámetros avanzados desde la misma instalación, por ejemplo instalemos el sistema con la siguiente linea de comandos analizando las mejoras propuestas por el instructor

`# ipa-server-install --realm=DOMAIN08.NET --domain=domain08.net --ds-password=Passw0rd$ --admin-password=Passw0rd$ --hostname=idm.domain08.net --ip-address=192.168.100.18 --setup-dns --forwarder 192.168.100.200 --reverse-zone=100.168.192.in-addr.arpa --idstart=60000 --idmax=10000000 --allow-zone-overlap --no-ntp` 

De ser necesario, volver a configurar las reglas de firewall
<br>`# firewall-cmd --permanent --add-service=http`
<br>`# firewall-cmd --permanent --add-service=https`
<br>`# firewall-cmd --permanent --add-service=ldap`
<br>`# firewall-cmd --permanent --add-service=ldaps`
<br>`# firewall-cmd --permanent --add-service=kerberos`
<br>`# firewall-cmd --permanent --add-service=dns`
<br>`# firewall-cmd --permanent --add-service=ntp`
<br>`# firewall-cmd --reload`

Una vez instalado el sistema debería realizar respaldos de la infraestructura, podría apoyarse en soluciones de respaldo sobre plataforma de virtualización o en los comandos `ipa-backup` e `ipa-restore` y realizar actualizaciones con el comando `ipa-server-upgrade`

<h3><strong>Laboratorio</strong></h3>
<br>- Instale su propio servidor IDM en su dominio asignado de forma interactiva
<br>- Desinstale el servicio IDM para recuperar el sistema operativo
<br>- Reinicie el sistema
<br>- Instale el servidor IDM de forma automatizada con integración con el servidor DNS del salon(192.168.100.200) y los usuarios tenga el ID desde 70000 hasta 80000



<p><br><a href="idm">volver</a></p>
