# 🧪 Writeup – Compromiso de Máquina Linux (SSH + Metadata)

## 📌 Información general

* **Tipo:** Linux
* **Dificultad:** Muy fácil
* **Vector principal:** Credenciales + escalada por sudo
* **Servicios expuestos:** SSH (22), HTTP (80)

---

## 🔍 Enumeración inicial

### 1️⃣ Escaneo de puertos y servicios

Se comenzó con un escaneo para identificar puertos abiertos y servicios en ejecución:

```bash
nmap -sS -sV -p- 172.17.0.2
```

**Resultado relevante:**

* **22/tcp** → SSH
* **80/tcp** → HTTP

---

## 🌐 Enumeración web

Al acceder al servicio web desde el navegador:

```text
http://172.17.0.2
```

La página solo mostraba **una imagen**, sin formularios ni contenido interactivo visible.
Esto indicó que el vector web no era directo, pero la imagen podía contener información oculta.
Intente usar gobuster sin resultados
---

## 🔐 Ataque al servicio SSH

### 2️⃣ Intento de fuerza bruta con Hydra

Dado que el puerto 22 estaba abierto, se intentó un ataque de fuerza bruta contra SSH usando el usuario `root`:

```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt.gz ssh://172.17.0.2 -t 4
```

❌ **Resultado:**
No se logró acceso. El usuario `root` no tenía credenciales débiles o no era válido para autenticación directa.

---

## 🖼️ Análisis de metadatos (punto clave)

Luego de revisar otros writeups y analizar el comportamiento del sitio, se decidió inspeccionar la imagen encontrada en la web.

### 3️⃣ Extracción de metadatos con ExifTool

Se descargó la imagen y se analizaron sus metadatos:

```bash
exiftool imagen.jpg
```

📌 **Hallazgo importante:**

* Se encontró un **nombre de usuario** oculto en los metadatos.
* No se obtuvo la contraseña, pero el usuario era válido en el sistema.
borazuwarah

---

## 🔓 Acceso al sistema

### 4️⃣ Fuerza bruta con usuario válido

Con el nuevo usuario obtenido desde los metadatos, se volvió a ejecutar Hydra:

```bash
hydra -l borazuwarah -P /usr/share/wordlists/rockyou.txt.gz ssh://172.17.0.2 -t 4
```

✅ **Resultado:**
Se obtuvo acceso exitoso al servicio SSH. Password 123456

---

## 🖥️ Enumeración post-explotación

Una vez dentro del sistema:

```bash
whoami
id
```

Se confirmó:

* El usuario con el que se accedió
* Su pertenencia a grupos
* Sus privilegios actuales

---

## 🚀 Escalada de privilegios

### 5️⃣ Revisión de privilegios sudo

Se listaron los permisos sudo del usuario:

```bash
sudo -l
```

📌 **Resultado crítico:**

* El usuario tenía permisos para ejecutar **`/bin/bash` como root** sin contraseña.

---

### 6️⃣ Escalada a root

Se aprovechó este permiso para obtener una shell con privilegios elevados:

```bash
sudo bash
```

🎉 **Resultado:**
Acceso completo como **root** al sistema.

---

## 🏁 Conclusión

La máquina fue comprometida mediante una combinación de:

* Enumeración básica de servicios
* Análisis de metadatos en recursos web
* Ataque de fuerza bruta dirigido
* Mala configuración de privilegios sudo

Este laboratorio demuestra la importancia de:

* No exponer información sensible en metadatos
* Limitar permisos sudo
* Restringir accesos SSH innecesarios

---

## 🛡️ Mitigaciones recomendadas

* Eliminar metadatos de imágenes públicas
* Deshabilitar login SSH innecesario
* Usar contraseñas robustas
* Configurar correctamente `sudoers`

---
