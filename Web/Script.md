## 1. Managing your Open WebUI and Ollama environment involves specific steps before opening, commands to start and stop services, and procedures after closing.
#### Before opening http://localhost:3000 in your browser, check that the two background services are active:
``` Bash
sudo systemctl status docker
```
This Common use to start server Docker.
## If it Off
```bash
sudo systemctl start docker
```
---
## 2. Ensure Ollama is running:
On Linux, Ollama usually runs in the background automatically as a service. You can check if it's running with:
```Bash 
systemctl status ollama
```
## 3. Check if the Open WebUI container is active:
```Bash 
docker ps
```
### If open-webui is in the list, you are ready! If it's not listed, start it:
```Bash
docker start open-webui
```
# Common Commands to OPEN (Start) and CLOSE (Stop)
``` bash 
# 1. Start the Docker service (if not running)
sudo systemctl start docker

# 2. Start the Open WebUI container
docker start open-webui
```
# To STOP everything (Close cleanly):
``` Bash
# 1. Stop the Open WebUI container
docker stop open-webui

# 2. (Optional) Stop Ollama service if you want to free up RAM completely
sudo systemctl stop ollama
```
---
# To Login to another computer
## Step 1: Find your Host Laptop's Local IP Address
On your main ROG Strix laptop (where Docker and Ollama are installed), open a terminal and run:
``` Bash 
hostname -I
```
## Step 2: Open the UI on the Second Laptop
Connect the second laptop to the same Wi-Fi/LAN network as your main laptop.

Open any web browser on the second laptop.

Type your host laptop's IP address followed by :3000 (or :8080, whichever port you used):
``` bASH 
http://192.168.X.X:3000
```
The IP you get is from the common of step 1

---

