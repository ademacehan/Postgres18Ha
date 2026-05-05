
```bash
vi /etc/keepalived/notify.sh
```

```bash
#!/bin/bash
TYPE=$1
NAME=$2
STATE=$3

case $STATE in
    "MASTER")
        echo "$(date) - Becoming MASTER" >> /var/log/keepalived-notify.log
        # Email gönder veya Slack bildirimi
        ;;
    "BACKUP")
        echo "$(date) - Becoming BACKUP" >> /var/log/keepalived-notify.log
        ;;
    "FAULT")
        echo "$(date) - FAULT detected" >> /var/log/keepalived-notify.log
        ;;
esac

exit 0
```

```bash 
sudo chmod 755 /etc/keepalived/notify.sh
sudo chown root:root /etc/keepalived/notify.sh
sudo chmod +x /etc/keepalived/notify.sh
```