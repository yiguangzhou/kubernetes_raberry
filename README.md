# kubernetes_raberry
树莓派安装kubernetes集群


$ sudo dpkg-reconfigure keyboard-configuration

# WiFi connection  
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
# Static IP address  
/etc/dhcpcd.conf  
interface wlan0  
static ip_address=192.168.1.101/24  
static routers=192.168.1.10  
static domain_name_servers=62.179.1.63  

# Hostname
$ sudo rasbpi-config  

# Docker
$ curl -sSL get.docker.com | sh  
$ sudo usermod pi -aG docker  

# Disable Swap
$ sudo dphys-swapfile swapoff  
$ sudo dphys-swapfile uninstall  
$ sudo update-rc.d dphys-swapfile remove  

# Configure CGroups
/boot/cmdline.txt  
cgroup_enable=cpuset cgroup_memory=1  

# Enable SSH
$ sudo raspi-config  
ssh pi@192.168.1.101  

# Install Kubernetes
$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -  
$ echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list  
$ sudo apt-get update -q  
$ sudo apt-get install -qy kubeadm  

# Configure Kubernetes master node
$ sudo kubeadm init --apiserver-advertise-address=192.168.1.101  

# To start using your cluster
mkdir -p $HOME/.kube  
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config  
sudo chown $(id -u):$(id -g) $HOME/.kube/config  
Run "kubectl apply -f [podnetwork].yaml"   
sudo kubeadm join --token c5334d.24da95e5e961a5af 192.168.1.101:6443 --discovery-token-ca-cert-hash sha256:acbae0a1b8a12496cd1de3fb750e36c53179d8a014ed51717fc1b89888c28a66  
$ mkdir -p $HOME/.kube  
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config  
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config  
$ kubectl apply -f https://git.io/weave-kube-1.6  
$ kubectl get nodes  

# Configure Kubernetes nodes
$sudo kubeadm join --token c5334d.24da95e5e961a5af 192.168.1.101:6443 --discovery-token-ca-cert-hash sha256:acbae0a1b8a12496cd1de3fb750e36c53179d8a014ed51717fc1b89888c28a66  
$ kubectl get nodes  

NAME      STATUS    ROLES     AGE       VERSION  
pi01      Ready     master    19h       v1.9.0  
pi02      Ready     <none>    19h       v1.9.0  
pi03      Ready     <none>    19h       v1.9.0  
pi04      Ready     <none>    16h       v1.9.0  
pi05      Ready     <none>    16h       v1.9.0  
pi06      Ready     <none>    16h       v1.9.0  
    
# Configure local computer
$ brew install kubectl  
$ kubectl version  
$ scp pi@192.168.1.101:/home/pi/.kube/config .  
$ mkdir .kube  
$ mv config ./kube  
$ kubectl get nodes  

# Kubernetes dashboard
curl https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/alternative/kubernetes-dashboard-arm.yaml > dashboard.yaml $ kubectl create -f dashboard.yaml  
create-admin-user.yaml:  
apiVersion: v1  
kind: ServiceAccount  
metadata:  
  name: admin-user  
  namespace: kube-system  
  
$ kubectl create -f create-admin-user.yaml  
add-role.yaml:  
 apiVersion: rbac.authorization.k8s.io/v1beta1  
kind: ClusterRoleBinding  
metadata:  
  name: admin-user  
roleRef:  
  apiGroup: rbac.authorization.k8s.io  
  kind: ClusterRole  
  name: cluster-admin  
subjects:  
- kind: ServiceAccount  
  name: admin-user  
  namespace: kube-system  
$ kubectl create -f add-role.yaml    

# Verify dashboard
$ kubectl get svc --namespace kube-system  
NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)         AGE  
kube-dns               ClusterIP   10.96.0.10      <none>        53/UDP,53/TCP   20h  
kubernetes-dashboard   ClusterIP   10.108.75.231   <none>        80/TCP          17h  
    
$ sudo ssh -L 8080:10.108.75.231:80 pi@192.168.1.101  
$ kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')  
Name:         admin-user-token-v454k
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name=admin-user
kubernetes.io/service-account.uid=f64be1d8-e807-11e7-aaa1-b827eb632aa7
Type:  kubernetes.io/service-account-token
Data
====
ca.crt:     1025 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLXY0NTRrIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiJmNjRiZTFkOC1lODA3LTExZTctYWFhMS1iODI3ZWI2MzJhYTciLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06YWRtaW4tdXNlciJ9.n5Jtq0FAy9mi51OGEUIn6WwzyL24Cqy1ewSbeFJU19RG1_bTaKrM9TPEcx6GypANNT2uMPSriIh7rpH-yu0lbjxvTSkEH6RW_Y7jLYbJC5EfI4FgiMteyl_uslnlkS561ZiR8H-xpZybM2CYyuSJYHPFnDmT14lpr_iyGXuDkikfyEtiKbXg9DioVKdZF9xloCRBKDBllPFdtUs4digaMgLMK4ABxEGiSFgBF9mdnsk-qRCla-5ieu2tfU1L4_YDOpLBnSaP0FZcxWuo8hrb3mQ1ukwRRGvxLdXOIXDx2KYQaVDyWGhJJOy48ZCxfkQXbfWMm6WrX-RidDwpolfodg  

# https://medium.com/@mczachurski/kubernetes-on-raspberry-pi-with-net-core-36ea79681fe7  


