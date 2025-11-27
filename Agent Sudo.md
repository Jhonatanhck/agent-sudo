
# Write-up: Agent Sudo (TryHackMe)

Empezamos como siempre, con un escaneo básico de nmap:

![Escaneo de Nmap](Pasted%20image%2020251126184952.png)

Vemos que hay 3 puertos abiertos:

* **21 FTP**
* **22 SSH**
* **80 HTTP**

![Puertos abiertos](Pasted%20image%2020251126185200.png)

Probamos si FTP tiene el usuario `anonymous` configurado, pero vemos que no. Entonces vamos por la otra vía (el servidor HTTP).

![Pagina web](Pasted%20image%2020251126185346.png)

Vemos esto en la página web: hay un mensaje para un tal **"user-agent"** de parte de un **"Agent R"**. Parece que R es uno de los nombres claves que usan los usuarios.

### Enumeración Web (User-Agent)

Intentemos hacer *Spoofing* del User-Agent para ver qué tal.

![Curl User Agent](Pasted%20image%2020251126185634.png)

Usamos `curl` y le indicamos con `-A` lo que queremos *Spoofear*, que en este caso es la **R**, y con `-L` seguimos cualquier redirección.

Vemos que R es un nombre de usuario pero no el que queremos. Poniéndonos en perspectiva de R, hay 25 empleados y 26 letras del alfabeto. Vamos a ver si así es que están organizados los nombres de los empleados.

Probamos con otras letras:

![Intento fallido 1](Pasted%20image%2020251126190045.png)

Nada.

![Intento fallido 2](Pasted%20image%2020251126190106.png)

Nada por igual.

![Encontramos a Chris](Pasted%20image%2020251126190129.png)

**¡ENCONTRAMOS ALGO!**

Con la letra **C**, vemos un nombre (**chris**) y podemos ver que le están diciendo algo sobre un trato, que le diga al **agente J**, y también que cambie su contraseña porque es muy débil.

### Fuerza Bruta (FTP)

¿Ahora qué podemos hacer con el nombre que tenemos? ¿Servirá para SSH o FTP?
Probemos primero un ataque de fuerza bruta para FTP.

![Hydra FTP](Pasted%20image%2020251126191237.png)

Tuvimos suerte, ya tenemos las credenciales para FTP. Vamos a loguearnos.

![Login FTP](Pasted%20image%2020251126191532.png)

Vemos estos 3 archivos, así que me los voy a llevar a mi máquina para poder verlos mejor.

![Descarga de archivos](Pasted%20image%2020251126191619.png)

El archivo que más me interesó fue la carta para el agente J. Vemos que dice que su contraseña está almacenada en la **"imagen falsa"** (*fake image*), eso es una pista.

### Esteganografía

Vamos a ver las imágenes primero.

**Esta es la primera imagen:**
![Imagen 1](Pasted%20image%2020251126191829.png)

**Esta es la segunda imagen:**
![Imagen 2](Pasted%20image%2020251126191905.png)

No veo nada raro a simple vista. Usaremos una herramienta para ver datos binarios llamada `binwalk`.

![Binwalk](Pasted%20image%2020251126192359.png)

Utilizamos `binwalk` en la imagen "fake" (porque ahí dice la carta que está la contraseña) y vemos que tiene un archivo **ZIP** dentro. Con el parámetro `-e` podemos extraer archivos de la imagen.

![Extraccion binwalk](Pasted%20image%2020251126192700.png)

Ya tenemos nuestro archivo extraído, vamos a ver qué tiene dentro.

![Archivos extraidos](Pasted%20image%2020251126192824.png)

Tenemos esos archivos. Voy a descomprimir primero el `.zip`.

![Unzip fallido](Pasted%20image%2020251126193440.png)

Cuando lo quiero descomprimir me pide una contraseña. Vamos a utilizar una herramienta muy buena para zips encriptados que es `zip2john`.

![Zip2John](Pasted%20image%2020251126193726.png)

Ya tenemos el hash para poder crackearlo con **John the Ripper** y así poder tener los archivos.

![Cracking con John](Pasted%20image%2020251126193822.png)

¡Y aquí tenemos la contraseña para el zip!

![Contenido del zip](Pasted%20image%2020251126193917.png)

Aquí tenemos el archivo del zip que parece ser otra carta y dice que tienen que mandar una carta a `'QXJlYTUx'`. Parece ser un mensaje encodeado, así que voy a utilizar **CyberChef** para decodificar base64.

![Cyberchef](Pasted%20image%2020251126194307.png)

Vemos que el mensaje dice `Area51`.

Ya tenemos "Area51", ahora solo nos queda buscar en la otra imagen que tenemos. Hay una herramienta llamada `steghide` que básicamente se usa para guardar datos dentro de una imagen (y en la room de TryHackMe hay una pregunta sobre "steg password").

![Steghide](Pasted%20image%2020251126195052.png)

Efectivamente, con `Area51` podemos acceder a los archivos de la imagen y vemos un `message.txt`.

![Credenciales SSH](Pasted%20image%2020251126195235.png)

Con este comando podemos ver los datos. Vemos un nombre **"james"** y una contraseña **"hackerrules!"**. Supongo que estas son las credenciales para conectarse vía SSH.

### Acceso Inicial (SSH)

![Login SSH](Pasted%20image%2020251126195552.png)

Efectivamente, esas eran las credenciales. Ya estamos dentro de la máquina.

![User Flag](Pasted%20image%2020251126195754.png)
*(User Flag capturada)*

---

### Escalada de Privilegios

![Sudo -l](Pasted%20image%2020251126201048.png)

Investigando el sistema, encontramos una imagen interesante. Con `scp` descargamos la imagen que había en el usuario james a nuestra máquina local.

![Imagen alien](Pasted%20image%2020251126201126.png)

Esta es la imagen que aparece. Como en las preguntas de la room hay un apartado que dice **"reverse image search"**, traté de buscar información de la imagen en Google Images / Yandex.

![Busqueda inversa](Pasted%20image%2020251126202405.png)

Como las 2 primeras están en un idioma que no conozco, vamos a ver la de Fox News.

![Noticia Fox News](Pasted%20image%2020251126202523.png)

Aquí están todos los datos de la noticia y el nombre del incidente que parece ser **"Roswell alien autopsy"**.

Volviendo a la máquina, vemos con `sudo -l` que podemos ejecutar `/bin/bash` como cualquier usuario (ALL) excepto root... ¿o sí?

![Permisos sudo](Pasted%20image%2020251126204047.png)

Vemos que dice `(ALL, !root) /bin/bash`. Pero cuando queremos ejecutar una bash como root no nos deja, eso está raro. Entonces vamos a buscar un exploit.

![Version Sudo](Pasted%20image%2020251126204414.png)

Revisando la versión de Sudo y buscando CVEs, encontramos: **CVE-2019-14287**.

> En Sudo en versiones anteriores a 1.8.28, un atacante con acceso a una cuenta de Runas ALL sudoer puede omitir ciertas listas negras de políticas y módulos PAM de sesión... invocando sudo con un ID de usuario manipulado. Por ejemplo: `sudo -u #-1 /bin/bash`.

![Verificacion version](Pasted%20image%2020251126204715.png)

La versión es menor a la 1.8.28, así que podemos explotarla.

```bash
sudo -u#-1 /bin/bash


