# Pipe Migration 2

```
sudo apt update && sudo apt upgrade -y
```

```
sudo apt install curl iptables build-essential git wget jq make gcc nano tmux htop nvme-cli pkg-config libssl-dev tar clang bsdmainutils ncdu unzip libleveldb-dev lz4 screen bc fail2ban ca-certificates -y
```

```
curl -fsSL https://get.docker.com | sh
```

```
systemctl enable docker
systemctl start docker
```

```
cat <<EOF | sudo tee /etc/sysctl.d/99-popcache.conf
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
EOF
```

```
sudo sysctl -p /etc/sysctl.d/99-popcache.conf
```

```
cat <<EOF | sudo tee /etc/security/limits.d/popcache.conf
*    hard nofile 65535
*    soft nofile 65535
EOF
```

```
sudo mkdir -p /opt/popcache
```

```
cd /opt/popcache
```

```
sudo chmod 777 /opt/popcache
```

```
wget -q https://download.pipe.network/static/pop-v0.3.1-linux-x64.tar.gz
```

```
tar -xzf pop-v0.3.1-linux-*.tar.gz
```

```
chmod 755 pop
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
cat <<EOF > config.json
{
  "pop_name": "POP_NAME",
  "pop_location": "POP_LOCATION",
  "server": {
    "host": "0.0.0.0",
    "port": 443,
    "http_port": 80,
    "workers": 0
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
    "node_name": "NODE_NAME",
    "name": "NAME",
    "email": "EMAIL",
    "website": "WEBSITE",
    "discord": "DISCORD",
    "telegram": "TELEGRAM",
    "solana_pubkey": "SOLANA_PUBKEY"
  }
}
EOF
```

```
cat <<EOF > Dockerfile
FROM ubuntu:24.04

RUN apt update && apt install -y \\
    ca-certificates \\
    curl \\
    libssl-dev \\
    && rm -rf /var/lib/apt/lists/*

WORKDIR /opt/popcache

COPY pop .
COPY config.json .

RUN chmod +x ./pop

CMD ["./pop", "--config", "config.json"]
EOF
```

```
docker build -t popnode .
```

Редактируем конфиг:
```r
invite_code: Ваш код приглашения из письма (заменяем Code на свой).

```

```
docker run -d \
  --name popnode \
  -p 80:80 \
  -p 443:443 \
  -v /opt/popcache:/app \
  -w /app \
  -e POP_INVITE_CODE=INVITE_CODE \
  --restart unless-stopped \
  popnode
```

Полезные команды:

Просмотр логов -

```
docker logs -f popnode
```

$IP - заменяем на IP вашего сервера где стоит нода

Проверьте состояние в браузере: http://$IP/health

Проверьте состояние безопасности: https://$IP/state














