# Kali Linux con IA: Guía para Claude Desktop en Windows 11 + MCP Server en Kali Linux

Esta guía detalla la integración de **Claude Desktop** con un servidor **MCP (Model Context Protocol)** para ejecutar herramientas de **Kali Linux** desde Windows 11.

## 📌 Origen y Créditos
Esta documentación se basa en los siguientes recursos:
* La guía oficial de Kali para macOS.
* El vídeo de YouTube "Kali Linux con IA".
* El paquete oficial `mcp-kali-server` de Kali Tools.

---

## 🛠 Requisitos Previos
* **Kali Linux:** Instalado y en ejecución (VM, Cloud o Bare-metal).
* **Conectividad:** Debe estar en la misma red que Windows 11 y ser accesible por IP.
* **Windows 11:** Sistema actualizado.
* **SSH:** Sin firewalls bloqueando el puerto 22.
* **Cuenta Anthropic:** Para el uso de Claude Desktop.

> [!WARNING]
> **Seguridad:** Nunca expongas el servidor MCP directamente a internet. Claude siempre pedirá confirmación antes de ejecutar comandos en Kali.

---

## 1. Configuración en Kali Linux

### 1.1. Habilitar SSH
Si no está configurado, abre una terminal en Kali y ejecuta:

```bash
sudo apt update
sudo apt install -y openssh-server 
sudo systemctl enable --now ssh
```
Verifica el estado con: `sudo systemctl status ssh`.

### 1.2. Instalar el servidor MCP
Instala el paquete necesario e inicia el servidor para crear la API:

```bash
sudo apt update 
sudo apt install -y mcp-kali-server 
kali-server-mcp
```
*El servidor correrá en `http://127.0.0.1:5000`*.

Para comprobar que todo funciona hasta ahora, en otro terminal ejecuta `mcp-server` (esto es lo que nuestro cliente MCP, Claude Desktop, acabará ejecutando):

```bash
$ mcp-server
```

### 🛠 Corrección de Bug (Health Check)
Existe un error conocido donde la función de verificación no encuentra las herramientas. Debes editar `mcp_server.py` en `/usr/share/mcp-kali-server`:

**Código a corregir:**
```python
import shutil 
# Dentro de def health_check(): 
for tool in essential_tools:
    tools_status[tool] = shutil.which(tool) is not None 
```

### 1.3. Herramientas de Kali
Instala el set de herramientas que Claude utilizará:

```bash
sudo apt install -y dirb gobuster nikto nmap enum4linux-ng hydra john metasploit-framework sqlmap wpscan wordlists
sudo gunzip -v /usr/share/wordlists/rockyou.txt.gz 
```

---

## 2. Configuración en Windows 11

### 2.1. Software Necesario
* **Git for Windows:** Necesario para disponer de `ssh.exe`. Descarga en [git-scm.com](https://git-scm.com/download/win).
* **Claude Desktop:** Descarga en [claude.com/download](https://claude.com/download) e inicia sesión.

### 2.2. Llaves SSH (Git Bash)
Genera llaves sin contraseña para la conexión automática:

```bash
cd ~/.ssh 
ssh-keygen.exe -t ed25519 -C "claude@kali" 
# Nombre: llave1 (Enter) | Passphrase: vacío (Enter x2) 
```

### 2.3. Vincular con Kali
Copia la clave pública a tu IP de Kali:

```bash
ssh-copy-id -i ~/.ssh/llave1.pub kali@TU_IP_KALI 
```
Prueba la conexión directa: `ssh -i ~/.ssh/llave1 kali@TU_IP_KALI`.

### 2.4. Configurar Claude Desktop
En Claude Desktop, ve a **Ajustes > Desarrollador > Editar configuración**. Edita el archivo `claude_desktop_config.json`:

```json
{
  "preferences": {
    "coworkWebSearchEnabled": true,
    "coworkScheduledTasksEnabled": false,
    "ccdScheduledTasksEnabled": false
  },
  "mcpServers": {
    "mcp-kali-server": {
      "command": "C:\\Program Files\\Git\\usr\\bin\\ssh.exe",
      "args": [
        "-i", "C:\\Users\\TU_USUARIO\\.ssh\\llave1",
        "-o", "StrictHostKeyChecking=no",
        "kali@TU_IP_KALI",
        "mcp-server"
      ],
      "transport": "stdio"
    }
  }
}
```
*Importante: *
* Usa doble barra `\\` en las rutas y reinicia la aplicación al finalizar.
* Reemplaza TU_USUARIO por tu nombre de usuario de Windows (ej. jeler).
* Reemplaza TU_IP_KALI por la IP real de Kali.

---

## 3. Pruebas
Pregunta a Claude en el chat:
> "¿Podrías hacer un escaneo de puertos básico con nmap en scanme.nmap.org?" 

Claude te pedirá confirmación y, tras permitirlo, mostrará los resultados obtenidos directamente desde tu terminal de Kali].
