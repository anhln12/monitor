1. Trên server chạy proxysql kiểm tra xem đã có cài proxysql
```
proxysql --version
systemctl status proxysql --no-pager
```

Kiểm tra Admin interface
```
mysql -u admin -p -h 127.0.0.1 -P 6032
```

Lấy thông tin credentials
```
SELECT * FROM global_variables
WHERE variable_name IN (
    'admin-admin_credentials',
    'admin-mysql_ifaces'
);
```

2. Kiểm tra PMM Client
```
pmm-admin status
pmm-admin list
```

Bạn đã có PMM Client và ProxySQL trước đó, nên nếu ProxySQL service đã tồn tại thì kiểm tra
```
pmm-admin list | grep -i proxysql
```

3. Add ProxySQL vào PMM
```
pmm-admin add proxysql \
  --username=admin \
  --password='PASSWORD_PROXYSQL' \
  --host=127.0.0.1 \
  --port=6032 \
  --service-name=proxysql-prod
```

Sau đó
```
pmm-admin list
```

Kết quả mong muốn dạng
```
Service type  Service name       Address
PROXYSQL      proxysql-prod      127.0.0.1:6032
```

4. Kiểm tra metric
Kiểm tra metric
```
curl -s http://127.0.0.1:42000/metrics | grep -i proxysql | head
```

5. Trong PMM

Bạn có thể theo dõi các metric quan trọng như:
- Connections
- Connection pool
- Connnection erros
- Queries/sec
- Query latency
- traffic
- Backend Mysql Status
- Hostgroup
- Connection utilization
- Queries routed tới Primary/Replica
- Backend connection errors

Monitor tối thiểu:
```
ProxySQL
├── Process / Service
│   ├── proxysql up/down
│   ├── CPU
│   ├── Memory
│   └── Disk
│
├── Connection
│   ├── Client connections
│   ├── Backend connections
│   └── Connection errors
│
├── Query
│   ├── Queries/sec
│   ├── Latency
│   ├── Query errors
│   └── Query processing time
│
├── Hostgroup
│   ├── ONLINE
│   ├── SHUNNED
│   ├── OFFLINE_SOFT
│   └── OFFLINE_HARD
│
└── MySQL backend
    ├── Primary
    ├── Replica
    ├── Replication lag
    └── Backend health
```




