# RDMA exmaple

A simple RDMA server client example. The code contains a lot of comments. Here is the workflow that happens in the example: 

Client: 
  1. setup RDMA resources   
  2. connect to the server 
  3. receive server side buffer information via send/recv exchange 
  4. do an RDMA write to the server buffer from a (first) local buffer. The content of the buffer is the string passed with the `-s` argument. 
  5. do an RDMA read to read the content of the server buffer into a second local buffer. 
  6. compare the content of the first and second buffers, and match them. 
  7. disconnect 

Server: 
  1. setup RDMA resources 
  2. wait for a client to connect 
  3. allocate and pin a server buffer
  4. accept the incoming client connection 
  5. send information about the local server buffer to the client 
  6. wait for disconnect

###### How to run      
```text
● git clone https://github.com/DaiZhiyuan/rdma_example.git
● cd ./rdma-example
● cmake .
● make
``` 
 
###### server
```text
● ./bin/rdma_server
Server is listening successfully at: 0.0.0.0 , port: 20886
```
###### client
```text
● ./bin/rdma_client -a 10.0.2.15 -s jerrydai
Passed string is : jerrydai , with count 8
Trying to connect to server at : 10.0.2.15 port: 20886
The client is connected successfully
---------------------------------------------------------
buffer attr, addr: 0x5884e07a40d0 , len: 8 , stag : 0x2e87
---------------------------------------------------------
...
SUCCESS, source and destination buffers match
Client resource clean up is complete
```

# Does not have an RDMA device?

In case you do not have an RDMA device to test the code, you can setup Soft-RoCE emulates RDMA device on your Linux machine.

###### Kernel module

```text
● modprobe rdma_rxe
```

###### Installing userspace libraries and tools

```text
●  apt-get install libibverbs-dev librdmacm-dev rdmacm-utils ibverbs-utils
```

###### Add a new device

```text
● ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:1a:3a:f3 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic noprefixroute enp0s3
       valid_lft 81186sec preferred_lft 81186sec
    inet6 fe80::d0ae:a230:6d8e:f9a5/64 scope link noprefixroute
       valid_lft forever preferred_lft forever

● rdma link add rxe0 type rxe netdev enp0s3

● ibv_devices
    device                 node GUID
    ------              ----------------
    rxe0                0a0027fffe1a3af3

● ibv_devinfo
hca_id: rxe0
        transport:                      InfiniBand (0)
        fw_ver:                         0.0.0
        node_guid:                      0a00:27ff:fe1a:3af3
        sys_image_guid:                 0a00:27ff:fe1a:3af3
        vendor_id:                      0xffffff
        vendor_part_id:                 0
        hw_ver:                         0x0
        phys_port_cnt:                  1
                port:   1
                        state:                  PORT_ACTIVE (4)
                        max_mtu:                4096 (5)
                        active_mtu:             1024 (3)
                        sm_lid:                 0
                        port_lid:               0
                        port_lmc:               0x00
                        link_layer:             Ethernet

```

###### Remove a device

```text
● rdma -d link
link rxe0/1 state ACTIVE physical_state LINK_UP netdev enp0s3 netdev_index 2

● rdma link del rxe0

● ibv_devices
    device                 node GUID
    ------              ----------------
```

###### Testing the setup with rping

rping is like ping utility to test the working of an RDMA system. It has two components, a server and a client. In the example below, I have siw device attached to the loopback interface.


###### server

```text
● rping -s -a 10.0.2.15 
```

###### client

```text
● rping -c -a 10.0.2.15 -v -C 10
```

###### You should see a running output like

```text
ping data: rdma-ping-0: ABCDEFGHIJKLMNOPQRSTUVWXYZ[\]^_`abcdefghijklmnopqr
ping data: rdma-ping-1: BCDEFGHIJKLMNOPQRSTUVWXYZ[\]^_`abcdefghijklmnopqrs
ping data: rdma-ping-2: CDEFGHIJKLMNOPQRSTUVWXYZ[\]^_`abcdefghijklmnopqrst
ping data: rdma-ping-3: DEFGHIJKLMNOPQRSTUVWXYZ[\]^_`abcdefghijklmnopqrstu
ping data: rdma-ping-4: EFGHIJKLMNOPQRSTUVWXYZ[\]^_`abcdefghijklmnopqrstuv
ping data: rdma-ping-5: FGHIJKLMNOPQRSTUVWXYZ[\]^_`abcdefghijklmnopqrstuvw
ping data: rdma-ping-6: GHIJKLMNOPQRSTUVWXYZ[\]^_`abcdefghijklmnopqrstuvwx
ping data: rdma-ping-7: HIJKLMNOPQRSTUVWXYZ[\]^_`abcdefghijklmnopqrstuvwxy
ping data: rdma-ping-8: IJKLMNOPQRSTUVWXYZ[\]^_`abcdefghijklmnopqrstuvwxyz
ping data: rdma-ping-9: JKLMNOPQRSTUVWXYZ[\]^_`abcdefghijklmnopqrstuvwxyzA
```
