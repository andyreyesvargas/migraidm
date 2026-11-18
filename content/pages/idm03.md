<h1>Creación de usuarios y políticas de usuario</h1>

<p>
<strong>Meta:</strong>
<br>Administrar los usuarios del sistema IDM.
</p>
<p>
<strong>Objetivos:</strong>
<br>- Crear usuarios
<br>- Crear grupos

</p>
<p>
<strong>Secciones:</strong>
<br>- Gestión de usuarios y grupos
</p>
<p>
<strong>Laboratorios:</strong>
<br><strong>- Creación de un usuario vía GUI</strong>
<br><strong>- Creación de un usuario vía CLI</strong>
<br><strong>- Creación de un grupo de usuarios vía GUI</strong>
<br><strong>- Creación de un grupo de usuarios vía CLI</strong>
<br><strong>- Gestión por Rest APIs</strong>


</p>

<strong>Requisitos:</strong>
<br><strong>- Sistema IDM instalado</strong>


<h3><br><strong>## Actualización de plataforma IDM</strong></h3>
En el caso la plataforma IDM tenga servicios caidos a pesar de que se reinicio el servicio, existe la posibilidad de BUGs que deben ser corregidos por actualizaciones, para ellos verificamos la plataforma

```
[root@server ~]# ipactl start
[root@server ~]# ipactl status
Directory Service: RUNNING
krb5kdc Service: RUNNING
kadmin Service: RUNNING
named Service: STOPPED
httpd Service: RUNNING
ipa-custodia Service: RUNNING
pki-tomcatd Service: RUNNING
ipa-otpd Service: RUNNING
ipa-dnskeysyncd Service: RUNNING
1 service(s) are not running
```

Como todo servidor IDM nuevo, maneja como servicio DNS su propia plataforma, si el servicio named(DNS) esta caído, entonces el server lo resolverá nombres hasta solucionar el problema, el primer paso es mapear repositorios de actualizaciones para instalar las correcciones
```
[root@server ~]# echo "192.168.100.128 ftp.migraciones.gob.pe" >> /etc/hosts
[root@server ~]# vi /etc/yum.repos.d/updates.repo
[baseos_updates]
name=baseos_updates
baseurl=ftp://ftp.migraciones.gob.pe/rhel98/BaseOS
enabled=1
gpgheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
[appstream_updates]
name=appstream_updates
baseurl=ftp://ftp.migraciones.gob.pe/rhel98/AppStream
enabled=1
gpgheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
```

Ahora se puede actualizar los RPMs de IDM, durante la actualización del rpm ipa-server-4.13.1-3.el9_8.2.x86_64 podria tardar mas de lo habitual porque valida el servicio con errores
```
[root@server ~]# yum update
Complete!
```

Se actualiza la configuración
```
[root@server ~]# ipa-server-upgrade
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
Certificate is OK; nothing to do
The IPA services were upgraded
The ipa-server-upgrade command was successful
```

Se reinicia el sistema
```
[root@server ~]# reboot
```

Se valida los servicios de IDM
```
[root@server ~]# ipactl status
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

Editar el archivo de configuracion
```
[root@server ~]# vi /etc/named/ipa-options-ext.conf
dnssec-validation no;
```

Reiniciar el servicio de ipactl
```
[root@server ~]# ipactl restart
[root@server ~]# ipactl status
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



<h3><br><strong>## Creación de un usuario vía GUI</strong></h3>

El proceso de creación de un usuario inicia con el logueo a la pagina de gestión `https://server.migraciones09.gob.pe`
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm401.png?raw=true"></p>


En la sección de USUARIOS seleccionamos el botón AGREGAR para crear un nuevo usuario
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm402.png?raw=true"></p>


Ingresamos los datos del nuevo usuario
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm403.png?raw=true"></p>

Verificamos que el usuario este creado
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm404.png?raw=true"></p>

Para sistemas compatibles con PAM, puede agregar la siguiente linea a los archivos `/etc/pam.d/system-auth` y `/etc/pam.d/password-auth`

```
session   required      pam_mkhomedir.so skel=/etc/skel umask=0002
```


<h3><br><strong>## Creación de un usuario vía CLI</strong></h3>

Desde el terminal Linux, podemos crear los usuarios del IDM, como primero paso debemos autenticarnos como administrador con el comando

`# kinit admin@MIGRACIONES09.GOB.PE`

Luego podemos crear un usuario con el domando

`# ipa user-add juan.reyes --first=Juan --last=Reyes`

En caso desee modificar algun parametro de creacion, como el shell, podemos usar

`# ipa user-mod --shel=/bin/bash juan.reyes`

Para setear la contraseña usamos

`# ipa passwd juan.reyes`

Otra opción de creación es la contraseña aleatoria con el comando

`# ipa user-add jaime.zamudio --first=Jaime --last=Zamudio --random`

Podemos borrar el usuario anterior con el comando
`# ipa user-del --no-preserve jaime.zamudio`

Notamos que al crear el usuario de nuevo, este no recupera su UID original, este UID  es diferente, podemos hacer algunas ediciones al usuario en el caso que el UID y GUID original sea 70002

`# ipa user-mod jaime.zamudio --shel=/bin/bash --uid=70002 --gid=70002`

En el servidor IDM podemos loguearnos con el usuario jaime.zamudio
```
login as: jaime.zamudio
Keyboard-interactive authentication prompts from server:
| Password:
End of keyboard-interactive prompts from server
Creating directory '/home/jaime.zamudio'.
[jaime.zamudio@idm ~]$ pwd
/home/jaime.zamudio
[jaime.zamudio@idm ~]$ echo $SHELL
/bin/bash
[jaime.zamudio@idm ~]$ id
uid=60002(jaime.zamudio) gid=6002 groups=6002 context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```

A fin de mejorar la seguridad, podemos activar el doble factor de autenticación con un token que complemente la contraseña local para formar una nueva contraseña de la forma <LOCAL><TOKEN>

<br><strong>Requisitos:</strong>
<br>En su celular instalar un aplicativo que generen tokens escaneando un QR (ejemplo FreeOTP)
<br>Habilitar con el usuario admin la doble autenticación de cada usuario
<br>Cada usuario deberá crear su propio token

Con el usuario admin, seleccionamos el usuario andy.reyes
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm_upgrade/idm_a001.png?raw=true"></p>

Activar la casilla de doble factor de autenticación y guardamos los cambios
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm_upgrade/idm_a002.png?raw=true"></p>

Iniciar sesión por GUI  con el usuario andy.reyes con la contraseña tradicional
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm_upgrade/idm_a003.png?raw=true"></p>

En la sección de FICHAS OTP, agregar una nueva
<<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm_upgrade/idm_a004.png?raw=true"></p>>

Creamos una nueva ficha basada en tiempo OTP
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm_upgrade/idm_a005.png?raw=true"></p>

El sistema generara un QR que escanearemos con el app FreeOTP
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm_upgrade/idm_a006.png?raw=true"></p>

En su app vera un código que se regenerara cada cierta cantidad de segundos, usara ese código para complementar su contraseña actual
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm_upgrade/idm_a007.png?raw=true"></p>

Desloguese del sistema y vuelva a ingresar usando ahora la contraseña doble con la forma de <CONTRASEÑA LOCAL><TOKEN>
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm_upgrade/idm_a008.png?raw=true"></p>

Puede validar su Ficha OTP
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm_upgrade/idm_a009.png?raw=true"></p>


<h3><br><strong>## Creación de un grupo de usuarios vía GUI</strong></h3>

El proceso de creación de un grupo y asignación de usuarios en ese grupo inicia con la creación de un usuario ingresando a la sección IDENTIDAD > GRUPOS y el botón de AGREGAR
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm405.png?raw=true"></p>

Ingresamos un nombre para el grupo y una descripcion
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm406.png?raw=true"></p>

Verificamos la creación del grupo y hacemos click sobre el nombre para editar sus integrantes
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm407.png?raw=true"></p>

Con el botón AGREGAR modificamos la lista de integrantes
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm408.png?raw=true"></p>

En la lista de usuarios, desplazamos algunos usuarios a la zona derecha para integrarlo al grupo
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm409.png?raw=true"></p>

Finalmente aceptamos los cambios
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm410.png?raw=true"></p>

<h3><br><strong>## Creación de un grupo de usuarios vía CLI</strong></h3>

El proceso de creación de un grupo se realiza con los siguientes comandos

`# ipa group-add redes --desc="Administradores de Red"`

Para agregar uno o mas usuarios al grupo usamos

`# ipa group-add-member redes --users=luis.fernandez --users=pedro.benitez`

Para remover uno o mas usuarios al grupo usamos

`# ipa group-remove-member redes --users=luis.fernandez --users=pedro.benitez`



<h3>Laboratorio 01: Usuarios</h3>

<br>Cree un usuario con la forma `<su nombre>.<su apellido>` y cambie su contraseña inicial por una contraseña permanente
<br>Con lo aprendido en esta sección y sus conocimientos de sistemas Linux, cree los siguientes usuarios
<br>Juan.Perez
<br>Pedro.Rodriguez
<br>Maria.Gomez
<br>Luis.Fernandez
<br>Andy.Lopez
<br>Jorge.Diaz
<br>Mario.Martinez
<br>Esteban.Perez
<br>Jose.Garcia
<br>Jorge.Sanchez
<br>Jaime.Romero
<br>Edgar.Sosa
<br>Julio.Torres
<br>Victor.Alvarez
<br>Paul.Ruiz
<br>Renzo.Ramirez
<br>Carlos.Flores
<br>Pedro.Benitez
<br>Rosa.Acosta
<br>

<br><strong>Opcional</strong>
<br>Al primer usuario creado (usuario personal), habilite el doble factor de autenticación

<strong>Tips de Linux:<strong> puede utilizar la siguiente estructura de Linux como apoyo
```
for A in $(cat users.txt)
do
nombre=$(echo $A | awk -F "." '{print $1}' )
apellido=$(echo $A | awk -F "." '{print $2}' )
echo $nombre $apellido
done
```
<br>`# sh script.sh`


<h3>Laboratorio 02: Grupos</h3>
<br>Cree 3 grupos de usuarios bajo las siguientes condiciones:
<br>- Grupo: Todos, Integrantes: Todos los usuarios del laboratorio 01
<br>- Grupo: Alpha, Integrantes: Todos los usuarios del laboratorio 01 cuyo nombre se encuentre en el rando de letras entre A y L
<br>- Grupo: Beta, Integrantes: Todos los usuarios del laboratorio 01 cuyo nombre se encuentre en el rando de letras entre M y Z

Tips de Linux: Apoyese en la directirva GREP para filtrar usuarios, `grep  '^[A-L]'`




<h3><br><strong>## Gestión por Rest APIs</strong></h3>

El instructor demostrara la creación de un usuario apoyándose en los REST APIs disponibles en la plataforma con el material disponible ejecutando el comando

`# git clone https://github.com/workshopopennova/examples.git`


<p><br><a href="idm.md">volver</a></p>


