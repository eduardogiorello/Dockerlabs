# 🔐 Writeup – Máquina **“Obsession”**

## 🔎 Reconocimiento Inicial

Realizamos un escaneo de puertos sobre la máquina objetivo:

```bash
sudo nmap -sS -sV 172.17.0.2
📌 Servicios encontrados
Puerto 21 – FTP

Puerto 22 – SSH

Puerto 80 – HTTP

🌐 Enumeración del Servicio Web
Al acceder al servicio web en el puerto 80, inspeccionamos el código fuente de la página.

Encontramos el siguiente comentario HTML:

<!-- Utilizando el mismo usuario para todos mis servicios, podré recordarlo fácilmente -->
Este comentario sugiere claramente que el mismo usuario es utilizado en todos los servicios.

👤 Identificación del usuario
El nombre de usuario aparece repetidamente:

En el título de la página (pestaña del navegador)

En links internos

En direcciones de correo visibles

El usuario identificado es:

russoski

🔐 Ataque de Fuerza Bruta (FTP y SSH)
Con el usuario russoski, realizamos ataques de fuerza bruta tanto contra FTP como SSH utilizando Hydra.

FTP
hydra -l russoski -P /usr/share/wordlists/rockyou.txt ftp://172.17.0.2 -V -I -t 4
SSH
hydra -l russoski -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -V -I -t 4
✅ Credenciales obtenidas
Las credenciales son las mismas para ambos servicios:

Usuario: russoski

Contraseña: iloveme

📂 Enumeración del Servicio FTP
Accedemos al servicio FTP con las credenciales obtenidas y encontramos varios archivos:

📷 Una imagen

🐍 Un archivo .py

📄 Un archivo README

🖼️ Análisis de la imagen
La imagen fue analizada con exiftool:

exiftool imagen.jpg
No se encontró información relevante.

🐍 Análisis de los archivos restantes
El archivo .py corresponde a un generador de contraseñas

El README explica su funcionamiento

➡️ Ninguno de estos archivos aporta información útil para la explotación.

🔑 Acceso por SSH
Accedemos al sistema mediante SSH:

ssh russoski@172..17.0.2
El usuario no es root, por lo que procedemos a verificar privilegios.

🚀 Escalada de Privilegios
Consultamos los permisos sudo del usuario:

sudo -l
El resultado indica:

(ALL) NOPASSWD: /usr/bin/vim
👉 El usuario puede ejecutar Vim como root sin contraseña.

🧠 Escalada usando GTFOBins
Buscamos el binario vim en GTFOBins y utilizamos el método indicado para obtener una shell como root:

sudo vi -c ':shell'
Verificamos privilegios:

sudo groups
Confirmando que ahora somos root.

🗂️ Resumen de Credenciales
Usuario: russoski

Contraseña: iloveme

Root: obtenido mediante abuso de sudo con vim

🧠 Conclusión
La máquina “Obsession” presenta una cadena clara de vulnerabilidades:

Uso del mismo usuario en todos los servicios

Pista explícita en comentarios HTML

Contraseña reutilizada en FTP y SSH

Configuración insegura de sudo permitiendo ejecutar vim como root sin contraseña

🔗 Ruta de explotación
Comentario HTML → Fuerza bruta → Acceso por SSH → Escalada con GTFOBins

✅ Máquina comprometida exitosamente.


---
