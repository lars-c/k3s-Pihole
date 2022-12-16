# k3s-Pihole

K3s v1.24.8+k3s1 on Ubuntu 20.04 (AMD64). Disabled servicelb and traefik. Installed metalLB and traefik after the initial installation. Using local-path storage class.

It is not a cluster solution. I.e. you cannot scale this setup - replicas: absove 1 will not work. A ReadWriteOnce volume can only be mounted to a single pod (unless the pods are running on the same node). 

Pihole enviroment variables: TZ (timezone), PIHOLE_DNS (Upstream DNS server(s)) and FTLCONF_REPLY_ADDR4 (server's LAN IP - recommended by pihole). https://github.com/pi-hole/docker-pi-hole#readme

Pihole admin password is stored as a secret.

Parts I am not really happy about: Needed two different persistent volumes. Did not figure out how to just use one single bucket. Adding certificats to pihole. Only port 80 for now.

Storageclass:
```shell
kubectl get storageclasses.storage.k8s.io
NAME                   PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
local-path (default)   rancher.io/local-path   Delete          WaitForFirstConsumer   false                  43h
```

loadbalancer: metalLB is configured witH a IP pool of 192.168.1.150-192.168.1.159. Pool is called 'first-pool'.

```shell
kubectl get ipaddresspool -n metallb-system
NAME         AUTO ASSIGN   AVOID BUGGY IPS   ADDRESSES
first-pool   true          false             ["192.168.1.150-192.168.1.159"]
```

```shell
kubectl describe ipaddresspool first-pool -n metallb-system
...
Spec:
  Addresses:
    192.168.1.150-192.168.1.159
  Auto Assign:       true
  Avoid Buggy I Ps:  false
```

  
Default docker-compose.yaml file (edited) (https://github.com/pi-hole/docker-pi-hole#readme)

```shell
version: "3"
services:
  pihole:
    container_name: pihole
    image: pihole/pihole:latest
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "80:80/tcp"
    environment:
      TZ: 'America/Chicago'
      WEBPASSWORD: 'password'
    volumes:
      - './etc-pihole:/etc/pihole'
      - './etc-dnsmasq.d:/etc/dnsmasq.d'
    restart: unless-stopped
```

As I use the same docker image, I will need to handle the following values:

image: pihole/pihole:latest		Good for anything but Windows.
Ports: Pihole container/pod need to accept traffic on port 53 tcp/udp and port 80 (web interface)
Enviroment variables: Timezone and password.
Volumes: Need to persist two volumes from the docker image: /etc/pihole and /etc/dnsmasq.d
restart: K3s default restart policy is "always".
 
