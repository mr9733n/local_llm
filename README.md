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

#### **Explanation of the options:**
- `--gpus=all` (only in GPU mode) → Enables GPU acceleration for the container.
- `-v /home/$USER/ollama_data:/root/.ollama` → Mounts the host directory `/home/$USER/ollama_data` into the container at `/root/.ollama`. This ensures that downloaded models persist even after restarting the container.
- `-p 127.0.0.1:11434:11434` → Maps port `11434` on the host to the container, making Ollama accessible **only locally**.
- `--network=my-ollama-net` → Attaches the container to the internal Podman network `my-ollama-net` for secure communication with Open WebUI.
- `--name ollama` → Assigns the container the name `ollama`.
- `docker.io/ollama/ollama` → Specifies the Ollama image from Docker Hub.

🔹 **Do I need to create `/home/$USER/ollama_data` manually?**
Yes, before running the container, create the directory if it does not exist:
```sh
mkdir -p /home/$USER/ollama_data
```
This ensures the volume is properly mounted and persistent.

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

#### **Explanation of the options:**
- `-p 127.0.0.1:7860:8080` → Maps Open WebUI's internal port (`8080`) to `127.0.0.1:7860`, ensuring it is only accessible from the local machine.
- `-v /home/$USER/open-webui:/app/backend/data` → Mounts the host directory `/home/$USER/open-webui` to store WebUI settings and conversation history persistently.
- `-e OLLAMA_BASE_URL="http://ollama:11434"` → Configures Open WebUI to use Ollama as the backend by pointing it to the Ollama container.
- `--network=my-ollama-net` → Connects the WebUI container to the same network as Ollama, allowing seamless communication.
- `--name open-webui` → Assigns the container the name `open-webui`.
- `ghcr.io/open-webui/open-webui:main` → Uses the latest Open WebUI image from GitHub Container Registry.

🔹 **Do I need to create `/home/$USER/open-webui` manually?**
Yes, before starting the container, ensure the directory exists:
```sh
mkdir -p /home/$USER/open-webui
```
This will prevent volume mounting issues and ensure settings persist.

---

## **6. Check Containers**
```sh
podman ps -a
```

---

## **7. Download Models**

Run the following commands to pull models into Ollama:
```sh
podman exec -it ollama ollama pull gemma:2b
podman exec -it ollama ollama pull llama3.2:1b
podman exec -it ollama ollama pull deepseek-r1:1.5b
podman exec -it ollama ollama pull mistral
```

---

## **8. Troubleshooting**

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

