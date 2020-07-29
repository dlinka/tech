容器和宿主机之间网络连通测试

    ip add
    ↓
    //Docker的docker0网卡
    docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP
    
    docker run -d tomcat
    
    ip addr
    ↓
    //Docker给容器分配了一块网卡
    202: veth89c4b2d@if201: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP
     inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
    
    docker exec -it feeeccce7613 ip addr
    ↓
    //容器内跟宿主机相对应的网卡
    201: eth0@if202: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
     inet 172.17.0.4/16 brd 172.17.255.255 scope global eth0
    
    //宿主机ping容器
    ping 172.17.0.4
    PING 172.17.0.4 (172.17.0.4) 56(84) bytes of data.
    64 bytes from 172.17.0.4: icmp_seq=1 ttl=64 time=0.115 ms
    
    //容器ping宿主机
    docker exec -it feeeccce7613 ping 172.17.0.1
    PING 172.17.0.1 (172.17.0.1) 56(84) bytes of data.
    64 bytes from 172.17.0.1: icmp_seq=1 ttl=64 time=0.079 ms
    
容器和容器之间网络连通测试

    docker run -d tomcat
    
    docker exec -it 6d6a82d8e029 ip addr
    ↓
    203: eth0@if204: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
     inet 172.17.0.5/16 brd 172.17.255.255 scope global eth0
    
    //容器之间ping
    docker exec -it feeeccce7613 ping 172.17.0.5
    PING 172.17.0.5 (172.17.0.5) 56(84) bytes of data.
    64 bytes from 172.17.0.5: icmp_seq=1 ttl=64 time=0.178 ms

Docker自定义网络

    docker network create --driver bridge  --subnet 192.168.0.0/16 --gateway 192.168.0.1 diy-net

    docker run -d -P --name tomcat01 --net diy-net tomcat
    docker run -d -P --name tomcat02 --net diy-net tomcat
    
    //使用容器名进行ping
    docker exec -it tomcat01 ping tomcat02
    PING tomcat02 (192.168.0.3) 56(84) bytes of data.
    64 bytes from tomcat02.diy-net (192.168.0.3): icmp_seq=1 ttl=64 time=0.091 ms
    
---
