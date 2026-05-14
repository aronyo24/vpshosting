Fix 5 — Reinstall pgAdmin Clean

If nothing works, clean reinstall:
bash# Remove old pgAdmin
```
sudo apt remove pgadmin4-web -y
sudo rm -rf /var/lib/pgadmin4
sudo rm -rf /var/log/pgadmin4

# Reinstall
sudo apt install pgadmin4-web -y
sudo /usr/pgadmin4/bin/setup-web.sh
```
