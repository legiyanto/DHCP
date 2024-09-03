#!/bin/bash

# Fungsi untuk mencetak teks di tengah layar
print_center() {
  local termwidth
  local padding
  local message="$1"
  local color="$2"
  termwidth=$(tput cols)
  padding=$(( (termwidth - ${#message}) / 2 ))
  printf "%s%${padding}s%s%s\n" "$color" "" "$message" "$(tput sgr0)"
}
# Banner Teks
print_center "    __               _                   __      "
print_center "   / /   ___  ____ _(_)_  ______ _____  / /_____ "
print_center "  / /   / _ \\/ __ \`/ / / / / __ \`/ __ \\/ __/ __ \\"
print_center " / /___/  __/ /_/ / / /_/ / /_/ / / / / /_/ /_/ /"
print_center "/_____/\___/\\__, /_/\\__, /\\__,_/_/ /_/\\__/\\____/ "
print_center "           /____/  /____/                         "
print_center ""
print_center ""
print_center ""
print_center "+-+-+-+ +-+-+-+-+ +-+ +-+-+-+-+-+-+-+"
print_center "|T|K|J| |S|M|K|N| |5| |B|A|N|D|U|N|G|"
print_center "+-+-+-+ +-+-+-+-+ +-+ +-+-+-+-+-+-+-+"
print_center ""
print_center ""
print_center ""

# Jeda waktu 2 detik
sleep 2


# Warna hijau untuk pesan sukses
GREEN='\033[0;32m'
NC='\033[0m' # No Color

# Mengganti repository menjadi repository lokal Kartolo Ubuntu 20.04
cat <<EOF > /etc/apt/sources.list
deb http://kartolo.sby.datautama.net.id/ubuntu/ focal main restricted universe multiverse
deb http://kartolo.sby.datautama.net.id/ubuntu/ focal-updates main restricted universe multiverse
deb http://kartolo.sby.datautama.net.id/ubuntu/ focal-security main restricted universe multiverse
deb http://kartolo.sby.datautama.net.id/ubuntu/ focal-backports main restricted universe multiverse
deb http://kartolo.sby.datautama.net.id/ubuntu/ focal-proposed main restricted universe
EOF

# Update sistem dan install ISC DHCP Server, iptables, dan iptables-persistent
apt update &> /dev/null
apt install -y isc-dhcp-server iptables &> /dev/null
echo -e "${GREEN}Berhasil menginstall ISC DHCP Server dan iptables${NC}"

# Install iptables-persistent tanpa prompt interaktif
echo iptables-persistent iptables-persistent/autosave_v4 boolean true | debconf-set-selections
echo iptables-persistent iptables-persistent/autosave_v6 boolean false | debconf-set-selections
apt install -y iptables-persistent &> /dev/null
echo -e "${GREEN}Berhasil menginstall iptables-persistent ${NC}"

# Konfigurasi Netplan
cat <<EOF > /etc/netplan/01-netcfg.yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      addresses:
        - 192.168.100.1/24
EOF

# Terapkan konfigurasi Netplan
netplan apply &> /dev/null
echo -e "${GREEN}Berhasil menerapkan konfigurasi Netplan${NC}"

# Konfigurasi DHCP Server
cat <<EOF > /etc/dhcp/dhcpd.conf
default-lease-time 600;
max-lease-time 7200;
option domain-name "local";
authoritative;

subnet 192.168.100.0 netmask 255.255.255.0 {
  range 192.168.100.2 192.168.100.200;
  option routers 192.168.100.1;
  option subnet-mask 255.255.255.0;
  option broadcast-address 192.168.100.255;
  option domain-name-servers 192.168.100.1,8.8.8.8,8.8.4.4;
  
  # Fixed IP assignment for specific clients based on MAC address
  host client1 {
    hardware ethernet 00:11:22:33:44:55; # Ganti dengan MAC address asli
    fixed-address 192.168.100.10;
  }

  host client2 {
    hardware ethernet 66:77:88:99:AA:BB; # Ganti dengan MAC address asli
    fixed-address 192.168.100.20;
  }
}
EOF

# Tentukan interface DHCP
echo "INTERFACESv4=\"enp0s8\"" > /etc/default/isc-dhcp-server

# Restart DHCP server untuk menerapkan konfigurasi
systemctl restart isc-dhcp-server &> /dev/null
echo -e "${GREEN}Berhasil mengkonfigurasi dan me-restart DHCP server${NC}"

# Aktifkan IP forwarding
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
sysctl -p &> /dev/null
echo -e "${GREEN}Berhasil mengaktifkan IP forwarding${NC}"

# Konfigurasi NAT dengan iptables
iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE &> /dev/null
iptables -A FORWARD -i enp0s3 -o enp0s8 -m state --state RELATED,ESTABLISHED -j ACCEPT &> /dev/null
iptables -A FORWARD -i enp0s8 -o enp0s3 -j ACCEPT &> /dev/null

# Simpan aturan iptables agar tetap ada setelah reboot
iptables-save > /etc/iptables/rules.v4
echo -e "${GREEN}Berhasil mengkonfigurasi iptables dan menyimpan aturan${NC}"

# Restart DHCP server dan network untuk memastikan semua berjalan dengan baik
systemctl restart isc-dhcp-server &> /dev/null
systemctl restart systemd-networkd &> /dev/null
echo -e "${GREEN}Berhasil me-restart layanan jaringan dan DHCP server${NC}"

