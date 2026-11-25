<h1>Lineamientos de aseguramiento</h1>

<p>
<strong>Meta:</strong>
<br>Establecer lineamientos de seguridad.
</p>
<p>
<strong>Objetivos:</strong>
<br>- Implementar políticas de contraseñas
<br>- Aplicar políticas de acceso al cliente IDM
</p>
<p>
<strong>Secciones:</strong>
<br>- Gestión de seguridad en IDM
</p>
<p>
<strong>Laboratorios:</strong>
<br><strong>- Establecer políticas de contraseñas</strong>
<br><strong>- Establecer HBCA</strong>
</p>

<strong>Requisitos:</strong>
<br><strong>- Servidor IDM</strong>
<br><strong>- Cliente IDM</strong><br>
</p>

<h3><strong>## Establecer políticas de contraseñas</strong></h3>
Una forma de implementar una política de contraseñas es a través de los comandos del sistema operativo, sin embargo, estas políticas solo se aplicarían a los usuarios locales

```
# useradd student
# passwd student
# chage -l student
Last password change                                    : Mar 25, 2023
Password expires                                        : never
Password inactive                                       : never
Account expires                                         : never
Minimum number of days between password change          : 0
Maximum number of days between password change          : 99999
Number of days of warning before password expires       : 7
```


Podemos modificar estos parámetros, apoyándonos con el comando

```
# chage --help
Usage: chage [options] LOGIN

Options:
  -d, --lastday LAST_DAY        set date of last password change to LAST_DAY
  -E, --expiredate EXPIRE_DATE  set account expiration date to EXPIRE_DATE
  -h, --help                    display this help message and exit
  -I, --inactive INACTIVE       set password inactive after expiration
                                to INACTIVE
  -l, --list                    show account aging information
  -m, --mindays MIN_DAYS        set minimum number of days before password
                                change to MIN_DAYS
  -M, --maxdays MAX_DAYS        set maximum number of days before password
                                change to MAX_DAYS
  -R, --root CHROOT_DIR         directory to chroot into
  -W, --warndays WARN_DAYS      set expiration warning days to WARN_DAYS
```

Se puede bloquear el usuario con el comando

```
# usermod -L student
```

Tambien desbloquear con el comando

```
# usermod -U student
```

Las politicas locales pueden ser modificadas en los siguientes archivos
```
# vi /etc/login.defs
PASS_MAX_DAYS 60
PASS_MIN_DAYS 1
PASS_WARN_AGE 7

# vi /etc/security/pwquality.conf
minlen = 12
dcredit = 1
ucredit = 1
lcredit = 1
ocredit = 1
```



A fin de asegurar la autenticación de los usuarios, se requieren definir políticas de asignación de contraseñas a la plataforma actualmente implementada, en caso la organización requiera una política diferente al default debe crearse una nueva politica

<br>Vida máxima (días): 90
<br>Vida mínima (horas): 1
<br>Tamaño del historial (number of passwords): 0
<br>Clases de caracteres: 0
<br>Longitud mínima: 8
<br>Número máximo de fallas: 6
<br>Falló reajuste de intervalo (seconds): 60
<br>Duración de bloqueo (seconds): 600

Si quisiera saber cual es la politica de contraseñas aplicadas a un usuario especifico, usamos el comando
`# ipa pwpolicy-show --user=andy.reyes`

Es recomendable definir políticas nuevas aplicadas a grupos de usuarios, usando lo aprendido en clase creamos un grupo para gestionar la nueva politica de contraseñas
<p align="center"><img src="../images/idm92.png?raw=true"></p>

En la sección de POLITICAS > Politicas de contraseñas agregamos una nueva politica
<p align="center"><img src="../images/idm93.png?raw=true"></p>

Nombramos la política con una prioridad
<p align="center"><img src="../images/idm94.png?raw=true"></p>

Ingresamos a la politica creada
<p align="center"><img src="../images/idm95.png?raw=true"></p>

Luego creamos una política de contraseñas mas exigente en relación a la política default
<br>Vida máxima (días): 45
<br>Vida mínima (horas): 1
<br>Tamaño del historial: 6
<br>Clases de caracteres: 1
<br>Longitud mínima: 9
<br>Número máximo de fallas: 3
<br>Falló reajuste de intervalo: 30
<br>Duración de bloqueo: 900

Creamos la política y la aplicamos al grupo operadores, luego volvemos a verificar el usuario
`# ipa pwpolicy-show --user=andy.reyes`

Ahora creamos otra política asociada a un nuevo grupo, pero con una mejor prioridad, incluimos a ese usuario en el grupo y volvemos a ejecutar el comando
`# ipa pwpolicy-show --user=andy.reyes`


<h3><strong>## Reportes de OpenSCAP</strong></h3>
Los reportes de openscap nos permiten facilitar el proceso de gestion de lineamientos de seguridad y recomendaciones de entidades internacionales, en el caso de RHEL9, iniciamos implementando un servidor web o ftp para alojar los reportes del client

```
[root@client ~]# yum install -y httpd
[root@client ~]# systemctl enable httpd --now
[root@client ~]# firewall-cmd --permanent --add-service=http
[root@client ~]# firewall-cmd --permanent --add-service=https
[root@client ~]# firewall-cmd --reload
```

Ahora instalamos el producto de openscap en en client
```
[root@client ~]# yum install -y openscap openscap-scanner scap-security-guide unzip
```

Podemos examinar los perfiles para RHEL9 disponible en el archivo descargado desde la suscripcion de RHEL
```
[root@client ~]# oscap info /usr/share/xml/scap/ssg/content/ssg-rhel9-ds.xml
Document type: Source Data Stream
Imported: 2024-02-14T06:42:50

Stream: scap_org.open-scap_datastream_from_xccdf_ssg-rhel9-xccdf.xml
Generated: (null)
Version: 1.3
Checklists:
        Ref-Id: scap_org.open-scap_cref_ssg-rhel9-xccdf.xml
WARNING: Datastream component 'scap_org.open-scap_cref_security-data-oval-v2-RHEL9-rhel-9.oval.xml.bz2' points out to the remote 'https://access.redhat.com/security/data/oval/v2/RHEL9/rhel-9.oval.xml.bz2'. Use '--fetch-remote-resources' option to download it.
WARNING: Skipping 'https://access.redhat.com/security/data/oval/v2/RHEL9/rhel-9.oval.xml.bz2' file which is referenced from datastream
                Status: draft
                Generated: 2024-02-14
                Resolved: true
                Profiles:
                        Title: ANSSI-BP-028 (enhanced)
                                Id: xccdf_org.ssgproject.content_profile_anssi_bp28_enhanced
                        Title: ANSSI-BP-028 (high)
                                Id: xccdf_org.ssgproject.content_profile_anssi_bp28_high
                        Title: ANSSI-BP-028 (intermediary)
                                Id: xccdf_org.ssgproject.content_profile_anssi_bp28_intermediary
                        Title: ANSSI-BP-028 (minimal)
                                Id: xccdf_org.ssgproject.content_profile_anssi_bp28_minimal
                        Title: CCN Red Hat Enterprise Linux 9 - Advanced
                                Id: xccdf_org.ssgproject.content_profile_ccn_advanced
                        Title: CCN Red Hat Enterprise Linux 9 - Basic
                                Id: xccdf_org.ssgproject.content_profile_ccn_basic
                        Title: CCN Red Hat Enterprise Linux 9 - Intermediate
                                Id: xccdf_org.ssgproject.content_profile_ccn_intermediate
                        Title: CIS Red Hat Enterprise Linux 9 Benchmark for Level 2 - Server
                                Id: xccdf_org.ssgproject.content_profile_cis
                        Title: CIS Red Hat Enterprise Linux 9 Benchmark for Level 1 - Server
                                Id: xccdf_org.ssgproject.content_profile_cis_server_l1
                        Title: CIS Red Hat Enterprise Linux 9 Benchmark for Level 1 - Workstation
                                Id: xccdf_org.ssgproject.content_profile_cis_workstation_l1
                        Title: CIS Red Hat Enterprise Linux 9 Benchmark for Level 2 - Workstation
                                Id: xccdf_org.ssgproject.content_profile_cis_workstation_l2
                        Title: DRAFT - Unclassified Information in Non-federal Information Systems and Organizations (NIST 800-171)
                                Id: xccdf_org.ssgproject.content_profile_cui
                        Title: Australian Cyber Security Centre (ACSC) Essential Eight
                                Id: xccdf_org.ssgproject.content_profile_e8
                        Title: Health Insurance Portability and Accountability Act (HIPAA)
                                Id: xccdf_org.ssgproject.content_profile_hipaa
                        Title: Australian Cyber Security Centre (ACSC) ISM Official
                                Id: xccdf_org.ssgproject.content_profile_ism_o
                        Title: Protection Profile for General Purpose Operating Systems
                                Id: xccdf_org.ssgproject.content_profile_ospp
                        Title: PCI-DSS v4.0 Control Baseline for Red Hat Enterprise Linux 9
                                Id: xccdf_org.ssgproject.content_profile_pci-dss
                        Title: DISA STIG for Red Hat Enterprise Linux 9
                                Id: xccdf_org.ssgproject.content_profile_stig
                        Title: DISA STIG with GUI for Red Hat Enterprise Linux 9
                                Id: xccdf_org.ssgproject.content_profile_stig_gui
                Referenced check files:
                        ssg-rhel9-oval.xml
                                system: http://oval.mitre.org/XMLSchema/oval-definitions-5
                        ssg-rhel9-ocil.xml
                                system: http://scap.nist.gov/schema/ocil/2
                        security-data-oval-v2-RHEL9-rhel-9.oval.xml.bz2
                                system: http://oval.mitre.org/XMLSchema/oval-definitions-5
Checks:
        Ref-Id: scap_org.open-scap_cref_ssg-rhel9-oval.xml
        Ref-Id: scap_org.open-scap_cref_ssg-rhel9-ocil.xml
        Ref-Id: scap_org.open-scap_cref_ssg-rhel9-cpe-oval.xml
        Ref-Id: scap_org.open-scap_cref_security-data-oval-v2-RHEL9-rhel-9.oval.xml.bz2
Dictionaries:
        Ref-Id: scap_org.open-scap_cref_ssg-rhel9-cpe-dictionary.xml
```

Creamos un reporte de openscap del CIS Server level 2
```
[root@client ~]# oscap xccdf eval --fetch-remote-resources --oval-results \
--profile xccdf_org.ssgproject.content_profile_cis \
--report /var/www/html/$(hostname).$(date '+%Y-%m-%d_%H-%M-%S').html \
--results /var/www/html/$(hostname).$(date '+%Y-%m-%d_%H-%M-%S').xml \
/usr/share/xml/scap/ssg/content/ssg-rhel9-ds.xml
[root@client ~]# ll /var/www/html/
```

De la misma manera podemos crear un reporte ahora asociado al estandard PCI y lo generamos en la misma ruta web
```
[root@client ~]# oscap xccdf eval --fetch-remote-resources --oval-results \
--profile xccdf_org.ssgproject.content_profile_pci-dss \
--report /var/www/html/$(hostname).$(date '+%Y-%m-%d_%H-%M-%S').html \
--results /var/www/html/$(hostname).$(date '+%Y-%m-%d_%H-%M-%S').xml \
/usr/share/xml/scap/ssg/content/ssg-rhel9-ds.xml
[root@client ~]# ll /var/www/html/
```

Podemos actorar el analisis a lineamientos especificos, por ejemplo, descargamos del ftp del salon el archivo de tailoring que permite escanear solo los lineamientos de AIDE y AUDIT
```
[root@client ~]# yum install -y wget
[root@client ~]# wget ftp://192.168.100.128/openscap/ssg-rhel9-ds-aide-tailoring.xml
```

Ahora creamos el reporte en la misma ruta web, referenciando el tailoring y al perfil de ese tailoring
```
[root@client ~]# oscap info ssg-rhel9-ds-aide-tailoring.xml
Document type: XCCDF Tailoring
Imported: 2026-11-24T18:21:23
Benchmark Hint: /tmp/scap-workbench-FUJUih/ssg-rhel9-ds.xml
Profiles:
        Title: CIS Red Hat Enterprise Linux 9 Benchmark for Level 2 - Server [CUSTOMIZED]
                Id: xccdf_org.ssgproject.content_profile_cis_customized

[root@client ~]# oscap xccdf eval --fetch-remote-resources --oval-results \
--profile xccdf_org.ssgproject.content_profile_cis_customized \
--report /var/www/html/$(hostname).$(date '+%Y-%m-%d_%H-%M-%S').html \
--results /var/www/html/$(hostname).$(date '+%Y-%m-%d_%H-%M-%S').xml \
--tailoring-file ssg-rhel9-ds-aide-tailoring.xml \
/usr/share/xml/scap/ssg/content/ssg-rhel9-ds.xml
[root@client ~]# ll /var/www/html/
```

Con el reporte inicial customizado, descargamos un script en ansible que pueda resolver la mayoria de las observaciones
```
[root@client ~]# wget ftp://192.168.100.128/openscap/ssg-rhel9-ds-aide-tailoring.yaml
```

Instalamos el componente de ansible y ejecutamos el script
```
[root@client ~]# yum install -y ansible-core
[root@client ~]# ansible-playbook -c local -i localhost, ssg-rhel9-ds-aide-tailoring.yaml
```

Volvemos a generar un nuevo reporte customizado y verificamos el reporte
```
[root@client ~]# oscap xccdf eval --fetch-remote-resources --oval-results \
--profile xccdf_org.ssgproject.content_profile_cis_customized \
--report /var/www/html/$(hostname).$(date '+%Y-%m-%d_%H-%M-%S').html \
--results /var/www/html/$(hostname).$(date '+%Y-%m-%d_%H-%M-%S').xml \
--tailoring-file ssg-rhel9-ds-aide-tailoring.xml \
/usr/share/xml/scap/ssg/content/ssg-rhel9-ds.xml
[root@client ~]# ll /var/www/html/
```

<h3><strong>## Analisis de logs de IDM</strong></h3>
El sistema creara dentro de la ruta de logs /var/log/dirsrv/ una carpeta con el nombre de la instancia de IDM, dentro de ella se vera los principales logs del sistema, el instructor hara demostraciones de como se va poblando el log con acciones en el idm

```
[root@server ~]# cd /var/log/dirsrv/
[root@server dirsrv]# ll
total 4
drwxrwx---. 2 dirsrv dirsrv 4096 Nov 25 13:49 slapd-MIGRACIONES09-GOB-PE
[root@server dirsrv]# cd slapd-MIGRACIONES09-GOB-PE
[root@server slapd-MIGRACIONES09-GOB-PE]# ll
total 24648
-rw-------. 1 dirsrv dirsrv  689435 Nov 25 14:59 access
-rw-------. 1 dirsrv dirsrv 4846060 Nov 18 22:16 access.20261118-002616
-rw-------. 1 dirsrv dirsrv 3864775 Nov 21 13:41 access.20261120-134643
-rw-------. 1 dirsrv dirsrv 2472377 Nov 22 13:41 access.20261121-134644
-rw-------. 1 dirsrv dirsrv 1125333 Nov 23 13:44 access.20261122-134645
-rw-------. 1 dirsrv dirsrv 6958524 Nov 24 13:44 access.20261123-134946
-rw-------. 1 dirsrv dirsrv 4789843 Nov 25 13:44 access.20261124-134947
-rw-------. 1 dirsrv dirsrv     753 Nov 25 13:49 access.rotationinfo
-rw-------. 1 dirsrv dirsrv       0 Nov 18 00:26 audit
-rw-------. 1 dirsrv dirsrv      63 Nov 18 00:26 audit.rotationinfo
-rw-------. 1 dirsrv dirsrv  276005 Nov 23 12:09 errors
-rw-------. 1 dirsrv dirsrv      63 Nov 18 00:26 errors.rotationinfo
-rw-------. 1 dirsrv dirsrv  175295 Nov 23 20:44 security
-rw-------. 1 dirsrv dirsrv      63 Nov 18 00:26 security.rotationinfo
```

<h3><strong>## Asegurando aplicaciones con SELinux</strong></h3>
La seguridad de las aplicaciones puede ser mejorara usando SELinux como una capa mas de seguridad que evite intrusión o accesos no autorizados, incluso errores de configuración, para este ejemplo vamos a considerar una aplicacion nueva basa en apache/httpd

La aplicacion basada en apache tiene la particularidad que debera implementarse sobre la carpeta reports y sus reportes seran accedidos via web

En el client como ya tenemos un apache, vamos a migrar los reportes de openscap a la nueva ruta
```
[root@client ~]# mkdir /reports
[root@client ~]# vi /etc/httpd/conf/httpd.conf
DocumentRoot "/reports"
<Directory "/reports">
    Options Indexes FollowSymLinks
    AllowOverride None
    Require all granted
</Directory>
[root@client ~]# systemctl restart httpd
```

vamos a copiar algunos reportes de /var/www/html a /reports
```
[root@client ~]# cp /var/www/html/*.html /reports/
[root@client ~]# ll /reports/
total 145284
-rw-r--r--. 1 root root  7470896 Nov 25 15:52 client.migraciones09.gob.pe.2026-11-24_17-02-29.html
-rw-r--r--. 1 root root 70144466 Nov 25 15:52 client.migraciones09.gob.pe.2026-11-24_17-05-15.html
-rw-r--r--. 1 root root 70144466 Nov 25 15:52 client.migraciones09.gob.pe.2026-11-24_18-18-12.html
-rw-r--r--. 1 root root   510903 Nov 25 15:52 client.migraciones09.gob.pe.2026-11-24_18-25-34.html
-rw-r--r--. 1 root root   490050 Nov 25 15:52 client.migraciones09.gob.pe.2026-11-25_13-14-45.html
```

Intentamos ver por navegador un reporte antiguo
<p align="center"><img src="../images/idm96.png?raw=true"></p>

Como /reports no es parte de una estructura de SELinux, el sistema previene el acceso, si desactivamos temporalmente el SELinux, volver a hacer la prueba
```
[root@client ~]# setenforce 0
```
<p align="center"><img src="../images/idm97.png?raw=true"></p>

Como el objetivo no es desactivar SELinux sino configurarlo para que trabaje con la nueva ruta de la aplicacion, lo reactivamos
```
[root@client ~]# setenforce 1
```

Ejecutamos el comando que nos permite ver como estan etiquetados los archivos desde la perspectiva de SELinux
```
[root@client ~]# ls -lZ /var/www/html/
```

Vemos que los reportes antiguos estan etiquetas bajo el contexto httpd_sys_content_t, ahora verificamos en /reports
```
[root@client ~]# ls -lZ /reports/
```

Vemos que el reports el contexto aplicado es default_t, por eso no se permite el acceso, el servicio http necesita el contexto httpd_sys_content_t en sus archivos, la mayoria de los contextos como default_t se consideran inseguros, modificamos la carpeta /reports para que todo lo nuevo creada en ella tenga httpd_sys_content_t
```
[root@client ~]# semanage fcontext -a -t httpd_sys_content_t '/reports(/.*)?'
[root@client ~]# semanage fcontext -l | grep reports
```

Aplicamos el nuevo contexto a la carpeta
```
[root@client ~]# restorecon -Rv /reports/
[root@client ~]# touch /reports/test.html
[root@client ~]# ls -lZ /reports/
total 145284
-rw-r--r--. 1 root root unconfined_u:object_r:httpd_sys_content_t:s0  7470896 Nov 25 15:52 client.migraciones09.gob.pe.2026-11-24_17-02-29.html
-rw-r--r--. 1 root root unconfined_u:object_r:httpd_sys_content_t:s0 70144466 Nov 25 15:52 client.migraciones09.gob.pe.2026-11-24_17-05-15.html
-rw-r--r--. 1 root root unconfined_u:object_r:httpd_sys_content_t:s0 70144466 Nov 25 15:52 client.migraciones09.gob.pe.2026-11-24_18-18-12.html
-rw-r--r--. 1 root root unconfined_u:object_r:httpd_sys_content_t:s0   510903 Nov 25 15:52 client.migraciones09.gob.pe.2026-11-24_18-25-34.html
-rw-r--r--. 1 root root unconfined_u:object_r:httpd_sys_content_t:s0   490050 Nov 25 15:52 client.migraciones09.gob.pe.2026-11-25_13-14-45.html
-rw-r--r--. 1 root root unconfined_u:object_r:httpd_sys_content_t:s0        0 Nov 25 16:04 test.html
```

Volvemos a revisar el reporte y deberíamos recuperar el acceso
<p align="center"><img src="../images/idm97.png?raw=true"></p>

<h3><strong>## Uso del MOTD</strong></h3>
El sustento principal para incluir un aviso legal en el archivo /etc/motd (o en banners previos al inicio de sesión) radica en eliminar la expectativa de privacidad del usuario, demostrar el consentimiento de monitoreo y acreditar la intención delictiva en caso de un acceso no autorizado. Los sustentos juridicos principales son los siguientes:

<br> - Eliminación de la expectativa de privacidad. Al advertir explícitamente que el sistema es de uso exclusivo para personal autorizado y que toda actividad es monitoreada y registrada, se evita que un infractor (interno o externo) alegue violación a su intimidad o secreto de las comunicaciones durante una auditoría forense o juicio.
<br> - Acreditación del dolo (intención delictiva). En legislaciones penales sobre ciberdelincuencia —como la Ley N.º 30096 (Ley de Delitos Informáticos en Perú) o el Convenio de Budapest— el delito de acceso ilícito requiere que el agente actúe de forma deliberada e ilegítima. Un aviso visible prueba que el usuario fue advertido de la restricción y decidió continuar.
<br> - Consentimiento informado. Si un empleado o proveedor accede al servidor tras leer las políticas de uso y monitoreo, se configura una aceptación tácita de las condiciones de supervisión de la empresa.

```
[root@client ~]# vi /etc/motd
================================================================================
                        ADVERTENCIA DE SEGURIDAD Y USO
================================================================================
Este sistema es propiedad privada de [NOMBRE DE LA EMPRESA / ENTIDAD]. 
El acceso y uso de este servidor está restringido exclusivamente a personal 
autorizado y para fines estrictamente laborales u operativos.

AVISO LEGAL Y MONITOREO (PERÚ):
1. Todo acceso, comando ejecutado, transferencia de archivos y actividad en 
   este sistema es monitoreado, auditado y registrado de forma continua.
2. Al continuar con esta sesión, usted reconoce que no existe expectativa de 
   privacidad sobre las operaciones realizadas en este entorno y otorga su 
   consentimiento expreso para la supervisión y auditoría de sus acciones, 
   conforme a las políticas internas de seguridad de la información.
3. El acceso no autorizado, la excedencia de privilegios, la alteración de 
   datos o cualquier uso indebido constituyen delitos sancionados penalmente 
   bajo la Ley N.º 30096 (Ley de Delitos Informáticos del Perú) y normas 
   conexas, además de acarrear sanciones civiles o laborales.
4. Si este sistema aloja información personal, su tratamiento está sujeto 
   a estricto deber de confidencialidad bajo la Ley N.º 29733 (Ley de 
   Protección de Datos Personales).

Si usted no cuenta con autorización expresa, desconéctese inmediatamente.
================================================================================
```






<p><br><a href="idm.md">volver</a></p>

