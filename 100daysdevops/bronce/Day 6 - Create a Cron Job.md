```
sudo yum update && sudo yum install cronie -y

sudo crontab -e
```

## Agregar
```
*/5 * * * * echo hello > /tmp/cron_text
```

## Verificar
```
sudo crontab -l
```