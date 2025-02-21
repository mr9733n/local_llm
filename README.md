# Documentation: Setting Up Ollama with Open WebUI in Podman

## Links

- [Ollama GitHub](https://github.com/ollama/ollama)
- [Ollama Official Site](https://ollama.com/)
- [Ollama Docker Hub](https://hub.docker.com/r/ollama/ollama)
- [Llama Official Downloads](https://www.llama.com/llama-downloads/)
- [Hugging Face](https://huggingface.co/welcome)
- [Open WebUI GitHub](https://github.com/open-webui/open-webui)
- [Open WebUI Documentation](https://docs.openwebui.com/)
- [Podman Installation](https://podman.io/docs/installation)
- [Docker Installation (Ubuntu)](https://docs.docker.com/engine/install/ubuntu/)

---

## **1. Install WSL and Ubuntu**

```sh
wsl --list --online
wsl --install Ubuntu-24.04
```

Update and upgrade the system:

```sh
sudo apt-get update
sudo apt-get upgrade -y
```

Disable automatic Windows drive mounting in WSL:

```sh
sudo nano /etc/wsl.conf
```

Add the following:

```ini
[automount]
enabled = false
```

Save and exit (`Ctrl + X`, then `Y`, then `Enter`).

Restart WSL:

```sh
wsl --shutdown
```

---

## **2. Verify GPU Support**

Check NVIDIA GPU status:

```sh
nvidia-smi
```

Monitor GPU usage:

```sh
watch -n 1 nvidia-smi
```

---

## **3. Install Podman**

For Ubuntu 20.10 and newer:

```sh
sudo apt-get -y install podman
```

---

## **4. Create an Internal Network for Containers**

```sh
podman network create my-ollama-net
```

---

## **5. Create and Run Containers**

### **Ollama Container (GPU Mode)**

This command runs the **Ollama** container in detached mode (`-d`) with GPU acceleration enabled, ensuring that the container has access to all GPUs.

```sh
podman run -d \
  --gpus=all \
  -v /home/$USER/ollama_data:/root/.ollama \
  -p 127.0.0.1:11434:11434 \
  --network=my-ollama-net \
  --name ollama \
  docker.io/ollama/ollama
```

### **Ollama Container (CPU-Only Mode)**

If you do not have a compatible GPU or prefer to run Ollama on the CPU, remove the `--gpus=all` flag and use the following command:

```sh
podman run -d \
  -v /home/$USER/ollama_data:/root/.ollama \
  -p 127.0.0.1:11434:11434 \
  --network=my-ollama-net \
  --name ollama \
  docker.io/ollama/ollama
```

---

### **Open WebUI Container**

This command runs **Open WebUI** as a detached container, linking it with the **Ollama** backend.

```sh
podman run -d \
  -p 127.0.0.1:7860:8080 \
  -v /home/$USER/open-webui:/app/backend/data \
  -e OLLAMA_BASE_URL="http://ollama:11434" \
  --network=my-ollama-net \
  --name open-webui \
  ghcr.io/open-webui/open-webui:main
```

---

## **6. Check Containers**

```sh
podman ps -a
```

---

## **7. Kill Switch from Windows**

To quickly stop the containers or the entire WSL instance, you can create shortcuts or scripts:

### **Stopping Containers Only**

#### **Windows Command Prompt (cmd) Script:**

```cmd
@echo off
wsl -d Ubuntu-24.04 -e podman stop open-webui
wsl -d Ubuntu-24.04 -e podman stop ollama
exit
```

Save this as `stop_ollama.bat` and double-click to stop the containers.

#### **PowerShell Script:**

```powershell
Write-Output "Stopping Open WebUI and Ollama..."
wsl -d Ubuntu-24.04 -e podman stop open-webui
wsl -d Ubuntu-24.04 -e podman stop ollama
Write-Output "Containers stopped."
```

Save as `stop_ollama.ps1` and run it in PowerShell.

### **Stopping WSL Ubuntu-24.04 (Including All Running Containers)**

```cmd
wsl -t Ubuntu-24.04
```

This completely stops WSL and any running containers inside it.

### **Creating Windows Shortcuts for Start/Stop**

Instead of scripts, you can create shortcuts:

1. **Right-click on Desktop → New → Shortcut**
2. **For stopping containers**, use:
   ```
   powershell -Command "& {wsl -d Ubuntu-24.04 -e podman stop open-webui; wsl -d Ubuntu-24.04 -e podman stop ollama}"
   ```
3. **For starting containers**, use:
   ```
   powershell -Command "& {wsl -d Ubuntu-24.04 -e podman start ollama; wsl -d Ubuntu-24.04 -e podman start open-webui}"
   ```
4. Name the shortcuts appropriately (e.g., "Stop WSL & Containers", "Start Containers").
5. Optionally, change icons for better recognition.

Now, you can easily start or stop everything with one click. 🚀

---

## **8. Download Models**

Run the following commands to pull models into Ollama:

```sh
podman exec -it ollama ollama pull gemma:2b
podman exec -it ollama ollama pull llama3.2:1b
podman exec -it ollama ollama pull deepseek-r1:1.5b
podman exec -it ollama ollama pull mistral
```

---

## **9. Troubleshooting**

### **Clear Cache in Open WebUI**

```sh
podman exec -it open-webui /bin/sh -c "rm -rf /app/backend/data/cache/*"
```

### **Check Connection to Ollama**

```sh
podman exec -it open-webui curl -X GET http://ollama:11434/api/tags
```

### **Restart Podman Service**

```sh
systemctl --user restart podman
```

### **Inspect Network and Containers**

```sh
podman network inspect my-ollama-net
podman inspect ollama | grep '"IPAddress"'
podman inspect open-webui | grep '"IPAddress"'
```

---

This setup ensures that **Ollama** runs as a local backend LLM service and **Open WebUI** serves as a frontend interface, all within an isolated Podman network.

