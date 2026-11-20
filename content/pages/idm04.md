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
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm501.png?raw=true"></p>

Ingresamos a la sección de Servicios de Red y la configuración DNS
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm502.png?raw=true"></p>

Seleccionamos el único servidor DNS configurado
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm503.png?raw=true"></p>

Con el botón AGREGAR, ingresamos un nuevo registro de forwarder para la resolución de registros externos a nuestro dominio
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm504.png?raw=true"></p>

Finalizamos la configuración con el botón GUARDAR
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm505.png?raw=true"></p>

<h3><br><strong>Laboratorio</strong></h3>
Cree un forwarder de DNS al servidor 4.2.2.1


<h3><br><strong>## Creación de registros DNS</strong></h3>
Para crear registros nuevos en la zona DNS que se implemento durante el proceso de instalación ingresamos a la sección de Servicios de Red y Zonas DNS
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm506.png?raw=true"></p>

Seleccionamos la zona DNS creada en el proceso de instalación
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm507.png?raw=true"></p>

Le damos click al botón AGREGAR
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm508.png?raw=true"></p>

Ingresamos los datos del nuevo registro, es importante seleccionar el check de registro inverso para que se cree el registro de IP a Nombre adicional al de Nombre a IP
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm_upgrade/idm_a010.png?raw=true"></p>

Para validar la resolución del registro DNS ingresamos al servidor con el DNS server configurado, primero nos aseguramos que el archivo de configuración `/etc/sysconfig/network-scripts/ifcfg-ens***` tenga el DNS con nuestro servidor. Luego usamos el comando DIG o NSLOOKUP para validar la resolución
<br>`# dig +short satellite.domain01.net`
<br>`# nslookup satellite.domain01.net`

Este procedimiento puede realizarse tambien con la linea de comandos:
<br>`# kinit admin@DOMAIN01.NET`
<br>`# ipa dnsrecord-add domain01.net ftp --a-rec 192.168.100.203`
<br>`# ipa dnsrecord-add 100.168.192.in-addr.arpa. 203 --ptr-rec ftp.domain01.net.`
<br>`# dig +short ftp.domain01.net`
<br>`# nslookup ftp.domain01.net`

<h3><strong>Laboratorio</strong></h3>
<br>Con lo aprendido cree un registro DNS mail que redireccione a la IP 192.168.100.203

<h3><br><strong>## Configurar registros especiales</strong></h3>

Algunos registros DNS son de tipo especial, estos permiten publicar servicios especificos, en el siguiente ejemplo crearemos un registro de dominio de correo que redireccione los correos a un servidor SMTP

Le damos click al botón AGREGAR
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm508.png?raw=true"></p>

Ingresamos los datos del registro MX
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm510.png?raw=true"></p>

<h3><strong>Laboratorio</strong></h3>
<br>Cree su propio registro MX acorde al procedimiento mencionado

Para validar la resolución del registro de correo usamos
<br>`# nslookup`
<br>`# > set q=mx`
<br>`# > domain01.net`

También podemos usar
<br>`# dig domain01.net MX`

En un segundo terminal ejecutamos


<h3><br><strong>## Configurar zonas DNS</strong></h3>

El mismo servidor IDM puede configurar diferentes dominios, estos dominios se dividen en zonas y se recomienda ademas de crear la zona de resolución directa también la inversa, para ellos ingresamos a la sección de Servicios de Red y Zonas DNS
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm511.png?raw=true"></p>

Agregamos una zona nueva
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm512.png?raw=true"></p>

Registramos la zona directa nueva
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm513.png?raw=true"></p>

Agregamos una zona nueva
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm514.png?raw=true"></p>

Registramos la zona inversa nueva
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm515.png?raw=true"></p>

Ingresamos a la zona nueva
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm516.png?raw=true"></p>

Agregamos un nuevo registro DNS
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm517.png?raw=true"></p>

Ingresamos los datos del registro
<p align="center"><img src="https://github.com/workshopopennova/tecnologiasredhat/blob/master/images/idm518.png?raw=true"></p>

Validamos el nuevo registro
<br>`# dig +short client.example.com`
<br>`# nslookup client.example.com`

Podemos ejecutar el mismo procedimiento con la linea de comandos
<br>`# kinit admin@DOMAIN01.NET`
<br>`# ipa dnszone-add opennovalabs.net`
<br>`# ipa dnszone-add 1.16.172.in-addr.arpa.`
<br>`# ipa dnsrecord-add opennovalabs.net client --a-rec 172.16.1.181`
<br>`# ipa dnsrecord-add 1.16.172.in-addr.arpa. 181 --ptr-rec client.opennovalabs.net.`
<br>`# dig +short client.opennovalabs.net`


<h3><br><strong>Laboratorio 01</strong></h3>
Cree un forwarder de DNS al servidor 4.2.2.1

<h3><strong>Laboratorio 02</strong></h3>
<br>Con lo aprendido cree un registro DNS mail que redireccione a la IP 192.168.100.203

<h3><strong>Laboratorio 03</strong></h3>
<br>Cree su propio registro MX que apunte al registro mail y valide su funcionamiento

<h3><strong>Laboratorio 04</strong></h3>
<br>Usando lo aprendido cree un dominio minombre.net del rango de IPs 192.168.200.0/24, recuerde crear la zona de resolución directa y reversa ademas de registros del tipo A y PTR. Cree un registro A llamado mail el cual sera el mail exchanger de su dominio, valide el funcionamiento a nivel de DNS.

<p><br><a href="idm">volver</a></p>
