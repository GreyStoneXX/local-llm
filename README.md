# local-llm

# Local AI Setup Guide: Ollama + VS Code (Qwen 2.5 Coder)

This guide walks you through installing Ollama, downloading the recommended AI coding models, running the server service, configuring the network host so team members can access it, and setting up VS Code via the **Continue** extension.

---

### Prerequisites
* **Server Machine:** Windows/Linux/macOS with an NVIDIA GPU or modern CPU.
* **Network:** Both the server and client machines must be on the same Local Area Network (LAN).
* **VS Code:** Installed on client machine(s).

---

## Part 1: Server Setup (Host Machine)

### 1. Install Ollama
Download and install Ollama on the main host computer from [ollama.com](https://ollama.com).

### 2. Configure Ollama for LAN Access
By default, Ollama only listens to local connections (`127.0.0.1`). To allow other computers on your network to connect, set the `OLLAMA_HOST` environment variable:

* **Windows:**
  1. Search for **"Environment Variables"** in the Start Menu and select **Edit system environment variables**.
  2. Click **Environment Variables...**
  3. Under **System variables**, click **New...**
     * **Variable name:** `OLLAMA_HOST`
     * **Variable value:** `0.0.0.0`
  4. Click **OK** to save.
  5. Close Ollama completely from the system tray (right-click the llama icon → **Quit**), then restart it.

* **Linux / macOS:**
  ```bash
  export OLLAMA_HOST=0.0.0.0
  ```

### 3. Open Firewall Port (Windows Server)
Allow incoming traffic on port `11434`:
1. Open **Windows Defender Firewall with Advanced Security**.
2. Click **Inbound Rules** → **New Rule...**
3. Select **Port** → **TCP** → Specific local port: `11434`.
4. Select **Allow the connection** and save the rule (e.g., name it "Ollama LAN").

### 4. Pull the Required Models
Open your terminal on the server and run:

```bash
# Main model for Chat, Refactoring, and Inline Editing
ollama pull qwen2.5-coder:7b

# Lightweight model for real-time Autocomplete (prevents VRAM crashes)
ollama pull qwen2.5-coder:1.5b

# Embeddings model (optional, for codebase context indexing)
ollama pull nomic-embed-text
```

### 5. Running & Managing Ollama Server

Ollama runs as a background service by default, but you can also manage or run it manually depending on your workflow:

* **Automatic Background Service (Default):**
  * **Windows:** Ollama starts automatically with Windows and sits in the system tray. If you set `OLLAMA_HOST=0.0.0.0`, simply restart the app from the Start Menu or system tray.
  * **Linux:** Managed via systemd. Start/enable it with:
    ```bash
    sudo systemctl enable --now ollama
    ```

* **Manual Execution via Terminal:**
  If you prefer running Ollama directly from the terminal (or need to debug CUDA/GPU issues):
  1. Start the server daemon:
     ```bash
     ollama serve
     ```
  2. Test a model interactively in a separate terminal window:
     ```bash
     ollama run qwen2.5-coder:7b
     ```

> **Note:** Clients using VS Code do **NOT** need to run `ollama run`. As long as the Ollama background service (`ollama serve` or system tray app) is active on the server, VS Code will load and unload models automatically on demand.

---

## Part 2: Client Setup (VS Code Integration)

Perform these steps on the machines that will use the AI model (whether locally or remotely over the network).

### 1. Get the Server IP Address
Find the IP address of the server machine running Ollama (e.g., `192.168.1.100`).
* Windows command: `ipconfig`
* Linux/macOS command: `ip a` or `ifconfig`

### 2. Install VS Code Extension
1. Open **VS Code**.
2. Go to Extensions (`Ctrl+Shift+X` or `Cmd+Shift+X`).
3. Search for and install **Continue**.

### 3. Configure `config.yaml` in VS Code
1. Click the **Continue** icon on the left sidebar in VS Code.
2. Click the gear icon (**⚙️**) in the bottom-right corner of the Continue panel to open `config.yaml`.
3. Replace the contents with the following configuration:

> **Note:** If running VS Code on the **same machine** as Ollama, keep `http://localhost:11434`. If connecting from **another PC**, replace `localhost` with your server's IP address (e.g., `http://192.168.1.100:11434`).

```yaml
name: Team AI Config
version: 1.0.0
schema: v1
models:
  - name: Qwen 2.5 Coder 7B
    provider: ollama
    model: qwen2.5-coder:7b
    apiBase: http://<SERVER_IP>:11434
    roles:
      - chat
      - edit
      - apply
  - name: Qwen 2.5 Coder 1.5B
    provider: ollama
    model: qwen2.5-coder:1.5b
    apiBase: http://<SERVER_IP>:11434
    roles:
      - autocomplete
  - name: Nomic Embed
    provider: ollama
    model: nomic-embed-text:latest
    apiBase: http://<SERVER_IP>:11434
    roles:
      - embed
```

---

## Part 3: Troubleshooting

* **CUDA / VRAM Errors (`0xc0000409`):** Occurs when VRAM is saturated. Separating the `7B` model (chat) and `1.5B` model (autocomplete) as shown in the config fixes this issue on GPUs with 4 GB–6 GB VRAM.
* **Connection Refused:** Ensure the server firewall allows port `11434` and that `OLLAMA_HOST=0.0.0.0` was set before launching Ollama.
* **Port Conflict / Service Not Responding:** If running `ollama serve` gives an error about port `11434` being in use, close the background Ollama process in the Windows system tray first.
