# 1) Gør systemet opdateret
sudo apt update
sudo apt upgrade -y

# 2) Installer nødvendige værktøjer
sudo apt install -y software-properties-common build-essential

# 3) Tilføj GNS3 PPA (officiel)
sudo add-apt-repository ppa:gns3/ppa -y
sudo apt update

# 4) Installer GNS3 server (og GUI hvis du ønsker)
# Hvis dette er en headless server (ingen desktop) installer kun server:
sudo apt install -y gns3-server ubridge dynamips vpcs

# Hvis du vil have GUI på maskinen (kun hvis du har desktop):
# sudo apt install -y gns3-gui

# 5) (Valgfrit) Hvis du vil køre som 'gns3' systembruger:
sudo useradd -m -s /bin/bash gns3 || true
sudo usermod -aG libvirt,plugdev,gns3 $USER 2>/dev/null || true

# 6) Opret/ret config (vælg 0.0.0.0 for at lytte på alle interfaces og auth=false til test)
mkdir -p ~/.config/GNS3
cat > ~/.config/GNS3/gns3_server.conf <<'EOF'
[Server]
host = 0.0.0.0
port = 3080
auth = false
EOF

# 7) Start serveren manuelt (til test)
pkill gns3server 2>/dev/null || true
gns3server --host 0.0.0.0 --port 3080 &

# 8) Tjek at den lytter
ss -tulnp | grep 3080

# 9) Test fra samme maskine eller en klient:
curl http://127.0.0.1:3080/v2/version
# eller fra en anden pc:
# curl http://<server-ip>:3080/v2/version
