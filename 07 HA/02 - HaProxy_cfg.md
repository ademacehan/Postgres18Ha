
```bash
cket /var/lib/haproxy/stats

defaults
    log                     global
    option                  dontlognull
    retries                 3
    timeout connect         10m
    timeout client          10m
    timeout server          10m
    timeout check           30s
    maxconn                 15000

listen stats
    mode http
    bind *:8000
    stats enable
    stats uri /

# ------------------------------------------------------------------------------------------------------
# ------------------------------------------------------------------------------------------------------
# ------------------------------------------------------------------------------------------------------
# ----------------------------------- master port bilgileri -------------------------------------------
# ------------------------------------------------------------------------------------------------------
# ------------------------------------------------------------------------------------------------------
# ------------------------------------------------------------------------------------------------------

# -------------------------------
# Primary port 5000
# -------------------------------
listen postgres-cluster1-master
    bind *:5000
    mode tcp
    option tcplog
    option httpchk GET /primary
    http-check expect status 200
    default-server inter 30s fall 3 rise 2 on-marked-down shutdown-sessions
    # Patroni REST API portu 8008 üzerinden health check
    # PostgreSQL portu 6432 üzerinden gerçek trafik
    server pg_node1 172.20.38.11:6432 maxconn 1000 check port 8008 send-proxy-v2
    server pg_node2 172.20.38.12:6432 maxconn 1000 check port 8008 send-proxy-v2
    server pg_node3 172.20.38.13:6432 maxconn 1000 check port 8008 send-proxy-v2

# -------------------------------
# Primary port 6000
# -------------------------------
listen postgres-cluster2-master
    bind *:6000
    mode tcp
    option tcplog
    option httpchk GET /primary
    http-check expect status 200
    default-server inter 30s fall 3 rise 2 on-marked-down shutdown-sessions
    # Patroni REST API portu 8008 üzerinden health check
    # PostgreSQL portu 6432 üzerinden gerçek trafik
    server pg_node1 172.20.38.14:6432 maxconn 1000 check port 8008 send-proxy-v2
    server pg_node2 172.20.38.15:6432 maxconn 1000 check port 8008 send-proxy-v2
    server pg_node3 172.20.38.16:6432 maxconn 1000 check port 8008 send-proxy-v2

# -------------------------------
# Primary port 7000
# -------------------------------
listen postgres-cluster3-master
    bind *:7000
    mode tcp
    option tcplog
    option httpchk GET /primary
    http-check expect status 200
    default-server inter 30s fall 3 rise 2 on-marked-down shutdown-sessions
    # Patroni REST API portu 8008 üzerinden health check
    # PostgreSQL portu 6432 üzerinden gerçek trafik
    server pg_node1 172.20.38.17:6432 maxconn 1000 check port 8008 send-proxy-v2
    server pg_node2 172.20.38.18:6432 maxconn 1000 check port 8008 send-proxy-v2
    server pg_node3 172.20.38.19:6432 maxconn 1000 check port 8008 send-proxy-v2
# ------------------------------------------------------------------------------------------------------
# ------------------------------------------------------------------------------------------------------
# ------------------------------------------------------------------------------------------------------
# ----------------------------------- replica port bilgileri -------------------------------------------
# ------------------------------------------------------------------------------------------------------
# ------------------------------------------------------------------------------------------------------
# ------------------------------------------------------------------------------------------------------

# -------------------------------
# Replica port 5001
# -------------------------------
listen replicas-cluster1
    bind *:5001
    mode tcp
    option tcplog
    balance roundrobin
    option httpchk GET /replica
    http-check expect status 200
    default-server inter 30s fall 3 rise 2 on-marked-down shutdown-sessions
    server pg_node1 172.20.38.11:6432 maxconn 1000 check port 8008 send-proxy-v2
    server pg_node2 172.20.38.12:6432 maxconn 1000 check port 8008 send-proxy-v2
    server pg_node3 172.20.38.13:6432 maxconn 1000 check port 8008 send-proxy-v2

# -------------------------------
# Replica port 6001
# -------------------------------
listen replicas-cluster2
    bind *:6001
    mode tcp
    option tcplog
    balance roundrobin
    option httpchk GET /replica
    http-check expect status 200
    default-server inter 30s fall 3 rise 2 on-marked-down shutdown-sessions
    server pg_node1 172.20.38.14:6432 maxconn 1000 check port 8008 send-proxy-v2
    server pg_node2 172.20.38.15:6432 maxconn 1000 check port 8008 send-proxy-v2
    server pg_node3 172.20.38.16:6432 maxconn 1000 check port 8008 send-proxy-v2

# -------------------------------
# Replica port 7001
# -------------------------------
listen replicas-cluster3
    bind *:7001
    mode tcp
    option tcplog
    balance roundrobin
    option httpchk GET /replica
    http-check expect status 200
    default-server inter 30s fall 3 rise 2 on-marked-down shutdown-sessions
    server pg_node1 172.20.38.17:6432 maxconn 1000 check port 8008 send-proxy-v2
    server pg_node2 172.20.38.18:6432 maxconn 1000 check port 8008 send-proxy-v2
    server pg_node3 172.20.38.19:6432 maxconn 1000 check port 8008 send-proxy-v2

# ------------------------------------------------------------------------------------------------------
# ------------------------------------------------------------------------------------------------------
# ------------------------------------------------------------------------------------------------------
# ----------------------------------- sync port bilgileri -------------------------------------------
# ------------------------------------------------------------------------------------------------------
# ------------------------------------------------------------------------------------------------------
# ------------------------------------------------------------------------------------------------------

# -------------------------------
# Sync port 5003
# -------------------------------
listen sync-cluster1
    bind *:5003
    mode tcp
    option tcplog
    balance roundrobin
    option httpchk GET /sync
    http-check expect status 200
    default-server inter 30s fall 3 rise 2 on-marked-down shutdown-sessions
    server pg_node1 172.20.38.11:6432 maxconn 1000 check port 8008 send-proxy-v2
    server pg_node2 172.20.38.12:6432 maxconn 1000 check port 8008 send-proxy-v2
    server pg_node3 172.20.38.13:6432 maxconn 1000 check port 8008 send-proxy-v2

# -------------------------------
# Sync port 6003
# -------------------------------
listen sync-cluster2
    bind *:6003
    mode tcp
    option tcplog
    balance roundrobin
    option httpchk GET /sync
    http-check expect status 200
    default-server inter 30s fall 3 rise 2 on-marked-down shutdown-sessions
    server pg_node1 172.20.38.14:6432 maxconn 1000 check port 8008 send-proxy-v2
    server pg_node2 172.20.38.15:6432 maxconn 1000 check port 8008 send-proxy-v2
    server pg_node3 172.20.38.16:6432 maxconn 1000 check port 8008 send-proxy-v2

# -------------------------------
# Sync port 7003
# -------------------------------
listen sync-cluster3
    bind *:7003
    mode tcp
    option tcplog
    balance roundrobin
    option httpchk GET /sync
    http-check expect status 200
    default-server inter 30s fall 3 rise 2 on-marked-down shutdown-sessions
    server pg_node1 172.20.38.17:6432 maxconn 1000 check port 8008 send-proxy-v2
    server pg_node2 172.20.38.18:6432 maxconn 1000 check port 8008 send-proxy-v2
    server pg_node3 172.20.38.19:6432 maxconn 1000 check port 8008 send-proxy-v2
```

send-proxy-v2 client adresini pgbouncer'a göndermek için eklendi.
