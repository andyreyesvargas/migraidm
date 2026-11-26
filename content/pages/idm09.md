<h1>Tareas de mantenimiento</h1>

<p>
<strong>Meta:</strong>
<br>Realizar las tareas de mantenimiento típicas de la plataforma IDM
</p>
<p>
<strong>Objetivos:</strong>
<br>- Recuperación de clave de admin
<br>- Respaldo y restauración
<br>- Actualización
</p>

<strong>Requisitos:</strong>
<br><strong>- Servidor IDM</strong>
<br><strong>- Cliente IDM</strong><br>
</p>

<h3><strong>## Recuperación de la clave de admin</strong></h3>

La recuperación de la clave de admin se realiza con un terminal de linux como root, siendo necesario la clave de Directory Manager el cual es un dato de instalación que debe ser documentado, para ello usamos el siguiente comando en el caso del domain01 (adaptar a su dominio)

```
[root@server ~]# ldappasswd -D "cn=Directory Manager" -W -s <nueva contraseña> -H ldaps://server.migraciones09.gob.pe uid=admin,cn=users,cn=accounts,dc=migraciones09,dc=gob,dc=pe
```

<h3><strong>## Respaldo y recuperación de plataforma</strong></h3>

El respaldo de la plataforma puede hacerse con un script automatizado
```
[root@server ~]# ipa-backup
Preparing backup on server.migraciones09.gob.pe
Local roles match globally used roles, proceeding.
Stopping IPA services
Backing up ipaca in MIGRACIONES09-GOB-PE to LDIF
Backing up userRoot in MIGRACIONES09-GOB-PE to LDIF
Backing up MIGRACIONES09-GOB-PE
Backing up files
Starting IPA service
Backed up to /var/lib/ipa/backup/ipa-full-2026-11-26-18-39-06
The ipa-backup command was successful

```

El script de respaldo generara una carpeta con toda la información necesaria para restaurar la plataforma, por ello podemos borrar algunos elementos del IDM como usuarios o equipos, recuerde que para el procedimiento de restore debera contar con la contraseña de Directory Manager

```
[root@server ~]# ipa-restore /var/lib/ipa/backup/ipa-full-2026-11-26-18-39-06
Directory Manager (existing master) password:

Preparing restore from /var/lib/ipa/backup/ipa-full-2026-11-26-18-39-06 on server.migraciones09.gob.pe
Performing FULL restore from FULL backup
Temporary setting umask to 022
Restoring data will overwrite existing live data. Continue to restore? [no]: yes
Each master will individually need to be re-initialized or
re-created from this one. The replication agreements on
masters running IPA 3.1 or earlier will need to be manually
re-enabled. See the man page for details.
Disabling all replication.
Stopping IPA services
Systemwide CA database updated.
Restoring files
Systemwide CA database updated.
Restoring from userRoot in MIGRACIONES09-GOB-PE
Restoring from ipaca in MIGRACIONES09-GOB-PE
Restarting GSS-proxy
Starting IPA services
Restarting SSSD
Restarting oddjobd
Restarting IPA services
Restoring umask to 18
The ipa-restore command was successful
```

<h3><strong>## Actualización de la plataforma</strong></h3>

Para actualizar la plataforma de IDM podemos apoyarnos en la suscripcion o en un repositorio con un nivel de actualizacion superior al que se tiene implementado, por ejemplo nuestro repositorio apunta al ISO de RHEL 8.2

```
# cat /etc/yum.repos.d/rhel.repo
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
```

Antes de realizar cualquier mantenimiento se sugiere realizar respaldos de la plataforma a nivel de VM o una solucion de backup externa. Modificamos para que ahora apunte al RHEL 8.3
```
# vi /etc/yum.repos.d/rhel.repo
[BaseOS]
name=BaseOS
enabled=1
baseurl=ftp://192.168.100.203/rhel83/BaseOS
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
###
[AppStream]
name=AppStream
enabled=1
baseurl=ftp://192.168.100.203/rhel83/AppStream
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
```

Documentamos la version actual de IDM
```
# rpm -qa | grep ipa-server
ipa-server-dns-4.8.4-7.module+el8.2.0+6046+aaa49f96.noarch
ipa-server-common-4.8.4-7.module+el8.2.0+6046+aaa49f96.noarch
ipa-server-4.8.4-7.module+el8.2.0+6046+aaa49f96.x86_64
```

Realizamos la actualización de rpms

```
# yum update -y
```

Luego ejecutamos el comando ipa-server-upgrade

```
[root@idm ~]# ipa-server-upgrade
Upgrading IPA:. Estimated time: 1 minute 30 seconds
  [1/11]: stopping directory server
  [2/11]: saving configuration
  [3/11]: disabling listeners
  [4/11]: enabling DS global lock
  [5/11]: disabling Schema Compat
  [6/11]: starting directory server
  [7/11]: updating schema
  [8/11]: upgrading server
  [9/11]: stopping directory server
  [10/11]: restoring configuration
  [11/11]: starting directory server
Done.
Update complete
Upgrading IPA services
Upgrading the configuration of the IPA services
Disabled p11-kit-proxy
[Verifying that root certificate is published]
[Migrate CRL publish directory]
CRL tree already moved
[Verifying that KDC configuration is using ipa-kdb backend]
[Fix DS schema file syntax]
Syntax already fixed
[Removing RA cert from DS NSS database]
RA cert already removed
[Enable sidgen and extdom plugins by default]
[Updating HTTPD service IPA configuration]
[Updating HTTPD service IPA WSGI configuration]
Nothing to do for configure_httpd_wsgi_conf
[Migrating from mod_nss to mod_ssl]
Already migrated to mod_ssl
[Moving HTTPD service keytab to gssproxy]
[Removing self-signed CA]
[Removing Dogtag 9 CA]
[Checking for deprecated KDC configuration files]
[Checking for deprecated backups of Samba configuration files]
dnssec-validation yes
[Add missing CA DNS records]
IPA CA DNS records already processed
named user config '/etc/named/ipa-ext.conf' already exists
named user config '/etc/named/ipa-options-ext.conf' already exists
[Upgrading CA schema]
CA schema update complete (no changes)
[Verifying that CA audit signing cert has 2 year validity]
[Update certmonger certificate renewal configuration]
Certmonger certificate renewal configuration already up-to-date
[Enable PKIX certificate path discovery and validation]
PKIX already enabled
[Authorizing RA Agent to modify profiles]
[Authorizing RA Agent to manage lightweight CAs]
[Ensuring Lightweight CAs container exists in Dogtag database]
[Adding default OCSP URI configuration]
[Disabling cert publishing]
[Ensuring CA is using LDAPProfileSubsystem]
[Migrating certificate profiles to LDAP]
[Ensuring presence of included profiles]
[Add default CA ACL]
Default CA ACL already added
[Migrating to authselect profile]
Already migrated to authselect profile
[Create systemd-user hbac service and rule]
hbac service systemd-user already exists
[Setup SPAKE]
[Setup PKINIT]
[Enable server krb5.conf snippet]
[Adding ipa-ca alias to HTTP certificate]
Resubmitting HTTP cert tracking request
The IPA services were upgraded
The ipa-server-upgrade command was successful
```

Finalmente en caso sea necesario, reiniciar el servidor(Actualizaciones de Kernel)
```
# reboot
```

Volver a validar el versionamiento del IDM y el status de los servicios

```
# rpm -qa | grep ipa-server
ipa-server-common-4.8.7-12.module+el8.3.0+8222+c1bff54a.noarch
ipa-server-4.8.7-12.module+el8.3.0+8222+c1bff54a.x86_64
ipa-server-dns-4.8.7-12.module+el8.3.0+8222+c1bff54a.noarch
# ipactl status
Directory Service: RUNNING
krb5kdc Service: RUNNING
kadmin Service: RUNNING
named Service: RUNNING
httpd Service: RUNNING
ipa-custodia Service: RUNNING
pki-tomcatd Service: RUNNING
ipa-otpd Service: RUNNING
ipa-dnskeysyncd Service: RUNNING
ipa: INFO: The ipactl command was successful
```

<p><br><a href="idm">volver</a></p>




