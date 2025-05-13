<p align="center">
  <img height="100" height="auto" src="https://github.com/user-attachments/assets/d2baddd0-2760-46af-b8ca-ce3a5cb1a678">
</p>

# Pipe Testnet

Official documentation:
>- [Validator setup instructions](https://docs.pipe.network/devnet-2)

Explorer:
>- [Explorer](https://dashboard.pipenetwork.com/node-lookup)

### Minimum Hardware Requirements
 - 4x CPUs; the faster clock speed the better
 - 6GB RAM
 - 100GB of storage (SSD or NVME)

### Recommended Hardware Requirements 
 - 8x CPUs; the faster clock speed the better
 - 12GB RAM
 - 1TB of storage (SSD or NVME)

 - Ubuntu 22.04

Ставим ноду:

``sudo apt update && sudo apt upgrade -y``

``sudo apt install curl iptables build-essential git wget jq make gcc nano tmux htop nvme-cli pkg-config libssl-dev tar clang bsdmainutils ncdu unzip libleveldb-dev lz4 screen bc fail2ban -y``

``mkdir /root/pop``

``cd /root/pop``

``curl -L -o pop "https://dl.pipecdn.app/v0.2.8/pop"``

``chmod +x pop``

``mkdir download_cache``

``nano /etc/systemd/system/pop.service``

Вписываем данные заменяя ```<SOLANA ADDRESS>``` на свой адрес кошелька Solana:

```rrr
[Unit]
Description=Pipe POP Node Service
After=network.target
Wants=network-online.target

[Service]
AmbientCapabilities=CAP_NET_BIND_SERVICE
CapabilityBoundingSet=CAP_NET_BIND_SERVICE
User=root
Group=root
ExecStart=/root/pop/pop \
    --ram 4 \
    --pubKey <SOLANA ADDRESS> \
    --max-disk 100 \
    --cache-dir /root/pop/download_cache \
    --no-prompt
Restart=always
RestartSec=5
LimitNOFILE=65536
LimitNPROC=4096
StandardOutput=journal
StandardError=journal
SyslogIdentifier=pop-node
WorkingDirectory=/root/pop

[Install]
WantedBy=multi-user.target
```

Сохранить CTRL + O

Выйти CTRL + X

``sudo ufw enable``

``sudo ufw allow 22``

``sudo ufw allow 80``

``sudo ufw allow 443``

``sudo ufw allow 8003``

``ufw reload``

``systemctl daemon-reload``

``systemctl enable pop``

``systemctl start pop``

Сохраните эти данные:

``cat node_info.json``

Полезные команды:

Просмотр логов:

``journalctl -u pop -f``

Статус:

``./pop --status``

# Pipe Migration

### Recommended Hardware Requirements 
 - 4x CPUs; the faster clock speed the better
 - 16GB RAM
 - 100GB of storage (SSD or NVME)

 - Ubuntu 24.04

```
sudo apt update && sudo apt upgrade -y
```

```
sudo apt install curl iptables build-essential git wget jq make gcc nano tmux htop nvme-cli pkg-config libssl-dev tar clang bsdmainutils ncdu unzip libleveldb-dev lz4 screen bc fail2ban ca-certificates -y
```

```rrr
sudo bash -c 'cat > /etc/sysctl.d/99-popcache.conf << EOL
net.ipv4.ip_local_port_range = 1024 65535
net.core.somaxconn = 65535
net.ipv4.tcp_low_latency = 1
net.ipv4.tcp_fastopen = 3
net.ipv4.tcp_slow_start_after_idle = 0
net.ipv4.tcp_window_scaling = 1
net.ipv4.tcp_wmem = 4096 65536 16777216
net.ipv4.tcp_rmem = 4096 87380 16777216
net.core.wmem_max = 16777216
net.core.rmem_max = 16777216
EOL'
```

```
sudo sysctl -p /etc/sysctl.d/99-popcache.conf
```

```rrr
sudo bash -c 'cat > /etc/security/limits.d/popcache.conf << EOL
*    hard nofile 65535
*    soft nofile 65535
EOL'
exit
```

Перезагружаем сервер через панель хостера.

```
sudo ufw allow ssh
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
```

```
sudo mkdir -p /opt/popcache/logs
cd /opt/popcache
```

```
wget https://download.pipe.network/static/pop-v0.3.0-linux-x64.tar.gz
sudo tar -xzf pop-v0.3.0-linux-x64.tar.gz
chmod 755 /opt/popcache/pop
```

Редактируем конфиг:
```r
pop_name: Придумайте имя узла (заменяем Name на свой).
pop_location: Город и страна вашего VPS (заменяем City, Country).
invite_code: Ваш код приглашения из письма (заменяем Code на свой).


node_name: Придумайте имя узла (заменяем NodeName на свой).
name: Ваше личное имя или имя организации (заменяем Name).
email: Ваш адрес электронной почты (заменяем Mail).
website: Ваш сайт (необязательно).
discord: Ваше имя пользователя Discord (заменяем Discord).
telegram: Ваш Telegram (заменяем Telegram).
solana_pubkey: Адрес вашего кошелька Solana для получения вознаграждений (заменяем Wallet).
```

```
nano /opt/popcache/config.json
```

```rrr
{
  "pop_name": "Name",
  "pop_location": "City, Country",
  "invite_code": "Code",
  "server": {
    "host": "0.0.0.0",
    "port": 443,
    "http_port": 80,
    "workers": 40
  },
  "cache_config": {
    "memory_cache_size_mb": 4096,
    "disk_cache_path": "./cache",
    "disk_cache_size_gb": 100,
    "default_ttl_seconds": 86400,
    "respect_origin_headers": true,
    "max_cacheable_size_mb": 1024
  },
  "api_endpoints": {
    "base_url": "https://dataplane.pipenetwork.com"
  },
  "identity_config": {
    "node_name": "NodeName",
    "name": "Name",
    "email": "Mail",
    "website": "Website",
    "discord": "Discord",
    "telegram": "Telegram",
    "solana_pubkey": "Wallet"
  }
}
```

Cохраняем CTRL + O - Enter, CTRL + X

```
sudo nano /etc/systemd/system/popcache.service
```

```rrr
[Unit]
Description=POP Cache Node
After=network.target

[Service]
Type=simple
User=root
Group=root
WorkingDirectory=/opt/popcache
ExecStart=/opt/popcache/pop
Restart=always
RestartSec=5
LimitNOFILE=65535
StandardOutput=append:/opt/popcache/logs/stdout.log
StandardError=append:/opt/popcache/logs/stderr.log
Environment=POP_CONFIG_PATH=/opt/popcache/config.json

[Install]
WantedBy=multi-user.target
```

```
sudo systemctl daemon-reload
sudo systemctl enable popcache
sudo systemctl start popcache
```

Полезные команды:

Проверить статус:

```
sudo systemctl status popcache
```

Просмотр логов:

```
sudo journalctl -u popcache -f
```

```
tail -f /opt/popcache/logs/stdout.log
```

```
tail -f /opt/popcache/logs/stderr.log
```

Проверка состояния конечной точки:

```
curl http://localhost/health
```

Проверить состояние ноды:

https://IP/state - вместо IP вводим IP сервера на котором установлена нода и вводим ссылку в браузере












