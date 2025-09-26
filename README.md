# Nessus Essentials Install on Debian/Kali

## 1. Update system
```bash
sudo apt update && sudo apt install -y wget curl apt-transport-https
2. Download Nessus (replace with the latest URL from Tenable)
bash
Copy code
wget -O nessus.deb "PASTE_URL_HERE"
3. Install Nessus
bash
Copy code
sudo dpkg -i nessus.deb || sudo apt -f install -y
4. Enable and start Nessus
bash
Copy code
sudo systemctl enable --now nessusd
5. Register Nessus (replace ACTIVATION_CODE)
bash
Copy code
sudo /opt/nessus/sbin/nessuscli fetch --register ACTIVATION_CODE
6. Update Nessus plugins
bash
Copy code
sudo /opt/nessus/sbin/nessuscli update --all
7. Restart Nessus
bash
Copy code
sudo systemctl restart nessusd
8. Open Nessus Web UI
cpp
Copy code
https://127.0.0.1:8834
or

cpp
Copy code
https://<your_server_ip>:8834
