# Raspberry Pi 5 + Hailo AI: Servidor LLM Local con Ollama y Open WebUI

Este proyecto esta dirigido para el asistente virtual local de la universidad Ibero Puebla, el cual se encuentra en la fase de software, detalla la configuración paso a paso de una **Raspberry Pi 5** equipada con el **AI HAT (Hailo)** también aplica para el **AI HAT+2** para ejecutar Modelos de Lenguaje de Gran Escala (LLMs) de forma local, aprovechando el NPU para un rendimiento optimizado.

---

## Pre-requisitos

### Hardware Principal
* [Raspberry Pi 5]
* [AI HAT o AI HAT+ 2 (Hailo)]

### Accesorios Adicionales
* [Tarjeta Micro SD] 
* [Fuente de alimentación USB-C (27W)] específica para Pi 5.
* Teclado, Ratón y Monitor HDMI.
---

### 1. Habilitar PCIe Gen 3
Abre la terminal de tu Raspberry Pi y ejecuta:

```bash
sudo raspi-config
```
1. Navega hasta Advanced Options → PCIe Speed.

3. Selecciona Yes cuando se te pregunte si deseas habilitar PCIe Gen 3.

5. Usa la tecla Tab para ir a Finish, presiona Enter y selecciona Yes para reiniciar.

### 2. Actualizar Firmware y Sistema
Una vez de vuelta en el escritorio, asegúrate de tener la última versión del firmware y del sistema operativo:
```bash
sudo apt update
sudo apt full-upgrade -y
sudo rpi-eeprom-update -a
sudo reboot
```
### 3. Instalar dependencias
Instalaremos los controladores necesarios para el chip Hailo. El sistema se reiniciará automáticamente al finalizar:
```bash
sudo apt install dkms
sudo apt install hailo-h10-all
sudo reboot
```
### 4. Verificar instalación
Para confirmar que el hardware de Hailo se reconoce correctamente, ejecuta:
```bash
hailortcli fw-control identify
```
Deberías ver una salida con los detalles del dispositivo Hailo-10.
### 5. Instalar Hailo Model Zoo (Gen AI)
Descarga el paquete Debian específico para la arquitectura ARM64 de la Pi 5:

1.[Haz clic aquí para descargar la versión 5.1.1.](https://dev-public.hailo.ai/2025_12/Hailo10/hailo_gen_ai_model_zoo_5.1.1_arm64.deb)

2.Instálalo desde la terminal (asumiendo que está en la carpeta de Descargas):
```bash
cd ~/Downloads
sudo dpkg -i hailo_gen_ai_model_zoo_5.1.1_arm64.deb
```

## Iniciar Ollama (Servidor y Modelos)
Ahora arrancaremos el servidor hailo-ollama e instalaremos los modelos (LLMs).

### 1. Arrancar el Servidor
Inicia el servicio en una terminal:
```bash
hailo-ollama
```
### 2. Listar e Instalar modelos
Abre una nueva terminal (sin cerrar la anterior) para consultar los modelos disponibles:
```bash
curl --silent http://localhost:8000/hailo/v1/list

```
Para descargar un modelo específico (por ejemplo, Qwen 2.5), usa el comando pull:
```bash
curl --silent http://localhost:8000/api/pull \
     -H 'Content-Type: application/json' \
     -d '{ "model": "qwen2:1.5b", "stream" : true }'
```
### 3. Prueba rápida de chat
Puedes verificar que todo funciona enviando una pregunta vía API:
```bash
curl --silent http://localhost:8000/api/chat \
     -H 'Content-Type: application/json' \
     -d '{"model": "qwen2:1.5b", "messages": [{"role": "user", "content": "Traduce al español: One, Two, Three and Four."}]}'
```
## Instalar y usar Open WebUI

Open WebUI proporciona una interfaz gráfica similar a ChatGPT para interactuar con tus modelos locales de forma sencilla.

### 1. Preparar Docker
Primero, eliminamos cualquier instalación previa de Docker para evitar conflictos:
```bash
sudo apt remove $(dpkg --get-selections docker.io docker-compose docker-doc podman-docker containerd runc | cut -f1)
```
Instalamos el repositorio oficial de Docker:
```bash
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL [https://download.docker.com/linux/debian/gpg](https://download.docker.com/linux/debian/gpg) -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: [https://download.docker.com/linux/debian](https://download.docker.com/linux/debian)
Suites: $(. /etc/os-release && echo "$VERSION_CODENAME")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
## 2. Configurar permisos
Añade tu usuario al grupo Docker para ejecutarlo sin sudo:
```bash
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
```
(Puedes verificar con docker run hello-world)
## 3. Desplegar el contenedor de Open WebUI
Descarga la imagen y ejecuta el contenedor vinculándolo al servidor de Hailo:
```bash
docker pull ghcr.io/open-webui/open-webui:main

docker run -d -e OLLAMA_BASE_URL=[http://127.0.0.1:8000](http://127.0.0.1:8000) -v open-webui:/app/backend/data --name open-webui --network=host --restart always ghcr.io/open-webui/open-webui:main
```
## 4. Acceso
El contenedor puede tardar un minuto en inicializarse. Puedes seguir el progreso con:
```bash
docker logs open-webui -f
```
Una vez listo, abre tu navegador y entra en:  http://127.0.0.1:8080
