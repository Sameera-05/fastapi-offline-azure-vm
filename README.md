# FastAPI Offline Deployment on Azure VM

This project demonstrates a **production-simulated FastAPI deployment** inside an Azure Linux VM, without direct internet access during runtime. It includes:
- Manual `.whl` package management (`pip install --no-index`)
- `systemd`-based process management
- Health-check endpoint and offline monitoring via `journalctl`
---

## Project Overview

We designed a minimal FastAPI service and deployed it **inside an Azure virtual machine** using a realistic on-prem-style workflow. The application is served via Uvicorn and auto-managed as a background service using `systemd`.

> This is intended to **simulate edge/on-prem deployments**, where internet access may be limited or blocked during runtime.

---

##  Technologies Used

- **Azure VM** (Debian/Ubuntu)
- **FastAPI** + **Uvicorn**
- **Python 3.11**
- **systemd** (to register FastAPI as a service)
- **SSH tunnel** for local endpoint testing
- **Offline pip install** with `--no-index`

---

##  Folder Structure

```
fastapi_offline_vm/
├── app/
│   ├── main.py              # FastAPI app with `/` and `/health` endpoints
│   └── requirements.txt     # FastAPI + uvicorn
├── offline_packages/        # Pre-downloaded .whl dependencies
└── fastapi.service          # systemd unit file
```

---

##  Deployment Steps

###  1. VM Setup
- Create Ubuntu/Debian Azure VM
- SSH using `.pem` key (via PowerShell or Git Bash)
- Install Python, pip, venv, tmux:
  ```bash
  sudo apt update
  sudo apt install -y python3 python3-pip python3-venv tmux
  ```

###  2. Prepare on Local (with Internet)
- Write FastAPI app and requirements.txt
- Run:
  ```bash
  pip download -r app/requirements.txt -d offline_packages/
  ```
- SCP the entire project folder to the VM
<img width="1960" height="1297" alt="image" src="https://github.com/user-attachments/assets/bf597f89-da05-4239-a8e9-bf301d5291ea" />
<img width="1470" height="867" alt="image" src="https://github.com/user-attachments/assets/01984848-2a97-472f-b117-143f02ce8638" />

###  3. Inside the VM
- Create and activate venv:
  ```bash
  python3 -m venv venv
  source venv/bin/activate
  ```
- Install packages offline:
  ```bash
  pip install --no-index --find-links=offline_packages -r app/requirements.txt
  ```
###  4. Or Use `systemd`
- Copy `fastapi.service` to `/etc/systemd/system/`
- Enable service:
  ```bash
  sudo systemctl daemon-reload
  sudo systemctl start fastapi.service
  sudo systemctl enable fastapi.service
  ```
- Check status and logs:
  ```bash
  sudo systemctl status fastapi.service
  journalctl -u fastapi.service -f
  ```
<img width="3060" height="1112" alt="image" src="https://github.com/user-attachments/assets/eba1b321-850e-428e-9030-565767aa24a1" />

###  5. Test Endpoint via SSH Tunnel
On your local machine:

```bash
ssh -i path/to/key.pem -L 8000:127.0.0.1:8000 azureuser@<vm-ip>
```

Then open browser to: [http://127.0.0.1:8000](http://127.0.0.1:8000)
<img width="1722" height="765" alt="image" src="https://github.com/user-attachments/assets/61f2b330-213c-4856-8f00-e1f97d7c8c75" />

---

##  Security Practices
- PEM file permissions were fixed via PowerShell ACLs to enable OpenSSH usage
- Port 8000 exposed only locally for testing (tunneled, not public)

---

##  Sample Endpoints

- `/` → `{{"message": "Hello from offline FastAPI on Azure VM!"}}`
- `/info` → `{{"status": "ok", "note": "offline FastAPI test"}}`
- `/health` → `{{"status": "healthy"}}`
<img width="1890" height="135" alt="image" src="https://github.com/user-attachments/assets/3ac5f4b5-26c5-4d14-99be-3812536b222f" />

---

##  What I Learned

- Working with **Azure CLI and VM creation**
- Running **FastAPI in offline environments**
- Writing **systemd unit files** for service monitoring
- Building **SSH tunnels** to test internal services

- Building SSH tunnels to test internal services

---
