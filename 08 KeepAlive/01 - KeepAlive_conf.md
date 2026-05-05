
```bash 
vi keepalived.conf
```

```bash
global_defs {
    router_id ha01
    script_user root
    enable_script_security

}

vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 100
    advert_int 1

    # VIP tanımı
    virtual_ipaddress {
        172.20.38.21/24
    }

    # Gratuitous ARP ayarları → istemciler yeni MAC'i hemen öğrenir
    garp_master_delay 1
    garp_master_repeat 5

    # Notify script → sadece log tutar, HAProxy restart etmez
    notify_master "/etc/keepalived/notify.sh master"
    notify_backup "/etc/keepalived/notify.sh backup"
    notify_fault  "/etc/keepalived/notify.sh fault"
}
```