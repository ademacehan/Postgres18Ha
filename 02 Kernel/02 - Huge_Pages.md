## Huge Page hesaplama

Shared Buffer(GB) / 2mb
Varsayılan hugepages = 2mb
```bash
cat <<EOF > /etc/sysctl.d/99-hugepages.conf
vm.nr_hugepages = 17000
vm.hugetlb_shm_group = postgres
EOF
```