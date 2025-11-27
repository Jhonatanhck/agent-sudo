
Empezamos como siempre, con un escaneo basico de nmap 


![[Pasted image 20251126184952.png]]

Vemos que hay 3 puertos abiertos 

21 FPT
22 ssh
80 http

![[Pasted image 20251126185200.png]]

Probamos si ftp tiene el usuario anonymous configurado pero vemos que no, entonces vamos por la otra via (el servidor http)

![[Pasted image 20251126185346.png]]

Vemos esto en la pagina web, vemos que hay un usuario llamado user-agent y que nos lo dice un tal Agent R , parece que R es uno de los nombres claves que usan los usuarios 

Intentemos Spoofearlo para ver que tal


![[Pasted image 20251126185634.png]]

Usamos curl y le indicamos con -A lo que queremos Spoofear que en este caso es la R y con -L lo que hacemos es que seguimos cualquier redireccion 

Entonces vemos que R es un nombre de usuario pero no el que queremos, poniendonos en perspectiva de R hay 25 empleados y hay 26 letras del alfabeto, vamos a ver si asi es que estan organizados los nombres de los empleados 

![[Pasted image 20251126190045.png]]

Nada

![[Pasted image 20251126190106.png]]

Nada por igual

![[Pasted image 20251126190129.png]]

ENCONTRAMOS ALGO

vemos un nombre (chris) y podemos ver que le estan diciendo algo sobre un trato y que le diga al agente J y que tambien que cambie su contrasena porque es muy debil

Ahora que podemos hacer con el nombre que tenemos?

servira para ssh o ftp?

probemos primero un ataque de fuerza bruta para ftp 

![[Pasted image 20251126191237.png]]

tuvimos suerte, ya tenemos las credenciales para ftp 

vamos a logearnos 


![[Pasted image 20251126191532.png]]

vemos estos 3 archivos asi que me los voy a llevar a mi maquina para poder verlos mejor

![[Pasted image 20251126191619.png]]

el archivo que mas me intereso fue este, la carta para el agente J, vemos que dice que su contrasena esta almacenada en la imagen falsa, eso es una pista

vamos a ver las imagenes primero

Esta es la primera imagen
![[Pasted image 20251126191829.png]]


Esta es la segunda imagen
![[Pasted image 20251126191905.png]]

No veo nada de raro dentro de ellas 

hay una manera de ver datos binarios con una herramienta llamada binwalk

![[Pasted image 20251126192359.png]]

Utilizamos binwalk en la imagen "fake" porque es hay donde dice la carta que esta la contrasena y vemos que tiene un archivo zip que con el parametro -e podemos extraer archivos de la imagen 

![[Pasted image 20251126192700.png]]

ya tenemos nuestro archivo extraido, vamos a ver que tiene dentro

![[Pasted image 20251126192824.png]]

tenemos esos archivos, voy a descomprimir primero el .zip

![[Pasted image 20251126193440.png]]

Cuando lo quiero descomprimir me pide una contrasena porque sabemos que esta encriptado 
vamos a utilizar una herramienta muy buena para zip encriptados que es zip2john 

![[Pasted image 20251126193726.png]]

ya tenemos la hash para poder deshashearla con john y asi poder tener los archivos

![[Pasted image 20251126193822.png]]

y aqui tenemos la contrasena para el zip


![[Pasted image 20251126193917.png]]

aqui tenemos el archivo del zip que parece ser otra carta y dice que tienen que mandar una carta a 'QXJlYTUx' parece ser un mensaje encodeado asi que voy a utilizar una herramienta en linea que se llama cyberchef que basicamente desencodea los mensajes en base64

![[Pasted image 20251126194307.png]]

y vemos que el mensaje dice area51 ya decodificado en base64 con cyberchef

Ya tenemos area51 ahora solo nos queda buscar en la otra imagen que tenemos 
hay algo que se llama steghide que basicamente se usa para guardar datos dentro de una imagen
y como en la room de Tryhackme hay una pregunta que se llama steg password supongo que eso debe ser

![[Pasted image 20251126195052.png]]
efectivamente con Area51 podemos acceder a los archivos de la imagen 
y vemos un message.txt

![[Pasted image 20251126195235.png]]

con este comando podemos descarganos los datos 
vemos un nombre "james" y una contrasena "hackerrules!" otra vez la room de tryhackme dice ssh password asi que supongo que estas son las credenciales para conectarse via ssh

![[Pasted image 20251126195552.png]]

Y efectivamente esas eran las credenciales para ssh
ya estamos dentro de la maquina ahora va la parte de escalada de privilegios 

![[Pasted image 20251126195754.png]]
La flag
# Escalada de privilegios

![[Pasted image 20251126201048.png]]

con este comando descargamos la imagen que habia en el usuario james y ahora podemos verla

![[Pasted image 20251126201126.png]]

esta es la imagen que aparece 

y como en la preguntas de la room hay un apartado que dice reverse image search trate de buscar informacion de la imagen en esta pagina que ayuda a ver noticias de la imagen 

![[Pasted image 20251126202405.png]]

como las 2 primeras estan en un idioma que no conozco vamos a ver la de foxnews que seguro estan en ingles

![[Pasted image 20251126202523.png]]

y aqui estan todos los datos de la noticia y el nombre del incidente que parece ser "Roswell alien autopsy"

![[Pasted image 20251126204047.png]]

volviendo a la maquina vemos que puede ejecutar james como root 

![[Pasted image 20251126204203.png]]

pero vemos que cuando queremos ejecutar una bash como root no nos deja, eso esta raro

entonces vamos a buscar un exploit

![[Pasted image 20251126204414.png]]
veamos como podemos ejecutar este

segun [https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-14287](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-14287&ref=blog.qz.sg)

__En Sudo en versiones anteriores a 1.8.28, un atacante con acceso a una cuenta de Runas ALL sudoer puede omitir ciertas listas negras de políticas y módulos PAM de sesión, y puede causar un registro incorrecto, invocando sudo con un ID de usuario manipulado. Por ejemplo, esto permite la derivación de ! Configuración de raíz, y USER = registro, para un `sudo -u \#$((0xffffffff))`Comando.__


![[Pasted image 20251126204715.png]]

vamos, es menor a la 1.8.28, asi que podemos explotarla con el exploit que encontramos arriba

![[Pasted image 20251126204943.png]]

y listo ya somos root y encotramos la flag

PWNED













