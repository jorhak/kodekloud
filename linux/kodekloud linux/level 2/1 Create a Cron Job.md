```
The `Nautilus` system admins team has prepared scripts to automate several day-to-day tasks. They want them to be deployed on all app servers in `Stratos DC` on a set schedule. Before that they need to test similar functionality with a sample cron job. Therefore, perform the steps below:  
  

  

a. Install `cronie` package on all `Nautilus` app servers and start `crond` service.  
  

b. Add a cron `*/5 * * * * echo hello > /tmp/cron_text` for `root` user.
```
# Ingresar al servidor y instalar cronie
```bash
ssh tony@stapp01
ssh steve@stapp02
ssh banner@stapp03
sudo yum install -y cronie
sudo systemctl enable crond
sudo systemctl start crond
sudo systemctl status crond
```
# Programar tarea
```bash
sudo crontab -e
```

```text
*/5 * * * * echo hello > /tmp/cron_text
```
# Verificar
```bash
cat /tmp/cron_text
```