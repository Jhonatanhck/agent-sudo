
# Write-up: Agent Sudo (TryHackMe)

Empezamos como siempre, con un escaneo básico de nmap:

![Escaneo de Nmap](images/Pasted%20image%2020251126184952.png)

Vemos que hay 3 puertos abiertos:

* **21 FTP**
* **22 SSH**
* **80 HTTP**

![Puertos abiertos](images/Pasted%20image%2020251126185200.png)

Probamos si FTP tiene el usuario `anonymous` configurado, pero vemos que no. Entonces vamos por la otra vía (el servidor HTTP).

![Pagina web](images/Pasted%20image%2020251126185346.png)

Vemos esto en la página web: hay un mensaje para un tal **"user-agent"** de parte de un **"Agent R"**. Parece que R es uno de los nombres claves que usan los usuarios.

### Enumeración Web (User-Agent)

Intentemos hacer *Spoofing* del User-Agent para ver qué tal.

![Curl User Agent](images/Pasted%20image%2020251126185634.png)

Usamos `curl` y le indicamos con `-A` lo que queremos *Spoofear*, que en este caso es la **R**, y con `-L` seguimos cualquier redirección.

Vemos que R es un nombre de usuario pero no el que queremos. Poniéndonos en perspectiva de R, hay 25 empleados y 26 letras del alfabeto. Vamos a ver si así es que están organizados los nombres de los empleados.

Probamos con otras letras:

![Intento fallido 1](images/Pasted%20image%2020251126190045.png)

Nada.

![Intento fallido 2](images/Pasted%20image%2020251126190106.png)

Nada por igual.

![Encontramos a Chris](images/Pasted%20image%2020251126190129.png)

**¡ENCONTRAMOS ALGO!**

Con la letra **C**, vemos un nombre (**chris**) y podemos ver que le están diciendo algo sobre un trato, que le diga al **agente J**, y también que cambie su contraseña porque es muy débil.

### Fuerza Bruta (FTP)

¿Ahora qué podemos hacer con el nombre que tenemos? ¿Servirá para SSH o FTP?
Probemos primero un ataque de fuerza bruta para FTP.

![Hydra FTP](images/Pasted%20image%2020251126191237.png)

Tuvimos suerte, ya tenemos las credenciales para FTP. Vamos a loguearnos.

![Login FTP](images/Pasted%20image%2020251126191532.png)

Vemos estos 3 archivos, así que me los voy a llevar a mi máquina para poder verlos mejor.

![Descarga de archivos](images/Pasted%20image%2020251126191619.png)

El archivo que más me interesó fue la carta para el agente J. Vemos que dice que su contraseña está almacenada en la **"imagen falsa"** (*fake image*), eso es una pista.

### Esteganografía

Vamos a ver las imágenes primero.

**Esta es la primera imagen:**
![Imagen 1](images/Pasted%20image%2020251126191829.png)

**Esta es la segunda imagen:**
![Imagen 2](images/Pasted%20image%2020251126191905.png)

No veo nada raro a simple vista. Usaremos una herramienta para ver datos binarios llamada `binwalk`.

![Binwalk](images/Pasted%20image%2020251126192359.png)

Utilizamos `binwalk` en la imagen "fake" (porque ahí dice la carta que está la contraseña) y vemos que tiene un archivo **ZIP** dentro. Con el parámetro `-e` podemos extraer archivos de la imagen.

![Extraccion binwalk](images/Pasted%20image%2020251126192700.png)

Ya tenemos nuestro archivo extraído, vamos a ver qué tiene dentro.

![Archivos extraidos](images/Pasted%20image%2020251126192824.png)

Tenemos esos archivos. Voy a descomprimir primero el `.zip`.

![Unzip fallido](images/Pasted%20image%2020251126193440.png)

Cuando lo quiero descomprimir me pide una contraseña. Vamos a utilizar una herramienta muy buena para zips encriptados que es `zip2john`.

![Zip2John](images/Pasted%20image%2020251126193726.png)

Ya tenemos el hash para poder crackearlo con **John the Ripper** y así poder tener los archivos.

![Cracking con John](images/Pasted%20image%2020251126193822.png)

¡Y aquí tenemos la contraseña para el zip!

![Contenido del zip](images/Pasted%20image
