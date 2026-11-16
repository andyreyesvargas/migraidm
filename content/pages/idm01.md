<h1>Introducción a Red Hat IDM</h1>

<p>
<strong>Meta:</strong>
<br>Entender el sistema Red Hat Identity Management.
</p>
<p>
<strong>Objetivos:</strong>
<br>- Conocer la solución Red Hat IDM
</p>

<h3><strong>Red Hat Identity Management</strong></h3>
Red Hat Identity Management (IdM) proporciona una forma centralizada y unificada de administrar almacenes de identidades, autenticación, políticas y políticas de autorización en un dominio basado en Linux. IdM reduce significativamente la sobrecarga administrativa de administrar diferentes servicios individualmente y usar diferentes herramientas en diferentes máquinas.

IdM es una de las pocas soluciones de software de identidad, políticas y autorización centralizadas que admite:
<br>- Funciones avanzadas de los entornos del sistema operativo Linux
<br>- Unificación de grandes grupos de máquinas Linux
<br>- Integración nativa con Active Directory


<h3><strong>Antes de Red Hat IDM</strong></h3>
<br><strong>Administrar identidades y políticas con varios servidores Linux</strong>
<br>Sin IdM: cada servidor se administra por separado. Todas las contraseñas se guardan en las máquinas locales. El administrador de TI administra a los usuarios en cada máquina, establece las políticas de autenticación y autorización por separado y mantiene las contraseñas locales.

<br><strong>Inicio de sesión único empresarial</strong>
<br>Sin IdM: los usuarios inician sesión en el sistema y se les solicita una contraseña cada vez que acceden a un servicio o aplicación. Estas contraseñas pueden ser diferentes y los usuarios deben recordar qué credencial usar para qué aplicación.

<br><strong>Administrar un entorno mixto de Linux y Windows</strong>
<br>Sin IdM: los sistemas Windows se administran en un bosque de Active Directory, pero los equipos de desarrollo, producción y otros tienen muchos sistemas Linux. Los sistemas Linux están excluidos del entorno de Active Directory.

<h3><strong>Dominio de gestión de identidades</strong></h3>
<br>El dominio de administración de identidad (IdM) consta de un grupo de máquinas que comparten la misma configuración, políticas y almacenes de identidad. Las propiedades compartidas permiten que las máquinas dentro del dominio se conozcan entre sí y funcionen juntas.
Desde la perspectiva de IdM, el dominio incluye los siguientes tipos de máquinas:
<br>- Servidores IdM, que funcionan como controladores de dominio.
<br>- Clientes de IdM, que están inscritos en los servidores.
<br>Los servidores IdM también son clientes IdM inscritos con ellos mismos: las máquinas servidor proporcionan la misma funcionalidad que los clientes.
<br>IdM admite máquinas Red Hat Enterprise Linux como servidores y clientes de IdM.

<h3><strong>Servicios alojados en los servidores IDM</strong></h3>

<p align="center"><img src="../images/idm01.png?raw=true"></p>

<br><strong>KDC de Kerberos</strong>
<br>IdM utiliza el protocolo Kerberos para admitir el inicio de sesión único. Con Kerberos, el usuario solo necesita presentar el nombre de usuario y la contraseña correctos una vez. Luego, el usuario puede acceder a los servicios de IdM sin que el sistema vuelva a solicitar las credenciales.

<br><strong>Servidor de directorio LDAP</strong>
<br>IdM incluye una instancia de servidor de directorio LDAP interno donde almacena toda la información de IdM, como información relacionada con Kerberos, cuentas de usuario, entradas de host, servicios, políticas, DNS y otros.

<br><strong>Autoridad certificadora</strong>
<br>En la mayoría de las implementaciones, se instala una autoridad de certificación (CA) integrada con el servidor IdM. También puede instalar el servidor sin la CA integrada si crea y proporciona todos los certificados necesarios de forma independiente.

<br><strong>Sistema de nombres de dominio (DNS)</strong>
<br>IdM utiliza DNS para el descubrimiento dinámico de servicios. La utilidad de instalación del cliente IdM puede utilizar información de DNS para configurar automáticamente la máquina cliente. Una vez que el cliente está inscrito en el dominio de IdM, utiliza DNS para ubicar los servidores y servicios de IdM dentro del dominio.

<br><strong>Protocolo de tiempo de red</strong>
<br>Muchos servicios requieren que los servidores y los clientes tengan la misma hora del sistema, dentro de una cierta variación. Por ejemplo, los tickets de Kerberos utilizan marcas de tiempo para determinar su validez y evitar ataques de reproducción. Si los tiempos entre el servidor y el cliente se desvían fuera del rango permitido, los tickets de Kerberos se invalidan.
<br>De forma predeterminada, IdM utiliza el Protocolo de hora de red (NTP) para sincronizar relojes en una red. Con NTP, un servidor central actúa como un reloj autorizado y los clientes sincronizan sus horas para que coincidan con el reloj del servidor. El servidor IdM se configura como servidor NTP para el dominio IdM durante el proceso de instalación del servidor.

<h3><strong>Clientes IDM</strong></h3>
<br>Los clientes IdM son máquinas configuradas para operar dentro del dominio IdM. Interactúan con los servidores IdM para acceder a los recursos del dominio. Por ejemplo, pertenecen a los dominios Kerberos configurados en los servidores, reciben certificados y tickets emitidos por los servidores y utilizan otros servicios centralizados para la autenticación y autorización.
<br>Un cliente de IdM no requiere software de cliente dedicado para interactuar como parte del dominio. Solo requiere la configuración adecuada del sistema de ciertos servicios y bibliotecas, como Kerberos o DNS. Esta configuración indica a la máquina cliente que utilice los servicios de IdM.

<p align="center"><img src="../images/idm02.png?raw=true"></p>

<br><strong>Daemon de servicios de seguridad del sistema</strong>
<br>System Security Services Daemon (SSSD) es una aplicación del lado del cliente para almacenar en caché las credenciales. Se recomienda usar SSSD en máquinas cliente porque simplifica la configuración requerida del cliente. SSSD también proporciona funciones adicionales, por ejemplo:
<br>- Autenticación de cliente sin conexión, garantizada mediante el almacenamiento en caché de las credenciales de las tiendas centralizadas de identidad y autenticación local
<br>- Consistencia mejorada del proceso de autenticación, porque no es necesario mantener una cuenta central y una cuenta de usuario local para la autenticación fuera de línea
<br>- Integración con otros servicios, como sudo
<br>- Autorización de control de acceso basado en host (HBAC)
<br>Con SSSD, los administradores de IdM pueden definir toda la configuración de identidad de forma centralizada en el servidor de IdM. El almacenamiento en caché permite que el sistema local continúe con las operaciones de autenticación normales si el servidor IdM deja de estar disponible o si el cliente se desconecta.
<br><strong>certmonger</strong>
<br>El certmongerservicio supervisa y renueva los certificados del cliente. Puede solicitar nuevos certificados para los servicios del sistema.
<br>


<h3><strong>Replicas IDM</strong></h3>
<br>Las réplicas son esencialmente clones de servidores de administración de identidad existentes y comparten una configuración de núcleo idéntica. El proceso de instalación de la réplica, entonces, tiene dos partes principales: copiar la configuración requerida del servidor existente y luego instalar la réplica en base a esa información.

<br>En el dominio IdM, hay tres tipos de máquinas:
<br>- Servidores, que administran todos los servicios utilizados por los miembros del dominio.
<br>- Réplicas, que son esencialmente copias de servidores (y, una vez copiadas, son idénticas a los servidores)
<br>- Los clientes, que pertenecen a los dominios Kerberos, reciben certificados y tickets emitidos por los servidores y utilizan otros servicios centralizados para la autenticación y autorización.

<br>Una réplica es un clon de un servidor IdM específico. El servidor y la réplica comparten la misma información interna sobre usuarios, máquinas, certificados y políticas configuradas. Estos datos se copian del servidor a la réplica en un proceso llamado réplica . Las dos instancias de Directory Server utilizadas por un servidor IdM, la instancia de Directory Server utilizada por el servidor IdM como almacén de datos y la instancia de Directory Server utilizada por Dogtag Certificate System para almacenar información del certificado, se replican en las correspondientes instancias de Directory Server del consumidor utilizadas. por la réplica de IdM. Las diferentes instancias de Directory Server se reconocen entre sí mediante acuerdos de replicación.. Se crea un acuerdo de replicación inicial entre un servidor maestro y una réplica cuando se crea la réplica; Se pueden agregar acuerdos adicionales a otros servidores o réplicas usando el <strong>ipa-replica-managecomando</strong>.

<p align="center"><img src="../images/idm03.png?raw=true"></p>

<br>Una de las topologías más resistentes es crear una configuración de celda para los servidores / réplicas, donde hay una pequeña cantidad de servidores en una celda que tienen acuerdos de replicación entre sí (una celda estrecha), y luego cada servidor tiene una replicación. acuerdo con otro servidor fuera de la celda, acoplando débilmente esa celda a todas las demás celdas del dominio general.

<p align="center"><img src="../images/idm04.png?raw=true"></p>

<br>Hay algunas recomendaciones sobre cómo lograr esto fácilmente:
<br>- Tenga al menos un servidor IdM en cada oficina principal, centro de datos o localidad. Preferiblemente, tenga dos servidores IdM.
<br>- No tenga más de cuatro servidores por centro de datos.
<br>- En lugar de utilizar un servidor o una réplica, las oficinas pequeñas pueden utilizar SSSD para almacenar en caché las credenciales y utilizar un servidor IdM externo como backend de datos.

<h3><strong>Integración con Active Directory</strong></h3>
<p align="center"><img src="../images/idm05.png?raw=true"></p>
<br>La sincronización es el proceso de copiar datos de usuario entre Active Directory y la administración de identidades.
La sincronización se define en un acuerdo entre un servidor IdM y un controlador de dominio de Active Directory. El acuerdo de sincronización define toda la información necesaria para identificar las entradas de usuario que se pueden sincronizar (como el subárbol para sincronizar y las clases de objetos requeridas en las entradas de usuario), así como definir cómo se manejan los atributos de la cuenta. Los acuerdos de sincronización se crean con valores predeterminados que se pueden modificar para satisfacer las necesidades de un dominio específico. Cuando dos servidores están involucrados en la sincronización, se denominan pares.
<br>La sincronización suele ser bidireccional. La información se envía y recibe entre el dominio de IdM y el dominio de Windows en un proceso que es muy similar a cómo los servidores de IdM y las réplicas comparten información entre sí. Es posible configurar la sincronización para sincronizar solo de una manera. Esa es la sincronización unidireccional.
<br>Para evitar el riesgo de conflictos de datos, la sincronización se configura entre un servidor de administración de identidad y un controlador de dominio de Active Directory. El servidor de administración de identidades propaga los cambios de regreso al dominio IdM, mientras que el controlador de dominio propaga los cambios nuevamente al dominio de Windows.
<br>Hay algunas características clave para la sincronización de IdM:

<br>- Una operación de sincronización se ejecuta cada cinco minutos.
<br>- La sincronización solo se puede configurar con un dominio de Active Directory. No se admiten varios dominios.
<br>- La sincronización solo se puede configurar con un controlador de dominio de Active Directory.
<br>- Solo se sincroniza la información del usuario.
<br>- Se pueden sincronizar tanto los atributos de usuario como las contraseñas.
<br>- Si bien las modificaciones son bidireccionales (van tanto de Active Directory a IdM como de IdM a Active Directory), la creación o adición de cuentas es solo unidireccional, desde Active Directory hasta Identity Management. Las nuevas cuentas creadas en Active Directory se sincronizan con IdM automáticamente. Sin embargo, las cuentas de usuario creadas en IdM también deben crearse en Active Directory antes de sincronizarse.
<br>- La información de bloqueo de la cuenta se sincroniza de forma predeterminada, por lo que una cuenta de usuario que está deshabilitada en un dominio está deshabilitada en el otro.
<br>- Los cambios de sincronización de contraseña entran en vigor inmediatamente.

<h3><strong>Directory Server</strong></h3>
<br>Red Hat Directory Server se compone de varios componentes. El núcleo del directorio en sí es el servidor que implementa el protocolo LDAP. Red Hat Directory Server tiene una interfaz gráfica de usuario del lado del cliente en la parte superior del servidor LDAP que permite a los usuarios finales buscar y cambiar entradas en el directorio. Otros clientes LDAP, tanto programas de terceros como programas personalizados escritos con Mozilla LDAP SDK y OpenLDAP SDK, se pueden usar con Red Hat Directory Server o para integrar otras aplicaciones con Red Hat Directory Server.
<p align="center"><img src="../images/idm06.png?raw=true"></p>




<p><br><a href="idm">volver</a></p>
