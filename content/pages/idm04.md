<h1>Creación de registros en el DNS de IDM</h1>

<p>
<strong>Meta:</strong>
<br>Administrar el servicio DNS en el IDM.
</p>
<p>
<strong>Objetivos:</strong>
<br>- Crear registros DNS
<br>- Integrar la plataforma a un DNS externo
</p>
<p>
<strong>Secciones:</strong>
<br>- Configurar registros DNS
<br>- Configurar zonas DNS
</p>
<p>
<strong>Laboratorios:</strong>
<br><strong>- Configurar forwarders</strong>
<br><strong>- Creación de registros DNS</strong>
<br><strong>- Configurar registros especiales</strong>
<br><strong>- Configurar zonas DNS</strong>
</p>

<strong>Requisitos:</strong>
<br><strong>- Sistema IDM instalado</strong>

<h3><br><strong>## Configurar forwarders</strong></h3>

La resolución de DNS se realiza a través del servidor IDM pero solo dentro de nuestro dominio registrado, existen registros DNS de acceso externo publico que deben ser resueltos, en este caso se requiere enviar estas peticiones a un servidor externo, aquí es importante la configuración de forwarders, para iniciar este procedimiento nos logueamos en la plataforma
<p align="center"><img src="../images/idm26.png?raw=true"></p>

Ingresamos a la sección de Servicios de Red y la configuración DNS
<p align="center"><img src="../images/idm27.png?raw=true"></p>

Seleccionamos el único servidor DNS configurado
<p align="center"><img src="../images/idm28.png?raw=true"></p>

Con el botón AGREGAR, ingresamos un nuevo registro de forwarder para la resolución de registros externos a nuestro dominio
<p align="center"><img src="../images/idm29.png?raw=true"></p>

Finalizamos la configuración con el botón GUARDAR
<p align="center"><img src="../images/idm30.png?raw=true"></p>


<h3><br><strong>## Creación de registros DNS</strong></h3>
Para crear registros nuevos en la zona DNS que se implemento durante el proceso de instalación ingresamos a la sección de Servicios de Red y Zonas DNS
<p align="center"><img src="../images/idm31.png?raw=true"></p>

Seleccionamos la zona DNS creada en el proceso de instalación
<p align="center"><img src="../images/idm32.png?raw=true"></p>

Le damos click al botón AGREGAR
<p align="center"><img src="../images/idm33.png?raw=true"></p>

Ingresamos los datos del nuevo registro, es importante seleccionar el check de registro inverso para que se cree el registro de IP a Nombre adicional al de Nombre a IP
<p align="center"><img src="../images/idm34.png?raw=true"></p>

Para validar la resolución del registro DNS ingresamos al servidor con el DNS server configurado, primero nos aseguramos que el archivo de configuración `/etc/sysconfig/network-scripts/ifcfg-ens***` tenga el DNS con nuestro servidor. Luego usamos el comando DIG o NSLOOKUP para validar la resolución
<br>`# nslookup satellite.migraciones09.gob.pe`

Este procedimiento puede realizarse tambien con la linea de comandos:
<br>`# kinit admin@MIGRACIONES09.GOB.PE`
<br>`# ipa dnsrecord-add migraciones09.gob.pe ftp --a-rec 192.168.109.202`
<br>`# ipa dnsrecord-add 109.168.192.in-addr.arpa. 202 --ptr-rec ftp.migraciones09.gob.pe.`
<br>`# dig +short ftp.migraciones09.gob.pe`
<br>`# nslookup ftp.migraciones09.gob.pe`


<h3><br><strong>## Configurar registros especiales</strong></h3>

Algunos registros DNS son de tipo especial, estos permiten publicar servicios especificos, en el siguiente ejemplo crearemos un registro de dominio de correo que redireccione los correos a un servidor SMTP

Le damos click al botón AGREGAR
<p align="center"><img src="../images/idm35.png?raw=true"></p>

Ingresamos los datos del registro A llamado mail
<p align="center"><img src="../images/idm36.png?raw=true"></p>

Le damos click al botón AGREGAR
<p align="center"><img src="../images/idm35.png?raw=true"></p>

Ingresamos los datos del registro MX
<p align="center"><img src="../images/idm37.png?raw=true"></p>

Para validar la resolución del registro de correo usamos
<br>`# nslookup`
<br>`# > set q=mx`
<br>`# > migraciones09.gob.pe`

También podemos usar
<br>`# dig migraciones09.gob.pe MX`

Abrimos 2 terminales de server, en el primero instalamos postfix para envio de correos
```
[root@server ~]# yum install -y postfix
[root@server ~]# vi /etc/postfix/main.cf
mydomain = migraciones09.gob.pe
myorigin = $mydomain
inet_interfaces = all
#inet_interfaces = localhost
[root@server ~]# systemctl enable postfix --now
[root@server ~]# firewall-cmd --permanent --add-service=smtp
[root@server ~]# firewall-cmd --reload
[root@server ~]# yum install -y mutt
```

En el segundo terminal de server, abrimos el log en vivo del correo
```
[root@server ~]# tail -f /var/log/maillog
```

En el primer terminal enviamos un correo de prueba
```
[root@server ~]# echo "correo de prueba del alumno09" | mutt -s "correo de prueba" root@migraciones09.gob.pe
```

En el segundo terminal examinamos el log, vemos si se envio el correo
```
Nov 19 23:32:07 server postfix/qmgr[6143]: 480461000847: from=<>, size=2652, nrcpt=1 (queue active)
Nov 19 23:32:07 server postfix/qmgr[6143]: 051A41000841: removed
Nov 19 23:32:07 server postfix/smtp[6334]: 480461000847: to=<root@migraciones09.gob.pe>, relay=mail.migraciones09.gob.pe[192.168.100.128]:25, delay=0.06, delays=0/0/0.04/0.01, dsn=2.0.0, status=sent (250 2.0.0 Ok: queued as 3516248843)
Nov 19 23:32:07 server postfix/qmgr[6143]: 480461000847: removed
Nov 19 23:33:44 server postfix/pickup[6142]: C90F51000841: uid=0 from=<root>
Nov 19 23:33:44 server postfix/cleanup[6332]: C90F51000841: message-id=<av/NqN1q1OK/zqJp@server.migraciones09.gob.pe>
Nov 19 23:33:44 server postfix/qmgr[6143]: C90F51000841: from=<root@migraciones09.gob.pe>, size=453, nrcpt=1 (queue active)
Nov 19 23:33:44 server postfix/smtp[6334]: C90F51000841: to=<root@migraciones09.gob.pe>, relay=mail.migraciones09.gob.pe[192.168.100.128]:25, delay=0.03, delays=0/0/0.02/0, dsn=2.0.0, status=sent (250 2.0.0 Ok: queued as AE77548843)
```


<h3><br><strong>## Configurar zonas DNS</strong></h3>

El mismo servidor IDM puede configurar diferentes dominios, estos dominios se dividen en zonas y se recomienda ademas de crear la zona de resolución directa también la inversa, para ellos ingresamos a la sección de Servicios de Red y Zonas DNS
<p align="center"><img src="../images/idm38.png?raw=true"></p>

Agregamos una zona nueva
<p align="center"><img src="../images/idm39.png?raw=true"></p>

Registramos los datos de la zona directa example.com
<p align="center"><img src="../images/idm40.png?raw=true"></p>

Agregamos una zona nueva
<p align="center"><img src="../images/idm39.png?raw=true"></p>

Registramos los datos de la zona reversa 192.168.209.0/24
<p align="center"><img src="../images/idm41.png?raw=true"></p>

Ingresamos a la zona creada
<p align="center"><img src="../images/idm42.png?raw=true"></p>

Registramos un registro a la zona
<p align="center"><img src="../images/idm43.png?raw=true"></p>

Ingresamos un registro tipo A
<p align="center"><img src="../images/idm44.png?raw=true"></p>

Validamos el nuevo registro
<br>`# dig +short client.example.com`
<br>`# nslookup client.example.com`

Podemos ejecutar el mismo procedimiento con la linea de comandos
<br>`# kinit admin@MIGRACIONES09.GOB.PE`
<br>`# ipa dnszone-add migracioneslabs.net --skip-overlap-check`
<br>`# ipa dnszone-add 9.16.172.in-addr.arpa. --skip-overlap-check`
<br>`# ipa dnsrecord-add migracioneslabs.net client --a-rec 172.16.9.11`
<br>`# ipa dnsrecord-add 9.16.172.in-addr.arpa. 11 --ptr-rec client.migracioneslabs.net.`
<br>`# dig +short client.migracioneslabs.net`


<h3><br><strong>Laboratorio 01</strong></h3>
Cree un forwarder de DNS al servidor 4.2.2.1

<h3><strong>Laboratorio 02</strong></h3>
<br>Con lo aprendido cree un registro DNS mail que redireccione a la IP 192.168.100.128

<h3><strong>Laboratorio 03</strong></h3>
<br>Cree su propio registro MX que apunte al registro mail y valide su funcionamiento con el servicio de correo postix

<h3><strong>Laboratorio 04</strong></h3>
<br>Usando lo aprendido cree un dominio migracioneslabs.net con el direccionamientos 172.16.X.0/24 donde X es su numero alumno y cree al menos 3 registros, para www, ftp, mesadeayuda con resolución inersa a su criterio

<h3><strong>Laboratorio 05</strong></h3>
<br>Usando lo aprendido cree un dominio llamado migracioneswin.gob.pe y descargue el archivo ftp://ftp.migraciones.gob.pe/DNS_MIGRACIONES.csv que es un export de un dominio de active directory,y extraiga todos los registros A envíelos al dominio migracioneswin.gob.pe manteniendo las IPs originales

<p><br><a href="idm.md">volver</a></p>
