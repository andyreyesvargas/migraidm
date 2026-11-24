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

Ahora instalamos el producto de openscap en en client
```
[root@client ~]# yum install -y openscap openscap-scanner scap-security-guide unzip
```

<p><br><a href="idm.md">volver</a></p>

