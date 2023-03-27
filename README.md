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

Create two namespaces : 
```
sudo ip netns add red
sudo ip netns add blue  
```

List the namespaces by : ```ip netns```
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

To list the interfaces within the namespace : ``` sudo ip netns exec red ip link ```

```
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
```

Another to that is : ``` sudo ip -n red link ```

# Creating virtual ethernet pair. 

To create virtual ethernet pairs execute : 
``` sudo ip link add veth-red type veth peer name veth-blue ```

Next step is to attach each interface to appropriate namespaces. 

```
sudo ip link set veth-red netns red 
sudo ip link set veth-blue netns blue 
```

Now we need to assign ip addresses. 

```
sudo ip -n blue addr add 192.168.15.2/24 dev veth-blue
sudo ip -n red  addr add 192.168.15.1/24 dev veth-red
```

Now, bring the interface UP.

```
sudo ip -n 1topar link set veth-red up
sudo ip -n 2topar link set veth-blue up
```



