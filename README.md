# Namespace 

In computing, a namespace is a set of signs (names) that are used to identify and refer to objects of various kinds. 
A namespace ensures that all of a given set of objects have unique names so that they can be easily identified.

# ps aux 
What is the use of ps -aux command in linux?


You can use ps command to list the currently running processes with their PIDs.

a :- This option prints the running processes from all users.

u :- This option shows user or owner column in output.

x :- This option prints the processes those have not been executed from the terminal.

Collectively the options " aux " print all the running process in system regardless from where they have been executed.


```ps aux ```

```
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.0 226116  9940 ?        Ss    2022   2:53 /lib/systemd/systemd --system --deserialize 18
root         2  0.0  0.0      0     0 ?        S     2022   0:01 [kthreadd]
root         3  0.0  0.0      0     0 ?        I<    2022   0:00 [rcu_gp]
root         4  0.0  0.0      0     0 ?        I<    2022   0:00 [rcu_par_gp]
root         6  0.0  0.0      0     0 ?        I<    2022   0:00 [kworker/0:0H-kb]
root         9  0.0  0.0      0     0 ?        I<    2022   0:00 [mm_percpu_wq]
root        10  0.0  0.0      0     0 ?        S     2022   0:03 [ksoftirqd/0]
root        11  0.0  0.0      0     0 ?        I     2022  96:03 [rcu_sched]
root        12  0.0  0.0      0     0 ?        S     2022   0:26 [migration/0]
root        13  0.0  0.0      0     0 ?        S     2022   0:00 [idle_inject/0]
root        14  0.0  0.0      0     0 ?        S     2022   0:00 [cpuhp/0]
root        15  0.0  0.0      0     0 ?        S     2022   0:00 [cpuhp/1]
```

1. Create two namespaces : 
```
sudo ip netns add red
sudo ip netns add blue  
```

2. List the namespaces by : ```ip netns```
```
red  
blue 
```

To see the network interfaces on host :  ``` ip link ```

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eno1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 18:c0:4d:0d:a0:32 brd ff:ff:ff:ff:ff:ff
3: eno2: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 18:c0:4d:0d:a0:33 brd ff:ff:ff:ff:ff:ff
4: enp193s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether b8:ce:f6:77:d7:6e brd ff:ff:ff:ff:ff:ff
```

3. To list the interfaces within the namespace : ``` sudo ip netns exec red ip link ```

```
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
```

Another to that is : ``` sudo ip -n red link ```

# Creating virtual ethernet pair. 

4. To create virtual ethernet pairs execute : 
``` sudo ip link add veth-red type veth peer name veth-blue ```

5. Next step is to attach each interface to appropriate namespaces. 

```
sudo ip link set veth-red netns red 
sudo ip link set veth-blue netns blue 
```

6. Now we need to assign ip addresses. 

```
sudo ip -n blue addr add 192.168.15.2/24 dev veth-blue
sudo ip -n red  addr add 192.168.15.1/24 dev veth-red
```

7. Bring the interface UP.

```
sudo ip -n red link set veth-red up
sudo ip -n blue link set veth-blue up
```

8. We can ping from red to blue :  ``` sudo ip netns exec red ping 192.168.15.2 ```

```
PING 192.168.15.2 (192.168.15.2) 56(84) bytes of data.
64 bytes from 192.168.15.2: icmp_seq=1 ttl=64 time=0.020 ms
64 bytes from 192.168.15.2: icmp_seq=2 ttl=64 time=0.018 ms
64 bytes from 192.168.15.2: icmp_seq=3 ttl=64 time=0.019 ms
64 bytes from 192.168.15.2: icmp_seq=4 ttl=64 time=0.018 ms
^C
--- 192.168.15.2 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3056ms
rtt min/avg/max/mdev = 0.018/0.018/0.020/0.005 ms
```

# Check ARP table in namespaces. 

9. To check the arp table inside namespaces simply execute : 
```
sudo ip netns exec red arp
sudo ip netns exec blue arp
```

The results would be : 
```
Address                  HWtype  HWaddress           Flags Mask            Iface
192.168.15.2             ether   ee:de:39:8c:54:ab   C                     veth-red

Address                  HWtype  HWaddress           Flags Mask            Iface
192.168.15.1             ether   22:7d:5d:c4:4e:02   C                     veth-blue
```

# Linux Bridge (virtual switches)

To add new virtual switch : 
```
sudo ip link add v-net-0 type bridge
```

If you check ``` ip link ```
```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eno1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 18:c0:4d:0d:a0:32 brd ff:ff:ff:ff:ff:ff
3: eno2: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 18:c0:4d:0d:a0:33 brd ff:ff:ff:ff:ff:ff
4: enp193s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether b8:ce:f6:77:d7:6e brd ff:ff:ff:ff:ff:ff
5: v-net-0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 56:e0:47:4c:69:e0 brd ff:ff:ff:ff:ff:ff
```

Since the interface is down, we need to bring it up. 

``` sudo ip link set dev v-net-0 up ```
State after bringing up : 
```
5: v-net-0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether 56:e0:47:4c:69:e0 brd ff:ff:ff:ff:ff:ff
```

To delete a link : ``` sudo ip -n red link del veth-red ``` 
When you delete one end, the other end deletes automatically. 

Create namespaces : 
```
sudo ip netns add red
sudo ip netns add blue  
```

Now we need to create cables to connect the namespace to the bridge. 
Run ip link add command and create a pair with veth-red on one end and the other end veth-red-br. It connects to bridge network. 

```
sudo ip link add veth-red type veth peer name veth-red-br
sudo ip link add veth-blue type veth peer name veth-blue-br
```

Don't forget to bring them up : 
```
sudo ip link set veth-blue-br up
sudo ip link set veth-red-br up

```

Now it's time to connect the cables to namespaces : 

```
sudo ip link set veth-red netns red
sudo ip link set veth-blue netns blue
```

To attach the other end to the bridge network run : 
```
sudo ip link set veth-red-br master v-net-0
sudo ip link set veth-blue-br master v-net-0
```

Give the designated ip addresses : 

```
sudo ip -n red addr add 192.168.15.1/24 dev veth-red
sudo ip -n blue addr add 192.168.15.2/24 dev veth-blue
```

Turn the interfaces up : 
```
sudo ip -n red link set veth-red up
sudo ip -n blue link set veth-blue up
```

Now, we can ping from red to blue :
```
sudo ip netns exec red ping 192.168.15.2 
```

```
64 bytes from 192.168.15.2: icmp_seq=1 ttl=64 time=0.052 ms
64 bytes from 192.168.15.2: icmp_seq=2 ttl=64 time=0.026 ms
64 bytes from 192.168.15.2: icmp_seq=3 ttl=64 time=0.026 ms
```


