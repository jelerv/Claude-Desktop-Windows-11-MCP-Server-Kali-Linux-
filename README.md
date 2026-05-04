# Kali Linux con IA: Guía para Claude Desktop en Windows 11 + MCP Server en Kali Linux

[cite_start]Esta guía detalla la integración de **Claude Desktop** con un servidor **MCP (Model Context Protocol)** para ejecutar herramientas de **Kali Linux** desde Windows 11[cite: 1].

## 📌 Origen y Créditos
Esta documentación se basa en los siguientes recursos:
* [cite_start]La guía oficial de Kali para macOS[cite: 3].
* [cite_start]El vídeo de YouTube "Kali Linux con IA"[cite: 4].
* [cite_start]El paquete oficial `mcp-kali-server` de Kali Tools[cite: 5].

---

## 🛠 Requisitos Previos
* [cite_start]**Kali Linux:** Instalado y en ejecución (VM, Cloud o Bare-metal)[cite: 7].
* [cite_start]**Conectividad:** Debe estar en la misma red que Windows 11 y ser accesible por IP[cite: 8].
* [cite_start]**Windows 11:** Sistema actualizado[cite: 9].
* [cite_start]**SSH:** Sin firewalls bloqueando el puerto 22[cite: 11].
* [cite_start]**Cuenta Anthropic:** Para el uso de Claude Desktop[cite: 12].

> [!WARNING]
> [cite_start]**Seguridad:** Nunca expongas el servidor MCP directamente a internet[cite: 14]. [cite_start]Claude siempre pedirá confirmación antes de ejecutar comandos en Kali[cite: 16].

---

## 1. Configuración en Kali Linux

### 1.1. Habilitar SSH
[cite_start]Si no está configurado, abre una terminal en Kali y ejecuta[cite: 18, 19]:

```bash
[cite_start]sudo apt update [cite: 20]
[cite_start]sudo apt install -y openssh-server [cite: 21]
[cite_start]sudo systemctl enable --now ssh [cite: 22]
```
[cite_start]Verifica el estado con: `sudo systemctl status ssh`[cite: 24].

### 1.2. Instalar el servidor MCP
[cite_start]Instala el paquete necesario e inicia el servidor para crear la API[cite: 25, 27, 28]:

```bash
[cite_start]sudo apt update [cite: 26]
[cite_start]sudo apt install -y mcp-kali-server [cite: 27]
[cite_start]kali-server-mcp [cite: 29]
```
[cite_start]*El servidor correrá en `http://127.0.0.1:5000`*[cite: 32].

### 🛠 Corrección de Bug (Health Check)
[cite_start]Existe un error conocido donde la función de verificación no encuentra las herramientas[cite: 36, 37]. [cite_start]Debes editar `mcp_server.py` en `/usr/share/mcp-kali-server`[cite: 37]:

**Código a corregir:**
```python
[cite_start]import shutil [cite: 56]

# [cite_start]Dentro de def health_check(): [cite: 62]
[cite_start]for tool in essential_tools: [cite: 67]
    [cite_start]tools_status[tool] = shutil.which(tool) is not None [cite: 68]
```

### 1.3. Herramientas de Kali
[cite_start]Instala el set de herramientas que Claude utilizará[cite: 70]:

```bash
[cite_start]sudo apt install -y dirb gobuster nikto nmap enum4linux-ng hydra john metasploit-framework sqlmap wpscan wordlists [cite: 72]
[cite_start]sudo gunzip -v /usr/share/wordlists/rockyou.txt.gz [cite: 73]
```

---

## 2. Configuración en Windows 11

### 2.1. Software Necesario
* [cite_start]**Git for Windows:** Necesario para disponer de `ssh.exe`[cite: 76]. [cite_start]Descarga en [git-scm.com](https://git-scm.com/download/win)[cite: 77].
* [cite_start]**Claude Desktop:** Descarga en [claude.com/download](https://claude.com/download) e inicia sesión[cite: 80, 81].

### 2.2. Llaves SSH (Git Bash)
[cite_start]Genera llaves sin contraseña para la conexión automática[cite: 82, 83]:

```bash
[cite_start]cd ~/.ssh [cite: 84]
[cite_start]ssh-keygen.exe -t ed25519 -C "claude@kali" [cite: 85]
# Nombre: llave1 (Enter) | [cite_start]Passphrase: vacío (Enter x2) [cite: 86, 87]
```

### 2.3. Vincular con Kali
[cite_start]Copia la clave pública a tu IP de Kali[cite: 89, 91]:

```bash
[cite_start]ssh-copy-id -i ~/.ssh/llave1.pub kali@TU_IP_KALI [cite: 91]
```
[cite_start]Prueba la conexión directa: `ssh -i ~/.ssh/llave1 kali@TU_IP_KALI`[cite: 95].

### 2.4. Configurar Claude Desktop
[cite_start]En Claude Desktop, ve a **Ajustes > Desarrollador > Editar configuración**[cite: 98, 99]. [cite_start]Edita el archivo `claude_desktop_config.json`[cite: 100]:

```json
{
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
[cite_start]*Importante: Usa doble barra `\\` en las rutas y reinicia la aplicación al finalizar*[cite: 122, 126].

---

## 3. Pruebas
Pregunta a Claude en el chat:
> [cite_start]"¿Podrías hacer un escaneo de puertos básico con nmap en scanme.nmap.org?" [cite: 129]

[cite_start]Claude te pedirá confirmación y, tras permitirlo, mostrará los resultados obtenidos directamente desde tu terminal de Kali[cite: 130, 131].
