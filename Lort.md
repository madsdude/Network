# 1) Tjek at Docker er installeret og kører
docker --version || sudo apt install -y docker.io
sudo systemctl enable --now docker
sudo systemctl status docker

# 2) Giv gns3-brugeren adgang til docker-gruppen
sudo groupadd docker 2>/dev/null || true
sudo usermod -aG docker gns3      # <- brugeren der kører gns3server
sudo usermod -aG docker $USER     # (valgfrit: også din nuværende bruger)

# 3) Genindlæs gruppemedlemskab for gns3 (uden reboot)
sudo -iu gns3                     # skift til gns3-brugerens shell
newgrp docker                     # aktiver gruppen i den session
docker ps                         # skal virke uden "permission denied"
exit

# 4) Genstart GNS3-serveren
pkill gns3server || true
gns3server --host 0.0.0.0 --port 3080 &
