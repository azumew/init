# 概要
自宅で運用してるラズパイ4Bのセットアップ内容

## 初期セットアップ
```
sudo su -
apt update 
apt upgrade -y
apt install -y dnsutils snapd
snap install core snap-store
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
sysctl -p
exit
```

## OpenMediaVaultセットアップ
```
wget -O - https://github.com/OpenMediaVault-Plugin-Developers/installScript/raw/master/install | sudo bash
```
1. 勝手に再起動されるので初期設定に移る
   - URL: http://raspberrypi.local
   - ID: admin
   - PASS: openmediavault
2. omv-extrasから「openmediavault-compose」をインストール
3. サービス→Compose→Filesから「Portainer」をインストール
   - 参考：https://saturday-in-the-park.netlify.app/AlpineLinux/Openmediavault/

## DNS設定変更
```
sudo su -
vi /etc/netplan/99-azumew.yaml
```
```
network:
  ethernets:
    eth0:
      match:
        macaddress: xx:xx:xx:xx:xx:xx
      accept-ra: true
      dhcp4: yes
      dhcp4-overrides:
        use-dns: true
        use-domains: true
      dhcp6: yes
      dhcp6-overrides:
        use-dns: true
        use-domains: true
      nameservers:
        addresses:
          - 1.1.1.1
```
```
netplan --debug generate
netplan --debug apply
```

## stub listener やめさせる
```
vi /etc/systemd/resolved.conf.d/adguardhome.conf
```
```
[Resolve]
DNS=127.0.0.1
DNSStubListener=no
```
```
mv /etc/resolv.conf /etc/resolv.conf.`date -I`
systemctl restart systemd-resolved
```

## AdguardHomeセットアップ
1. portainerからadguard-homeコンテナを起動
   - http://raspberrypi.local:9000/
   - Dockerfile：raspberrypi/docker/adguard-home.yml

2. 起動後以下URLにアクセスしセットアップ
- http://raspberrypi.local:3000/

### アップストリームDNSサーバー
```
https://cloudflare-dns.com/dns-query
https://dns.google/dns-query
https://unfiltered.adguard-dns.com/dns-query
https://dns10.quad9.net/dns-query
```

# tailscaleセットアップ
```
curl -fsSL https://tailscale.com/install.sh | sh
tailscale up --advertise-routes=192.168.1.0/24
```

# 廃止
## AdguardHomeセットアップ
```
sudo snap install adguard-home
```
