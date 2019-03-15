# Docker Network

## 1. Default network
Mặc định docker sẽ tạo ra 3 card mạng mặc định: bridge, host, none 

Ta có bảng link các chế độ card mạng của docker so với nền tảng thực 


    |        NỀN TẢNG THỰC          |             DOCKER          |
    |-------------------------------|-----------------------------|
    |           NAT Network         |           bridge            |
    |             Bridged           |           macvlan           |
    |        Private/Host-only      |           Host-only            |
    |      Overlay Network/VXLAN    |           overlay           |

Lệnh sau để xem có bao nhiêu card hiện đang tồn tại:
```sh
[root@localhost ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
bd93c41a1452        bridge              bridge              local
9a8f69b71a85        docker_gwbridge     bridge              local
63e6da0dffdb        extbr               macvlan             local
4c6f7ae82916        host                host                local
u5rnfumzpxqq        ingress             overlay             swarm
50606616eacf        none                null                local
```

### 1.1. None network 
Container sẽ không được cấu hình mạng khi ở chế độ này.
Khi sử dụng lệnh _ifconfig_ thì chỉ hiển thị duy nhất _lo interface_

### 1.2. Host network
Container sẽ có cấu hình mạng và thông số mạng giống hệt máy thật. Các cổng của container sẽ được map với cổng tương ứng trên máy thật.

### 1.3. Bridge
Chế độ mạng này dễ bị nhầm sang Bridged. Bridge của Docker giống với NAT Network.

### 1.4 macvlan
Chế độ này giống bridge của môi trường thực. Gói tin sẽ phải qua một switch ảo. Rồi mới ra ngoài Internet.
Ở chế độ này mỗi container sẽ có 1 IP riêng và MAC riêng.

Để tạo một macvlan ta sử dụng lệnh sau:

```sh
$ docker network create -d macvlan --subnet 172.16.69.0/24 --gateway 172.16.69.1 --ip-range=172.16.69.240/28 -o parent=eth0 macvlan0  
```
Trong đó: parent: là tên card mạng hiển thị ở _ifconfig_

## 2. User-defined networks
Để tự định nghĩa một network ta sử dụng lệnh sau:
```sh
$ docker network create --driver bridge --subnet 192.168.1.0/24 gateway=192.168.1.1 ten_bridge
```
Trong đó:
    * --driver bridge: là loại card mạng. Ở đây bridge có thể được thay thế bằng host, none, overlay, maclan,....

Để chạy một container với 1 loại card mạng xác định:
```sh
$ docker run --net=<Loại card mạng> -itd --name=<Tên container> <Tên Images>
```

## 3. Overlay network
Clustering Docker là một cụm các máy(Vật lí/ MVs) chạy một chức năng xác định.
Docker swarm là một Docker-native giải pháp tạo ra một cluster để quản lí những Docker host.

Overlay network là loại mạng kết nối nhiều clustering lại với nhau.

