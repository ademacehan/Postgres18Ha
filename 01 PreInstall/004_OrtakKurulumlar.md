# Tüm Sunucularda Yapılacak Ortak Kurulumlar

## İşletim Sisteminin güncellenmesi

```bash
sudo dnf update -y
```

## Netwotk tool ve vim kurulması
```bash
sudo dnf install -y telnet net-tools
```

## git kurulumu
```bash
dnf install git -y
```

## go nun kurulması
```bash
wget https://go.dev/dl/go1.23.2.linux-amd64.tar.gz
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.23.2.linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
source ~/.bashrc
```

## Türkçe dil paketinin yüklenmesi
```bash
sudo dnf install langpacks-tr
```