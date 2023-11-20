# Docker Network
- Docker network sẽ đảm nhiệm nhiệm vụ kết nối mạng giữa các container với nhau, kết nối giữa container với bên ngoài, cũng như kết nối giữa các cụm docker containers.
- Published ports: port container được xuất ra ngoài Internet
    - Cấu hình bởi tham số `-p` hay `--publish` trong lệnh khởi chạy container 
        | Giá trị tham số               | Giải thích                                                                                                                             |
        | ----------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
        | -p 8080:80                    | Map port 8080 trên máy host sang TCP port 80 trong container.                                                                          |
        | -p 192.168.1.100:8080:80      | Map port 8080 trên máy host có IP 192.168.1.100 tới TCP port 80 trong container.                                                       |
        | -p 8080:80/udp                | Map port 8080 trên máy host sang UDP port 80 trong container.                                                                          |
        | -p 8080:80/tcp -p 8080:80/udp | Map TCP port 8080 trên máy host tới  TCP port 80 trong container, and map UDP port 8080 trên máy host tới UDP port 80 trong container. |

- Các loại **network drivers** của Docker: 5 loại bridge, host, overlay, IPvLAN, macvlan
    + **BRIDGE** : driver mạng default. Bridge network thường được sử dụng khi cần chạy ứng dụng dưới dạng các container độc lập cần giao tiếp với nhau. Các container trong cùng mạng có thể giao tiếp với nhau qua địa chỉ IP. (Kết nối bắc cầu. các Container sẽ kết nối tới bridge và bridge sẽ kết nối với bên ngoài.) 
    + **HOST** : Dùng khi container cần giao tiếp với host và sử dụng luôn mạng ở máy host. (Kết nối trực tiếp với mạng máy host ko qua Docker)
    + **OVERLAY** : Mạng lớp phủ - Overlay network tạo một mạng phân tán giữa nhiều máy chủ Docker. Kết nối nhiều Docker daemons với nhau và cho phép các cụm services giao tiếp với nhau. Chúng ta có thể sử dụng overlay network để giao tiếp dễ dàng giữa cụm các services với một container độc lập, hay giữa 2 container với nhau ở khác máy chủ Docker daemons.(Mạng kết nối giữa các Docker trên các máy host khác nhau)
    + **IPvlan**: Cung cấp cho người dùng toàn quyền kiểm soát việc đánh địa chỉ cho cả IPv4 và IPv6. VLAN driver được xây dựng trên nền tảng đó để cung cấp cho các operator quyền kiểm soát hoàn toàn việc gắn thẻ VLAN layer 2 và thậm chí cả định tuyến IPvlan L3 cho những người dùng quan tâm đến tích hợp underlay network.(Đọc khó hiểu quá '.')
    + **MACVLAN**: Mạng Macvlan cho phép chúng ta gán địa chỉ MAC cho container, điều này làm cho mỗi container như là một thiết bị vật lý trong mạng. Docker daemon định tuyến truy cập tới container bởi địa chỉ MAC. Sử dụng driver macvlan là lựa chon tốt khi các ứng dụng khác cần phải connect đến theo địa chỉ vật lý hơn là thông qua các lớp mạng của máy chủ.
    - **NONE** Với container không cần networking hoặc cần disable đi tất cả mọi networking, chúng ta sẽ chọn driver này. Thường được dùng với mạng tùy chỉnh.

* Tóm tắt về Network driver
    - Bridge networks là tốt nhất khi cần nhiều container có thể giao tiếp trên CÙNG một Docker host.
    - Host networks là tốt nhất khi network của container không cần phải tách biệt khỏi Docker host, nhưng vẫn muốn các khía cạnh khác của container được cách ly.
    - Overlay networks là tốt nhất khi cần các container chạy trên các Docker host khác nhau có thể giao tiếp với nhau hoặc khi nhiều ứng dụng hoạt động cùng nhau bằng cách sử dụng dịch vụ Swarm.
    - Macvlan network là tốt nhất khi đang chuyển dịch rời khỏi việc dùng máy ảo hoặc cần container trông giống như host vật lý trên mạng, mỗi container có một địa chỉ MAC duy nhất.
    - IPvlan tương tự như Macvlan nhưng không gán địa chỉ MAC duy nhất cho container. Hãy cân nhắc sử dụng IPvlan khi có hạn chế về số lượng địa chỉ MAC có thể được gán cho giao diện mạng hoặc cổng.

##  Triển khai từng loại network driver 
#### Các lệnh hay dùng 
- Xem danh sách các networks
```
docker network ls
```
- Xóa network 
```
docker network rm <tên_network>
```
- Kết nối container đến networks
```
docker network connect <tên_network> <tên_container>
```
- Ngắt kết nối container đến networks
```
docker network disconnect <tên_network> <tên_container>
```
#### Bridge
- Driver mạng default. Bridge network thường được sử dụng khi cần chạy ứng dụng dưới dạng các container độc lập cần giao tiếp với nhau.
- Tạo network 
```
docker network create <tên_network>
```
- Xem thông tin network của docker
```
docker network inspect bridge
```
- Test
    + Tạo 1 network kiểu bridge tên test-net: `--driver bridge` tham số này có hay không cũng được do mặc định tạo network Docker sẽ nhận driver là bridge. Lệnh `docker network create --driver bridge test-net`
    ```
    root@instance:~# docker network create --driver bridge test-net
    e5fecdf0ee50e63c521a99425bf68845940d36b2da9848302cebe27ab0177715
    ```
    + List các network: Lệnh` docker network ls`
    ```
    root@instance:~# docker network ls
    NETWORK ID     NAME       DRIVER    SCOPE
    2c19aad76a16   bridge     bridge    local
    df3c9e4eeb18   host       host      local
    7bd1eb38165d   none       null      local
    e5fecdf0ee50   test-net   bridge    local
    ```
    + Tạo 4 container trong đó 1,2,4 kết nối vào test-net; 3,4 kết nối vào bridge
    ```
    docker run -itd --name alpine1 --network test-net alpine ash
    docker run -itd --name alpine2 --network test-net alpine ash
    docker run -itd --name alpine3 alpine ash
    docker run -itd --name alpine4 --network test-net alpine ash
    docker network connect bridge alpine4
    ```
    
    ```
    root@instance:~# docker run -itd --name alpine1 --network test-net alpine ash
    Unable to find image 'alpine:latest' locally
    latest: Pulling from library/alpine
    96526aa774ef: Pull complete
    Digest: sha256:eece025e432126ce23f223450a0326fbebde39cdf496a85d8c016293fc851978
    Status: Downloaded newer image for alpine:latest
    044a5f7b0fb7367e52b02ddce84c81251cb4a08f9a8d8eb4b642cc6f7feeedc7
    root@instance:~# docker run -itd --name alpine2 --network test-net alpine ash
    790f1eacf42d8d83e57ad6f17183e16ce5974551b2b3f883c8bc9d2db6bdc446
    root@instance:~# docker run -itd --name alpine3 alpine ash
    dbcf1e4c84278c4967743aab8c0e604c3cc4c59167066f5ea4cfc79099f77625
    root@instance:~# docker run -itd --name alpine4 --network test-net alpine ash
    3b7bb26989bc6739e3d6461b1d4ab382cd0d4694e32ba6870df5edb779d966e5
    root@instance:~# docker network connect bridge alpine4
    ```
    + Inspect the bridge và test-net network để kiểm tra 
    docker network inspect bridge
    docker network inspect test-net
        ```
        root@instance:~# docker network inspect bridge
        [
            {
                "Name": "bridge",
                "Id": "2c19aad76a16b8c0753e86b2005cad781cdaf0c3aa61fb8019518                                                                                                                                     ... 
                "Containers": {
                    "3b7bb26989bc6739e3d6461b1d4ab382cd0d4694e32ba6870df5edb                                                                                                                                                       779d966e5": {
                        "Name": "alpine4",
                        "EndpointID": "221fce5289549bdf2d3c08f1bee22ec66bc5e                                                                                                                                                       324c804d64b26776a048fae7f3e",
                        "MacAddress": "02:42:ac:11:00:03",
                        "IPv4Address": "172.17.0.3/16",
                        "IPv6Address": ""
                    },
                    "dbcf1e4c84278c4967743aab8c0e604c3cc4c59167066f5ea4cfc79                                                                                                                                                       099f77625": {
                        "Name": "alpine3",
                        "EndpointID": "cc308cbf091b8d93b0dcd9c82c49708fd09ed                                                                                                                                                       34214a2a244651d6a0131233368",
                        "MacAddress": "02:42:ac:11:00:02",
                        "IPv4Address": "172.17.0.2/16",
                        "IPv6Address": ""
                    }
              ...
              }
        ]
        root@instance:~# docker network inspect test-net
        [
            {
                "Name": "test-net",
                "Id": "e5fecdf0ee50e63c521a99425bf68845940d36b2da9848302cebe27ab0177715",
               ...
                "Containers": {
                    "044a5f7b0fb7367e52b02ddce84c81251cb4a08f9a8d8eb4b642cc6f7feeedc7": {
                        "Name": "alpine1",
                        "EndpointID": "003f78f91fcef9847e32a29313748530f4a5551e0f02b594991c4210b703ab39",
                        "MacAddress": "02:42:c0:a8:e0:02",
                        "IPv4Address": "192.168.224.2/20",
                        "IPv6Address": ""
                    },
                    "3b7bb26989bc6739e3d6461b1d4ab382cd0d4694e32ba6870df5edb779d966e5": {
                        "Name": "alpine4",
                        "EndpointID": "3eed1646245e4d9d05131549a3ad2e7490330770db66e8172f8ed6f993e642fa",
                        "MacAddress": "02:42:c0:a8:e0:04",
                        "IPv4Address": "192.168.224.4/20",
                        "IPv6Address": ""
                    },
                    "790f1eacf42d8d83e57ad6f17183e16ce5974551b2b3f883c8bc9d2db6bdc446": {
                        "Name": "alpine2",
                        "EndpointID": "e4d8947ed225fa9fb4233468c45d37d012a132b9f2b230140e380ceee5bf19e5",
                        "MacAddress": "02:42:c0:a8:e0:03",
                        "IPv4Address": "192.168.224.3/20",
                        "IPv6Address": ""
                    }
                },
                "Options": {},
                "Labels": {}
            }
        ]
        ```
     + Attach vào container alpine tiến hành test ping sang alpine 2,4 thấy ping dc do chung 1 1 network, còn ping sang alpine3 ko dc do ko chung 1 network. 
    ```docker container attach alpine1```
     ```ping -c 4 alpine2```
     => Bridge kết nối các container với nhau . Bridge kết nối ra bên ngoài
#### Host
- Dùng khi container cần giao tiếp với host và sử dụng luôn mạng ở máy host
- Test: 
    + Run container từ image ngnix với network là Host. 
    ```
    docker run --rm -d --network host --name my_nginx nginx
    ```
    ```
    root@instance:~# docker run --rm -d --network host --name my_nginx nginx
    Unable to find image 'nginx:latest' locally
    latest: Pulling from library/nginx
    578acb154839: Pull complete
    e398db710407: Pull complete
    85c41ebe6d66: Pull complete
    7170a263b582: Pull complete
    8f28d06e2e2e: Pull complete
    6f837de2f887: Pull complete
    c1dfc7e1671e: Pull complete
    Digest: sha256:86e53c4c16a6a276b204b0fd3a8143d86547c967dc8258b3d47c3a21bb68d3c6
    Status: Downloaded newer image for nginx:latest
    b93b93f08133030a123a5f25a4f5681cbde6278c4ec50605a17febe461dfea78
    ```
    + Kiểm tra `ip addr show` sẽ ko thấy 1 network interface mới nào được tạo
    + Kiểm tra port 80 `lsof -i:80`
    ```
    root@instance:~# lsof -i:80
    COMMAND     PID            USER   FD   TYPE   DEVICE SIZE/OFF NODE NAME
    nginx   2551719            root    6u  IPv4 30241678      0t0  TCP *:http (LISTEN)
    nginx   2551719            root    7u  IPv6 30241679      0t0  TCP *:http (LISTEN)
    nginx   2551759 systemd-resolve    6u  IPv4 30241678      0t0  TCP *:http (LISTEN)
    nginx   2551759 systemd-resolve    7u  IPv6 30241679      0t0  TCP *:http (LISTEN)
    nginx   2551760 systemd-resolve    6u  IPv4 30241678      0t0  TCP *:http (LISTEN)
    nginx   2551760 systemd-resolve    7u  IPv6 30241679      0t0  TCP *:http (LISTEN)
    ```
    Command chạy trực tiếp là ngnix ko có dấu ấn của Docker 
    => Host: Container kết nối mạng trực tiếp ra máy host ko thông qua Docker. 
#### Overlay 
- Đây là một phần của hệ thống Docker Swarm 
- Docker Swarm là một công cụ quản lý và triển khai các ứng dụng dựa trên Docker trên một cụm máy chủ (cluster). Nó là một phần của hệ sinh thái Docker và cho phép bạn tạo và quản lý các môi trường ứng dụng phân tán trên nhiều máy chủ.
- Docker Swarm gồm: Manager (Máy quản lí), Node(Các máy con), Service(Các dịch vụ chạy trên Node định nghĩa bởi Docker Compose )
- Một host tham gia vào Swarm sẽ có hai network
+ Overlay network gọi là `ingress` làm nhiệm vụ xử lý các control và data traffic liên quan đến swarm service
+ Bridge network gọi là `docker_gwbridge` dùng kết nối các máy host có Docker với nhau.
- Docker Swarm sử dụng các port 
    - TCP port 2377 cho việc giao tiếp vào quản lí các cluster
    - TCP and UDP port 7946 cho việc kết nối giữa các node trong swarm
    - UDP port 4789 cho lưu lượng mạng truyền trong overlay 
- Các lệnh 
    - Tạo 1 overlay network ` docker network create -d overlay my-overlay`
    - Tạo 1 overlay network có kèm mã hóa dữ liệu `docker network create --opt encrypted --driver overlay my-overlay-encrypted`
    - 1 overlay network có 2 phần: ingress và docker_gwbridge. Chúng ta có thể tự cấu hình các network này:
        + Ingress 
            + Xóa network ingress cũ `docker network rm ingress`
            + Tạo network ingress mới với cấu hình tự chọn: 
            ```
                docker network create \
                  --driver overlay \
                  --ingress \
                  --subnet=10.11.0.0/16 \
                  --gateway=10.11.0.2 \
                  --opt com.docker.network.driver.mtu=1200 \
                  my-ingress
             ```
        + Docker_gwbridge
            + Stop Docker `systemctl stop docker`
            + Xóa Docker_gwbridge cũ 
            ```
            sudo ip link set docker_gwbridge down
            sudo ip link del dev docker_gwbridge
            ```
            + Khởi động Docker nhưng không dc join vào Swarm nào: Nếu có join Swarm nào dùng lệnh `docker swarm leave --force` để thoát ra. 
            `systemctl start docker`
            + Tạo Docker_gwbridge mới với cấu hình tùy chọn: 
            ```
            docker network create \
            --subnet 10.11.0.0/16 \
            --opt com.docker.network.bridge.name=docker_gwbridge \
            --opt com.docker.network.bridge.enable_icc=false \
            --opt com.docker.network.bridge.enable_ip_masquerade=true \
            docker_gwbridge
            ```
            + Sau bước trên Docker_gwbridge mới dc tạo. Tiến hành join vào Docker Swarm lúc này Docker sẽ sử dụng Docker_gwbridge mình đã cấu hình. 
- Test : Dựng 1 swarm 1 máy manager 1 máy là worker sử dụng overlay mặc định
    + 2 Máy : instance1(manager) và instance2(worker)
    + Trên máy manager:
        + Khởi tạo Swarm mới 
        ```
        docker swarm init --advertise-addr=<IP-ADDRESS-OF-MANAGER>
        ```
        + Kết quả trả về lệnh để join Swarm trên máy worker.
        ```
        root@instance3:~# docker swarm init --advertise-addr=129.154.53.45
        Swarm initialized: current node (tkbia72d2nrj0eiy1vyk9umtc) is now a manager.

        To add a worker to this swarm, run the following command:

            docker swarm join --token SWMTKN-1-10fngep9om0l6tm7agpzc3cttbz7n31s73qxrweaw6b5bxp5r9-1gmcscljoui56u53ctx9s6qc9 129.154.53.45:2377

        To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.

        ```
    + Trên máy worker : 
        + Join Swarm với lệnh có được từ máy manager 
        ```
        docker swarm join --token TOKEN \
          --advertise-addr <IP-ADDRESS-OF-MANAGER>:2377
        ```
        + Join Swarm thành công 
        ```
        root@instance:~# docker swarm join --token SWMTKN-1-10fngep9om0l6tm7agpzc3cttbz7n31s73qxrweaw6b5bxp5r9-1gmcscljoui56u53ctx9s6qc9 129.154.53.45:2377
        This node joined a swarm as a worker.
        ```
    + Kiểm tra: 
        + Liệt kê các node trong Docker Swarm: Trên máy manager dùng lệnh `docker node ls` 
        ```
        root@instance3:~# docker node ls
        ID                            HOSTNAME    STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
        wsq2doyiytripoc4lqvgef5cb     instance    Ready     Active                          24.0.7
        tkbia72d2nrj0eiy1vyk9umtc *   instance3   Ready     Active         Leader           24.0.7
        ```
        + Liệt kê các network có trong Docker: Chạy lệnh `docker network ls` trên cả máy manager và máy worker 
            ```
            root@instance:~# docker network ls
            NETWORK ID     NAME              DRIVER    SCOPE
            2c19aad76a16   bridge            bridge    local
            c318a60f38c4   docker_gwbridge   bridge    local
            df3c9e4eeb18   host              host      local
            injhu1hqqd0n   ingress           overlay   swarm
            7bd1eb38165d   none              null      local
            ```
            ```
            root@instance3:~# docker network ls
            NETWORK ID     NAME              DRIVER    SCOPE
            6bb3b2ce341a   bridge            bridge    local
            c4a90e385ae8   docker_gwbridge   bridge    local
            e7d2350d2fb3   host              host      local
            injhu1hqqd0n   ingress           overlay   swarm
            71043fb74630   none              null      local
            ```
         => Ta thấy đã có 2 network docker_gwbridge và ingress
     + Trên máy manager tạo network overlay mới : Network này sẽ tự động được thêm trên máy worker 
     ```
     docker network create --driver=overlay --attachable test-net
     ```
     + Trên máy manager và worker tạo 2 container `alpine` 
         ```
         docker run -itd --name alpine1 --network test-net alpine
         ```
         ```
         docker run -itd --name alpine2 --network test-net alpine
         ```
    + Từ máy manager ping sang worker `docker exec -it alpine1 ping alpine2`
      ```
      root@instance:~# docker exec -it alpine1 ping -c 4 alpine2
        PING alpine2 (10.0.1.4): 56 data bytes
        64 bytes from 10.0.1.4: seq=0 ttl=64 time=288.576 ms
        64 bytes from 10.0.1.4: seq=1 ttl=64 time=288.577 ms
        64 bytes from 10.0.1.4: seq=2 ttl=64 time=288.526 ms
        64 bytes from 10.0.1.4: seq=3 ttl=64 time=289.498 ms

      ```
    + Từ máy worker ping sang manager `docker exec -it alpine2 ping alpine1`
      ```
        root@mail:~# docker exec -it alpine2 ping -c 4 alpine1
        PING alpine1 (10.0.1.2): 56 data bytes
        64 bytes from 10.0.1.2: seq=0 ttl=64 time=288.866 ms
        64 bytes from 10.0.1.2: seq=1 ttl=64 time=288.507 ms
        64 bytes from 10.0.1.2: seq=2 ttl=64 time=288.546 ms
        64 bytes from 10.0.1.2: seq=3 ttl=64 time=289.298 ms

      ```
         => C/m tác dụng của network overlay kết nối 2 docker host khác nhau
#### Macvlan (MAC Virtual LAN )
- Một vài ứng dụng đặc biệt là các ứng dụng giám sát mạng cần kết nối tới interface mạng vật lý. Với những ứng dụng như vậy cần sử dụng `macvlan` để đưa container kết nối mạng ra ngoài với mac riêng đứng như 1 thiết bị phần cứng trên mạng. 
- Có 2 hình thức triển khai Macvlan 
    + Bridge mode
    + 802.1Q trunk bridge mode
- Lệnh
    + Tạo Macvlan kiểu Bridge  
    ```
    docker network create -d macvlan \
      --subnet=172.16.86.0/24 \
      --gateway=172.16.86.1 \
      -o parent=eth0 pub_net
    ```
    + Tạo Macvlan kiểu 802.1Q trunk bridge 
    ```
    docker network create -d macvlan \
        --subnet=192.168.50.0/24 \
        --gateway=192.168.50.1 \
        -o parent=eth0.50 macvlan50
    ```
    
    + Phần subnet và gateway khác nhau tùy theo cấu hình mạng LAN. 
    + Phần parent khác nhau tùy theo tên interface mạng kết nối với mạng LAN
- Test : Môi trường là 1 máy tính chạy ubuntu làm host cho docker kết nối vào router. ![image](https://i.imgur.com/ZlMFQZ8.png)
    + Tạo 1 Macvlan kiểu Bridge và kết nối tới 1 container :
        ```
        sudo docker network create \
                --driver macvlan \
                --subnet 192.168.1.0/24 \
                --gateway 192.168.1.1 \
                --ip-range 192.168.1.80/28 \
                --aux-address 'host=192.168.1.80' \
                --opt parent=ens33 \
                macvlan_net    
        ```
        + Phần eth0 có thể là khác nhau tùy theo tên interface mạng chính của máy host. 
        + Tạo 1 container kết nối qua network mới tạo
        ```
        sudo docker run --name webapp --rm --detach --network macvlan_net nginx
        ```
        + Trong mạng truy cập aux address trên thấy có kết nối => Container hoạt động bt qua mạng MACVlan
        + Inspect network
        ```
        docker inspect macvlan_net
        ```
        + Thu kết quả 
        ```
        root@ah:~# docker inspect macvlan_net
        [
            {
                "Name": "macvlan_net",
                "Id": "3dd758d3897c3d9ffbf13fa8881ec97960660ccb86e4e582935d98232b3905dd",
                "Created": "2023-11-16T16:38:22.927303063Z",
                "Scope": "local",
                "Driver": "macvlan",
                "EnableIPv6": false,
                "IPAM": {
                    "Driver": "default",
                    "Options": {},
                    "Config": [
                        {
                            "Subnet": "192.168.1.0/24",
                            "IPRange": "192.168.1.80/28",
                            "Gateway": "192.168.1.1",
                            "AuxiliaryAddresses": {
                                "host": "192.168.1.80"
                            }
                        }
                    ]
                },
                "Internal": false,
                "Attachable": false,
                "Ingress": false,
                "ConfigFrom": {
                    "Network": ""
                },
                "ConfigOnly": false,
                "Containers": {
                    "88162a0767fcbe7a2ed67cd61c55cbcb864d05b376becb707b50da3d8c8f9538": {
                        "Name": "webapp",
                        "EndpointID": "83b7cc570c541657812eace0c482ee5c447c43e1d1e8056cd5efe51761f04ef6",
                        "MacAddress": "02:42:c0:a8:01:51",
                        "IPv4Address": "192.168.1.81/24",
                        "IPv6Address": ""
                    }
                },
                "Options": {
                    "parent": "ens33"
                },
                "Labels": {}
            }
        ]
        ```
        + Chú ý trong phần `Containers` mục MACaddress và Ipv4 đây là địa chỉ MAC và Ip riêng của container trong mạng LAN 
        + => Sử dụng network macvlan container có thể kết nối vào mạng LAN và có địa chỉ MAC riêng, IP riêng và đóng vai trò như một máy vật lý trong mạng. 
    + Tạo Macvlan kiểu 802.1Q trunk bridge và kết nối tới 1 container
    ```
    docker network create -d macvlan \
       --subnet=192.168.1.0/24 \
       --gateway=192.168.1.1 \
       --opt parent=ens33.10 \
       my-8021q-macvlan-net
    ```
    
     + Tạo 1 container kết nối qua network mới tạo
        ```
        sudo docker run --name webapp --rm --detach --network my-8021q-macvlan-net nginx
        ```
        + Inspect network
        ```
        docker inspect my-8021q-macvlan-net
        ```
        + Thu kết quả 
            ```
            root@ah:~# docker inspect my-8021q-macvlan-net
            [
                {
                    "Name": "my-8021q-macvlan-net",
                    "Id": "d88d08734e9dbbd745e5278749d5b8f785d22851157045b77c3c0954673f8220",
                    "Created": "2023-11-16T17:19:00.184302014Z",
                    "Scope": "local",
                    "Driver": "macvlan",
                    "EnableIPv6": false,
                    "IPAM": {
                        "Driver": "default",
                        "Options": {},
                        "Config": [
                            {
                                "Subnet": "192.168.1.0/24",
                                "Gateway": "192.168.1.1"
                            }
                        ]
                    },
                    "Internal": false,
                    "Attachable": false,
                    "Ingress": false,
                    "ConfigFrom": {
                        "Network": ""
                    },
                    "ConfigOnly": false,
                    "Containers": {
                        "d6d2b3d05dd032131ec5ab039660c56a012cac589597c602bd5cf4e60c4856b4": {
                            "Name": "webapp",
                            "EndpointID": "be8e1f07c76a8bf254b13937c1677f520c8e5459a5d7772e0ec3e9824f0afb3c",
                            "MacAddress": "02:42:c0:a8:01:02",
                            "IPv4Address": "192.168.1.2/24",
                            "IPv6Address": ""
                        }
                    },
                    "Options": {
                        "parent": "ens33.10"
                    },
                    "Labels": {}
                }
            ]
            ```
        + Chú ý trong phần `Containers` mục MacAddress và IPv4Address đây là địa chỉ MAC và Ip riêng của container trong mạng LAN 
        + Trong LAN truy cập vào IP trên load dc ngnix => Container hoạt động bình thường khi sử dụng Macvlan kiểu 802.1Q trunk.  

