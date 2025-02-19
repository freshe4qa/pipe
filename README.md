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

``curl -L -o pop "https://dl.pipecdn.app/v0.2.6/pop"``

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

