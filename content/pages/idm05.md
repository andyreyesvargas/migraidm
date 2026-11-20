<h1>Definición de reglas de acceso</h1>

<p>
<strong>Meta:</strong>
<br>Administrar los permisos a los sistemas a través de reglas de acceso a recursos.
</p>
<p>
<strong>Objetivos:</strong>
<br>- Crear reglas de acceso
</p>
<p>
<strong>Secciones:</strong>
<br>- Gestión de permisos SUDO
</p>
<p>
<strong>Laboratorios:</strong>
<br><strong>- Crear reglas sudo</strong>
<br><strong>- Crear grupos de reglas sudo</strong>
</p>

<strong>Requisitos:</strong>
<br><strong>- Sistema IDM instalado</strong>

<h3><br><strong>## Crear reglas sudo</strong></h3>
En ambientes donde varios usuarios usan uno o más sistemas Linux, es necesario otorgar distintos permisos o privilegios para que estos puedan hacer uso de comandos propios del usuario administrador 'root'. Totalmente fuera de lugar e impensable es 'entregar' la contraseña de root para que los usuarios puedan hacer uso de los programas propios de sus funciones pero que son propiedad de 'root'. Por otro lado, hacer uso del comando su tampoco es práctico porque es lo mismo, necesitan la contraseña de root, asi que la mejor alternativa es hacer uso de sudo.

SUDO permite implementar un control de acceso altamente granulado de que usuarios ejecutan que comandos. Si un usuario normal desea ejecutar un comando de root (o de cualquier otro usuario), sudo verifica en su lista de permisos y si está permitido la ejecución de ese comando para ese usuario, entonces sudo se encarga de ejecutarlo. Es decir, sudo es un programa que basado en una lista de control (/etc/sudoers) permite (o no) la ejecución al usuario que lo invocó sobre un determinado programa propiedad de otro usuario, generalmente del administrador del sistema 'root'.

Para entender el concepto de SUDO con el usuario root creamos un usuario nuevo
<br>`# useradd andy `
<br>`# passwd andy`

Cambiamos al usuario nuevo y creamos algunos archivos
<br>`# su - andy`
<br>`# touch a b`

Ahora intentamos cambiar el usuario de un archivo
<br>`# chown root a`

Ahora intentamos cambiar el grupo de un archivo
<br>`# chgrp root b`

En ambos casos no podremos realizar las operaciones porque los comandos `chown` y `chgrp` son de acceso privilegia solo de `root`. Para permitir el acceso al nuevo usuario con el comando `visudo` creamos un grupo de comandos `ARCHIVOS` para que el usuario nuevo pueda usarlos
<br>`# su - root`
<br>`# visudo`
<br>`Cmnd_Alias ARCHIVOS = /usr/bin/chgrp, /usr/bin/chown`
<br>`andy ALL = ARCHIVOS
`

Ahora volvemos a intentar el cambio de usuario y grupo
<br>`# su - andy`
<br>`# sudo chown root a`
<br>`# sudo chgrp root b`

Esto resuelve el problema de permisos en el sistema, sin embargo es un proceso que debería realizarse en cada uno de los sistemas operativos de la empresa, en este aspecto IDM permite crear una base de datos centralizada de permisos el cual se replicara en cada uno de los sistema registrados.
El primer paso es crear los comandos SUDO en la plataforma de IDM con el inicio de sesion
<p align="center"><img src="../images/idm45.png?raw=true"></p>

Ingresamos a la sección de POLITICA > SUDO > Comandos SUDO
<p align="center"><img src="../images/idm46.png?raw=true"></p>

Seleccionamos el botón AGREGAR
<p align="center"><img src="../images/idm47.png?raw=true"></p>

Ingresamos los datos del comando chown
<p align="center"><img src="../images/idm48.png?raw=true"></p>

Ingresamos los datos del comando chgrp
<p align="center"><img src="../images/idm49.png?raw=true"></p>

De esta manera registramos todos los comandos que el administrador desea delegar, sin embargo multiples roles requieren multiples permisos, por ello estos comandos de archivos los agruparemos en un GRUPO DE COMANDOS SUDO para facilitar su gestión en la seccion POLITICAS > SUDO > GRUPO
<p align="center"><img src="../images/idm50.png?raw=true"></p>

Agregamos el grupo de comandos nuevo
<p align="center"><img src="../images/idm51.png?raw=true"></p>

Ingresamos los datos del grupo nuevo
<p align="center"><img src="../images/idm52.png?raw=true"></p>

Seleccionamos el grupo nuevo
<p align="center"><img src="../images/idm53.png?raw=true"></p>

Agregamos comandos al nuevo grupo
<p align="center"><img src="../images/idm54.png?raw=true"></p>

Seleccionamos los comandos disponibles en la izquierda y los trasladamos a la derecha
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm528.png?raw=true"></p>

Aceptamos los cambios
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm529.png?raw=true"></p>

Creamos un nuevo usuario en la plataforma IDM y lo agregamos a un grupo
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm530.png?raw=true"></p>

Ahora vamos a la sección POLITICAS > SUDO > REGLAS para crear nuestra primera regla de acceso
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm531.png?raw=true"></p>

Creamos una nueva Regla SUDO
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm532.png?raw=true"></p>

nombramos la regla
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm533.png?raw=true"></p>

Seleccionamos la regla creada
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm534.png?raw=true"></p>

La regla tiene varias opciones para su configuracion, por ahora solo modificaremos la directiva QUIEN y QUE. Seleccionamos quien puede ejecutar la regla
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm535.png?raw=true"></p>

Permitimos al grupo de quien puede ejecutar
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm536.png?raw=true"></p>

Ahora modificamos QUE puede hacer este grupo
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm537.png?raw=true"></p>

Seleccionamos el grupo de comandos
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm538.png?raw=true"></p>

En este punto la regla se encuentra activa, la validación de la regla se realizara cuando tenga un equipo cliente registrado

La regla ya fue creada, sin embargo tambien existe la necesidad de aumentar los comandos sudo en el grupo, para ello podemos apoyarnos en la linea de comandos para agregar comandos nuevos
<br>`# kinit admin@DOMAIN01.NET`
<br>`# ipa sudocmd-add /usr/bin/less --desc="Comando LESS"`
<br>`# ipa sudocmd-add /usr/bin/more --desc="Comando MORE"`
<br>`# ipa sudocmdgroup-add-member --sudocmds '/usr/bin/more' gestionarchivos`
<br>`# ipa sudocmdgroup-add-member --sudocmds '/usr/bin/less' gestionarchivos`
<br>`# ipa sudocmdgroup-show gestionarchivos`

<h3><br><strong>Laboratorio</strong></h3>
<br>Use el comando visudo y elimine las 2 lineas que creo en el primer laboratorio y replique la regla de acceso creada por el instructor
<br>Ejecute los siguientes comandos para obtener un reporte de comandos, usando el reporte cree una regla sudo que permita la ejecución de todo el grupo de comandos

```
# yum install -y mlocate zip
# updatedb
# locate /usr/bin/ | grep zip
```



<p><br><a href="idm">volver</a></p>
