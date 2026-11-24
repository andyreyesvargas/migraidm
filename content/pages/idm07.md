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

Es recomendable definir políticas nuevas aplicadas a grupos de usuarios, usando lo aprendido en clase creamos un grupo operadores e ingresamos al usuario a este grupo.

En la sección de POLITICAS > Politicas de contraseñas agregamos una nueva politica
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm701.png?raw=true"></p>

Nombramos la política con una prioridad
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm702.png?raw=true"></p>

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

<h3><br><strong>Laboratorio</strong></h3>
<br>Cree la nueva política de contraseñas asociada al grupo operadores y registre su usuario personal al grupo operadores, valide la politica aplicada
<br>-

<h3><strong>## Establecer HBCA</strong></h3>

IDM permite gestionar acceso a servicios dentro de un equipo registrado, sin embargo por default todos los usuarios tienen acceso a todos los hosts, para ello vamos a desactivar el acceso deshabilitando las HBCA default

Ingresamos a la seccion POLITICAS>HBAC y seleccionamos la primera política
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm703.png?raw=true"></p>

Ejecutamos la acción de DESACTIVAR
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm704.png?raw=true"></p>

Repetimos el proceso hasta tener todas las HBAC desactivadas
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm705.png?raw=true"></p>

Antes de agregar cualquier regla, intentemos abrir un nuevo terminal para ingresar al cliente IDM, no deberia tener acceso porque desactivo el acceso default, ahora si intentemos agregar una nueva HBCA
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm705.png?raw=true"></p>

Nombramos la HBCA
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm706.png?raw=true"></p>

Ingresamos a la HBCA
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm707.png?raw=true"></p>

Agregamos en QUIEN el usuario que deseamos que tenga acceso
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm708.png?raw=true"></p>

En acceso agregamos el cliente IDM
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm709.png?raw=true"></p>

Finalmente agregamos los servicios permitidos, por ahora agreguemos todos los que estén disponibles
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm710.png?raw=true"></p>

Ahora intentemos volver a iniciar un terminal al cliente IDM

<h3><strong>Laboratorio</strong></h3>
<br>Cree una regla de acceso que permita a su usuario personal ingresar a su cliente IDM, valide su correcto uso
<br>-

<h3><strong>Laboratorio</strong></h3>
<br>Con lo aprendido en el workshop registre el equipo NODE 192.168.100.(30+X) en su plataforma IDM e instale el servicio vsftpd, recuerde que las reglas HBAC default deben estar desactivadas para efectos del laboratorio.
<br>El servicio vsftpd debera funcionar con los usuarios del IDM(haga las pruebas con su usuario personal) y solo debera permitir los servicios basicos necesarios para el vsftpd

<br>Instale el servicio VSFTPD en el server NODE
<br>
<br># yum install -y vsftpd
<br># systemctl start vsftpd
<br># systemctl enable vsftpd
<br># firewall-cmd --permanent --add-service=ftp
<br># firewall-cmd --reload
<br># su - andy.reyes
<br># touch file01
<br># touch file02
<br># touch file03
<br>

<br>Crear la regla de HBAC con la siguiente estructura
<br>Quien=usuario de prueba
<br>Donde=node.domain01.net
<br>Servicios=vsftpd


<br>Pruebas desde otro equipo
<br>
<br># yum install -y ftp
<br># ftp 192.168.100.(30+X)
<br>Ingresamos las credenciales del usuario de prueba
<br>Ejecutamos dir o ls

```
[root@client ~]# ftp 192.168.100.38
Connected to 192.168.100.38 (192.168.100.38).
220 (vsFTPd 3.0.3)
Name (192.168.100.38:root): andy.reyes
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
227 Entering Passive Mode (192,168,100,38,68,88).
150 Here comes the directory listing.
-rw-rw-r--    1 60001    60001           0 Mar 15 23:55 file01
-rw-rw-r--    1 60001    60001           0 Mar 15 23:55 file02
-rw-rw-r--    1 60001    60001           0 Mar 15 23:55 file03
226 Directory send OK.
ftp> exit
```

<p><br><a href="idm">volver</a></p>

