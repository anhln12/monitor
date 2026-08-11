PMM

Dùng chuyên sâu cho:
- MySQL performance
- Query Analytics
- Slow query
- InnoDB
- Locks
- Buffer Pool
- Connections
- QPS/TPS
- Replication
- Query latency
- Database performance troubleshooting


Kiến trúc triển khai

                  Internet/LAN
                       │
                       │ HTTPS 443
                       ▼
                ┌──────────────┐
                │              │
                │    Nginx     │
                │     :443     │
                └──────┬───────┘
                       │
                       │ HTTPS 8443
                       ▼
             ┌──────────────────┐
             │ PMM Server VM    │
             │ 10.50.1.150      │
             │                  │
             │ Nginx :8443      │
             │       │          │
             │       ▼          │
             │ pmm-managed      │
             │ 127.0.0.1:7771   │
             └──────────────────┘
                       ▲
                       │
                       │ HTTPS 443
                       │
             ┌─────────┴─────────┐
             │ server01          │
             │ MySQL + pmm-agent │
             └───────────────────┘



Cài đặt PMM Client trên MySQL

1. Cài PMM Client
```
apt update
apt install -y wget
wget https://repo.percona.com/apt/percona-release_latest.generic_all.deb
dpkg -i percona-release_latest.generic_all.deb
```

enable repository PMM 3:
```
percona-release enable pmm3-client
```

Cài client
```
apt update
apt install -y pmm-client
```

Kiểm tra
```
pmm-admin --version
```

2. Đăng ý Mysql với PMM Server

Vì PMM Server của bạn nằm sau Nginx port 433
```
pmm-admin config \
  --server-url=https://admin:YOUR_PMM_PASSWORD@pmm.example.com

root@server01:~# pmm-admin config --server-url=https://admin:YOUR_PMM_PASSWORD@pmm.example.com
Checking local pmm-agent status...
pmm-agent is running.
Registering pmm-agent on PMM Server...
Registered.
Configuration file /usr/local/percona/pmm/config/pmm-agent.yaml updated.
Reloading pmm-agent configuration...
Configuration reloaded.
Checking local pmm-agent status...
pmm-agent is running.

```

Kiêm tra:
```
pmm-admin status

**root@server01:~# pmm-admin status
Agent ID : 5be3661e-198e-4c6e-a9b4-8d06495b80ea
Node ID  : 76937319-5c8b-411d-88cf-ab9f3c907a85
Node name: Parrot-server01

PMM Server:
        URL    : https://pmm.codeinet.com:443/
        Version: 3.9.0

PMM Client:
        Connected        : true
        Time drift       : -26.28831ms
        Latency          : 934.225µs
        Connection uptime: 100
        pmm-admin version: 3.9.0
        pmm-agent version: 3.9.0
Agents:
        4dc89454-ac31-4777-a08f-cfea8b215848 node_exporter                  Running        42000
        736fba7c-db55-4b89-a9e0-33ecd2233b50 mysqld_exporter                Running        42002
        cf62d37a-45c6-4ef9-b15c-be05de7ffb95 mysql_slowlog_agent            Waiting        0
        d9355c7c-f308-477c-a46f-2e19ab103202 vmagent                        Running        42001**
```

3. Tạo user Mysql cho PMM

```
CREATE USER 'pmm'@'localhost'
IDENTIFIED BY 'StrongPMMPassword'
WITH MAX_USER_CONNECTIONS 10;

GRANT SELECT, PROCESS, REPLICATION CLIENT, RELOAD
ON *.* TO 'pmm'@'localhost';

SHOW GRANTS FOR 'pmm'@'localhost';
```

4. Add MySQL vào PMM
```
pmm-admin add mysql \
  MySQL-Production \
  127.0.0.1:3306 \
  --username=pmm \
  --password='StrongPMMPassword' \
  --query-source=slowlog \
  --environment=production

root@server01:~# pmm-admin add mysql \
  MySQL-Production \
  127.0.0.1:3306 \
  --username=pmm \
  --password='YOUR_PMM_PASSWORD' \
  --query-source=slowlog \
  --environment=production
MySQL Service added.
Service ID  : 62498994-903c-473a-b99b-de07c424ca9a
Service name: MySQL-Production

Table statistics collection enabled (the limit is 1000, the actual table count is 445).
```

Kiểm tra lại
```
pmm-admin list
```

Xem logs
```
journalctl -u pmm-agent -f
```

Bây giờ kiểm tra log của mysql_slowlog_agent
```
journalctl -u pmm-agent --since "15 minutes ago" --no-pager | \
grep -Ei 'slowlog|slow.log|qan|error|permission|mysql'
```
