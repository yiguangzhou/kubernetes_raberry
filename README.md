# kubernetes_raberry
树莓派安装kubernetes集群
$ sudo dpkg-reconfigure keyboard-configuration

#WiFi connection
/etc/network/interfaces
auto wlan0
iface wlan0 inet manual
wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf

/etc/wpa_supplicant/wpa_supplicant.conf
country=GB
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
network={
    ssid="your ssid"
    psk="your wifi password"
    key_mgmt=WPA-PSK
}

$ wpa_passphrase "you ssid" "your wifi password"

network={
    ssid="you ssid"
    #psk="your wifi password"
    psk=238c7d2c09814cbf8c98a30773e4e2d221a723c38a1d0f0ba1e2e
}
#Static IP address
/etc/dhcpcd.conf
interface wlan0
static ip_address=192.168.1.101/24
static routers=192.168.1.10
static domain_name_servers=62.179.1.63
