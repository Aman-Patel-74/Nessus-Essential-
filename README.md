# Nessus-Essential-

install Nessus Essentials on Debian 12 / Kali and get it registered + working. I’ll include two ready scripts you can save to your computer or push to GitHub. Replace the placeholders I mark (PASTE_URL_HERE, ACTIVATION_CODE) with your real values.

1 — Before you start

Make sure you have a 64-bit Debian/Kali system, root or sudo access, and an Internet connection.

Download your Nessus Essentials activation code from Tenable (you’ll need it during registration).

2 — Manual install steps (single commands)

update apt and install common helpers:

sudo apt update && sudo apt install -y wget curl gnupg apt-transport-https


Download the Nessus .deb package (open Tenable downloads page in browser and copy the Debian x86_64 .deb URL). Example pattern:

# Replace the URL below with the one you copy from Tenable downloads
NESSUS_DEB_URL="PASTE_URL_HERE"
cd /tmp
wget -O nessus.deb "$NESSUS_DEB_URL"


Install the package:

sudo dpkg -i /tmp/nessus.deb || sudo apt -f install -y


Start & enable the service:

sudo systemctl enable --now nessusd
sudo systemctl status nessusd --no-pager


Allow the web UI port (8834) through firewall:

# If using ufw
sudo ufw allow 8834/tcp
# Or with iptables (example)
sudo iptables -I INPUT -p tcp --dport 8834 -j ACCEPT


Register Nessus (one-time; replace ACTIVATION_CODE):

sudo /opt/nessus/sbin/nessuscli fetch --register ACTIVATION_CODE
sudo systemctl restart nessusd


Force plugin & core update:

sudo /opt/nessus/sbin/nessuscli update --all
sudo systemctl restart nessusd


Open the web UI in a browser:

https://<your_server_ip>:8834


Accept the self-signed cert (browser warning), then complete the on-screen setup: create the admin user and finish initialization.

3 — Two reusable scripts
Script A — install + start (save as nessus-install.sh)
#!/bin/bash
# nessus-install.sh
# Usage: bash nessus-install.sh PASTE_URL_HERE

set -e

if [ -z "$1" ]; then
  echo "Usage: $0 <NESSUS_DEB_URL>"
  exit 1
fi

URL="$1"
TMP="/tmp/nessus.deb"

echo "[*] Updating apt..."
sudo apt update

echo "[*] Installing helpers..."
sudo apt install -y wget curl apt-transport-https

echo "[*] Downloading Nessus package..."
wget -O "$TMP" "$URL"

echo "[*] Installing Nessus..."
sudo dpkg -i "$TMP" || sudo apt -f install -y

echo "[*] Enabling and starting nessusd..."
sudo systemctl enable --now nessusd
sudo systemctl status nessusd --no-pager

echo "[*] Done. Open: https://<your_ip>:8834"


Run:

chmod +x nessus-install.sh
./nessus-install.sh "PASTE_URL_HERE"

Script B — register & update (save as nessus-register-update.sh)
#!/bin/bash
# nessus-register-update.sh
# Usage: bash nessus-register-update.sh ACTIVATION_CODE

set -e

if [ -z "$1" ]; then
  echo "Usage: $0 <ACTIVATION_CODE>"
  exit 1
fi

CODE="$1"

echo "[*] Registering Nessus..."
sudo /opt/nessus/sbin/nessuscli fetch --register "$CODE"

echo "[*] Updating plugins & core..."
sudo /opt/nessus/sbin/nessuscli update --all

echo "[*] Restarting service..."
sudo systemctl restart nessusd

echo "[*] Check status and logs:"
echo "  sudo systemctl status nessusd --no-pager"
echo "  tail -f /opt/nessus/var/nessus/logs/nessusd.messages"


Run:

chmod +x nessus-register-update.sh
./nessus-register-update.sh "ACTIVATION_CODE"

4 — Quick troubleshooting (if web UI “New Scan” or templates are missing)

Confirm registration:

sudo /opt/nessus/sbin/nessuscli fetch --check


Check plugin update status:

sudo /opt/nessus/sbin/nessuscli update --all


Tail the Nessus log while reproducing the UI problem:

tail -f /opt/nessus/var/nessus/logs/nessusd.messages


Browser fixes:

Use Chrome/Chromium (some Firefox builds on Kali can misrender).

Clear cache and hard refresh: Ctrl+Shift+R.

Open Developer Console (F12) → Console tab and look for JS errors.

Ensure the logged-in user is Admin in Nessus (Settings → Users).

Wait after update: plugin processing can take several minutes (watch the log).

If nothing helps, restart Nessus and retry:

sudo systemctl restart nessusd

5 — Uninstall (if needed)
sudo systemctl stop nessusd
sudo dpkg -r nessus
sudo rm -rf /opt/nessus /var/nessus /etc/nessus

6 — Notes & security

Keep your activation code secret — treat like a license key.

Nessus will use a self-signed cert by default. For production, replace with a proper TLS cert.

If you plan automated installs across machines, keep nessus-register-update.sh secure (do not commit activation codes to public repos).
