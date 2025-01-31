# Telegram Daily Bot

Create token via @BotFather

```bash
# check
curl https://api.telegram.org/bot<TOKEN>/getMe

# add bot to group and then get chat id
curl https://api.telegram.org/bot<TOKEN>/getUpdates
```

Setup and run playbook

```bash
# create inventory, and set host
mv ansible/inventory.example ansible/inventory

# create secrets.yml and set bot_token and chat_id
mv ansible/secrets.yml.example ansible/secrets.yml

# update service_name, message, and calendar_on vars
nano ansible/playbook.yml

cd ansible
ansible-playbook playbooks/deploy.yml
```

## Manual Setup

```bash
sudo mkdir -p /opt/telegram

sudo nano /opt/telegram/send_daily_message.sh
sudo chmod +x /opt/telegram/send_daily_message.sh

sudo nano /etc/systemd/system/telegram_daily.service
sudo nano /etc/systemd/system/telegram_daily.timer

# test script
sudo systemctl start telegram_daily.service
sudo journalctl -u telegram_daily.service --no-pager --since "1 hour ago"

# run timer
sudo systemctl daemon-reload
sudo systemctl enable --now telegram_daily.timer
```

### /opt/telegram/send_daily_message.sh

```bash
#!/bin/bash

BOT_TOKEN="YOUR_BOT_TOKEN_HERE"
CHAT_ID="YOUR_CHAT_ID_HERE"
MESSAGE="<b>Daily.</b>%0AFormat:%0AWhat did you do yesterday?%0AWhat will you do today?%0AWhat problems are there?%0A%0A<i>Reply to this message to answer.</i>"

curl -s -X POST "https://api.telegram.org/bot$BOT_TOKEN/sendMessage" \
     -d "chat_id=$CHAT_ID" \
     -d "text=$MESSAGE" \
     -d "parse_mode=HTML"
```

### /etc/systemd/system/telegram_daily.service

```bash
[Unit]
Description=Send Telegram Daily Message
After=network.target

[Service]
Type=oneshot
ExecStart=/opt/telegram/send_daily_message.sh
StandardOutput=journal
StandardError=journal
```

### /etc/systemd/system/telegram_daily.timer

```bash
[Unit]
Description=Run Telegram Service at 10:00 Moscow Time on Workdays

[Timer]
OnCalendar=Mon..Fri 07:00:00 UTC
Persistent=true

[Install]
WantedBy=timers.target
```
