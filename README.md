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
- [Configure Podman GPU support](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/cdi-support.html)

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

## **2. Check and Configure GPU in WSL for Podman**

### **2.1 Verify GPU Access in WSL**

Before setting up Podman, ensure WSL detects your GPU:

```sh
ls /dev/dxg
```

If this file does not exist, update WSL and restart it:

```sh
wsl --update
wsl --shutdown
```

### **2.2 Enable GPU CDI for Podman**

Run the following command to generate CDI configuration:

```sh
sudo nvidia-ctk cdi generate --output=/etc/cdi/nvidia.yaml
```

Verify CDI files exist:

```sh
ls /etc/cdi
```

List available CDI devices:

```sh
nvidia-ctk cdi list
```

If `nvidia.com/gpu=all` appears in the output, CDI is properly configured.

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

```sh
podman run -d \
  --gpus=all \
  --device nvidia.com/gpu=all \
  -v /home/$USER/ollama_data:/root/.ollama \
  -p 127.0.0.1:11434:11434 \
  --network=my-ollama-net \
  --name ollama \
  docker.io/ollama/ollama
```

### **Ollama Container (CPU-Only Mode)**

```sh
podman run -d \
  -v /hom/$USER/ollama_data:/root/.ollama \
  -p 127.0.0.1:11434:11434 \
  --network=my-ollama-net \
  --name ollama \
  docker.io/ollama/ollama
```

### **Open WebUI Container**

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

```sh
podman exec -it ollama nvidia-smi
```

---

## **7. Kill Switch from Windows**

### **Stopping Containers Only**

#### **Command Prompt (cmd):**

```cmd
wsl.exe -d Ubuntu-24.04 -- podman stop ollama && wsl.exe -d Ubuntu-24.04 -- podman stop open-webui
```

#### **PowerShell Script:**

```powershell
powershell -Command "& {wsl -d Ubuntu-24.04 -e podman stop ollama; wsl -d Ubuntu-24.04 -e podman stop open-webui}"
```

### **Stopping WSL Ubuntu-24.04 (Including All Running Containers)**

```cmd
wsl -t Ubuntu-24.04
```

### **Creating Windows Shortcuts for Start/Stop**

- **Monitor utilization:**
  ```
  wsl.exe -d Ubuntu-24.04 -e watch -n 1 podman exec -it ollama nvidia-smi
  ```
- **Stop Containers:**
  ```
  wsl.exe -d Ubuntu-24.04 -- podman stop ollama && wsl.exe -d Ubuntu-24.04 -- podman stop open-webui
  ```
- **Start Containers:**
  ```
  wsl.exe -d Ubuntu-24.04 -- podman start ollama && wsl.exe -d Ubuntu-24.04 -- podman start open-webui
  ```

---

## **8. Download Models**

- **All in one**
```sh
echo "gemma gemma:2b gemma2 gemma2:2b qwen2.5-coder qwen2.5-coder:3b qwen2.5-coder:1.5b qwen2.5-coder:0.5b codellama llama3.2-vision mistral-nemo starcoder2 starcoder2:7b dolphin3 deepscaler llama3.2:1b llama3.2 deepseek-r1:1.5b deepseek-r1 deepseek-r1:8b deepseek-coder-v2 deepseek-coder deepseek-coder:6.7b llama2-uncensored openthinker codegemma codegemma:2b phi llava phi3 mistral" | xargs -n 1 podman exec -it ollama ollama pull
```
- **One by one**
```sh
podman exec -it ollama ollama pull gemma
```
```sh
podman exec -it ollama ollama pull gemma:2b
```
```sh
podman exec -it ollama ollama pull gemma2
```
```sh
podman exec -it ollama ollama pull gemma2:2b
```
```sh
podman exec -it ollama ollama pull qwen2.5-coder
```
```sh
podman exec -it ollama ollama pull qwen2.5-coder:3b
```
```sh
podman exec -it ollama ollama pull qwen2.5-coder:1.5b
```
```sh
podman exec -it ollama ollama pull qwen2.5-coder:0.5b
```
```sh
podman exec -it ollama ollama pull codellama
```
```sh
podman exec -it ollama ollama pull llama3.2-vision
```
```sh
podman exec -it ollama ollama pull mistral-nemo
```
```sh
podman exec -it ollama ollama pull starcoder2
```
```sh
podman exec -it ollama ollama pull starcoder2:7b
```
```sh
podman exec -it ollama ollama pull dolphin3
```
```sh
podman exec -it ollama ollama pull deepscaler
```
```sh
podman exec -it ollama ollama pull llama3.2:1b
```
```sh
podman exec -it ollama ollama pull llama3.2
```
```sh
podman exec -it ollama ollama pull deepseek-r1:1.5b
```
```sh
podman exec -it ollama ollama pull deepseek-r1
```
```sh
podman exec -it ollama ollama pull deepseek-r1:8b
```
```sh
podman exec -it ollama ollama pull deepseek-coder-v2
```
```sh
podman exec -it ollama ollama pull deepseek-coder
```
```sh
podman exec -it ollama ollama pull deepseek-coder:6.7b
```
```sh
podman exec -it ollama ollama pull llama2-uncensored
```
```sh
podman exec -it ollama ollama pull openthinker
```
```sh
podman exec -it ollama ollama pull codegemma
```
```sh
podman exec -it ollama ollama pull codegemma:2b
```
```sh
podman exec -it ollama ollama pull phi
```
```sh
podman exec -it ollama ollama pull llava
```
```sh
podman exec -it ollama ollama pull phi3
```
```sh
podman exec -it ollama ollama pull mistral
```

---


## **9. Troubleshooting**

### **General Issues**

- **Restart Podman Service**
  ```sh
  systemctl --user restart podman
  ```
- **Check Containers**
  ```sh
  podman ps -a
  ```

### **Ollama Issues**

- **Check Connection to Ollama**
  ```sh
  podman exec -it open-webui curl -X GET http://ollama:11434/api/tags
  ```
- **Clear Cache in Open WebUI**
  ```sh
  podman exec -it open-webui /bin/sh -c "rm -rf /app/backend/data/cache/*"
  ```
- **Check GPU in Ollama**
  ```sh
  podman exec -it ollama nvidia-smi
  ```

### **Podman Network Issues**

- **Check Podman Network**
  ```sh
  podman network inspect my-ollama-net
  ```
- **Check Container IP Addresses**
  ```sh
  podman inspect ollama | grep '"IPAddress"'
  podman inspect open-webui | grep '"IPAddress"'
  ```

### **GPU Issues**

- **Check if GPU is Detected in WSL**
  ```sh
  ls /dev/dxg
  ```
- **Check if CDI is Registered**
  ```sh
  nvidia-ctk cdi list
  ```
- **Test GPU in Podman**
  ```sh
  podman run --rm --device nvidia.com/gpu=all docker.io/nvidia/cuda:12.2.0-base-ubuntu22.04 nvidia-smi
  ```
---

This setup ensures that **Ollama** runs as a local backend LLM service and **Open WebUI** serves as a frontend interface, all within an isolated Podman network.

