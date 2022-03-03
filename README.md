# IPTables as a Load Balancer
 In this tutotial we will learn how to use IPTables as a TCP load balancer.

## What is IPTables
 Iptables is an extremely flexible firewall utility built for Linux operating systems. It is a command line utility that uses policy chains to allow or block traffic. iptables requires elevated privileges to operate and must be executed by user `root`, otherwise it fails to function. On most Linux systems, iptables is installed as `usr/sbin/iptables` and documented in its man pages, which can be opened using `man iptables` when installed. It may also be found in `/sbin/iptables`.

## What is Load Balancer
Load balancing refers to the process of distributing a set of tasks over a set of resources (computing units), with the aim of making their overall processing more efficient.

## IPTables Tables:
There are currently five independent tables:

### 1. filter
This is the default table (if no -t option is passed).

### 2. nat
This  table is consulted when a packet that creates a new connection is encountered.  It consists of four built-ins: PREROUTING (for altering packets as soon as they come in), INPUT (for altering packets destined for local sockets), OUTPUT (for altering locally-generated packets before routing), and POSTROUTING (for altering packets as they are about to go out).

### 3. mangle
### 4. raw
### 5. security

## IPTables Chain:
 There are five chain (policy):

### 1. INPUT
 This chain is used to control the behavior for incoming connections.

### 2. FORWARD
 This chain is used for incoming connections that aren't actually being delivered locally. Like a router - data is always being sent to it but rarely actually destined for the router itself; the data is just forwarded to its target.

### 3. OUTPUT
This chain is used for outgoing connections.  

### 4. PREROUTING
This chain is used before making routing decision by routing table.

### 5. POSTROUTING
This chain is used after making routing decision by routing table.

## IPTables Policy chain default behaviour
Below command shows the default behaviour of iptables. Since no table is specified here, we can assume it is `filter` table.
```
iptables -L | grep policy
```

## IPTables Response:
There are three responses:

### 1. ACCEPT
Allow the connection.

### 2. DROP
Drop the connection, act like it never happened.

### 3. REJECT
Don't allow the connection, but send back an error. It is like *drop* but with response.

## IPTables Command Syntax
IPTables follow below syntax:
```
iptables -t [table] - OPTIONS [CHAINS] [matching component] [action component]
```
Example: <br />
`iptables -t filter -A FORWARD -d 11.11.11.1 -j REJECT`
<br /> Meaning in filter table append (-A) in the forward chain by adding a rule. The rule is if destination is 11.11.11.1 then REJECT the packet. Here -j means Jump which is an action. Matching component is ignored here but can be used using -m.


## Save IPTables rule
IPTables rules are volatile is not saved by below command (for ubuntu linux).
```
sudo /sbin/iptables-save
```

## TCP Load Balancer using IPTables
In this scenerio we will use three docker container as nodes.


### Create Custom docker network
We will create a custom network in docker first.
```
docker network create --subnet=11.11.11.0/24 lbnet0`
```
### Create nodes

*Node1*
```
docker build -t node1:v1 .
docker run -di --net lbnet0 --ip 11.11.11.7 --name node1 node1:v1
```

*Node2*
```
docker build -t node2:v1 .
docker run -di --net lbnet0 --ip 11.11.11.8 --name node2 node2:v1
```

*Node3*

```
docker build -t node3:v1 .
docker run -di --net lbnet0 --ip 11.11.11.9 --name node3 node3:v1
```

### Load Balancing IPTables rule
We can use two methods: <br /> 
*Round Robin* and *Random*. <br />
To use above two method `--match statistic` is used (Details in iptables-extensions manual).

*1. Using Round Robin:* <br />
Below three rules need to be added in NAT table:

```
iptables -A PREROUTING -t nat -p tcp -d 192.168.88.205 --dport 12345 -m statistic --mode nth --every 3 --packet 0 -j DNAT --to-destination 11.11.11.7:80
iptables -A PREROUTING -t nat -p tcp -d 192.168.88.205 --dport 12345 -m statistic --mode nth --every 2 --packet 0 -j DNAT --to-destination 11.11.11.8:80
iptables -A PREROUTING -t nat -p tcp -d 192.168.88.205 --dport 12345 -m statistic --mode nth --every 1 --packet 0 -j DNAT --to-destination 11.11.11.9:80
```
In the last rule `--mode nth --every 1 --packet 0` is not necessary as last packer will always enter in the third node.

Since DROP rules is applied in filter table by default while installing docker explicit ACCEPT rules are required for the nodes.
```
iptables -t filter -A FORWARD -d 11.11.11.7 -j ACCEPT
iptables -t filter -A FORWARD -d 11.11.11.8 -j ACCEPT
iptables -t filter -A FORWARD -d 11.11.11.9 -j ACCEPT
```

Now if we use `curl 192.168.88.205:12345` we can see tcp load balancer is working. 

*Note:* For some reason if browser is used it is always connecting node1. 

*2. Using Random:* <br />
For Random method below rules are required:
```
iptables -A PREROUTING -t nat -p tcp -d 192.168.88.205 --dport 12345 -m statistic --mode random --probability 0.33 -j DNAT --to-destination 11.11.11.7:80
iptables -A PREROUTING -t nat -p tcp -d 192.168.88.205 --dport 12345 -m statistic --mode random --probability 0.5 -j DNAT --to-destination 11.11.11.8:80
iptables -A PREROUTING -t nat -p tcp -d 192.168.88.205 --dport 12345 -m statistic --mode random --probability 1.0 -j DNAT --to-destination 11.11.11.9:80
```
Again `-m statistic --mode random --probability 1.0` is not required for last rule. 
Here probability is calculted using below formula:
```
P = 1/(n-i+1)
```
Where P = Probality, n = Number of nodes (in this example n = 3) and i is index meaning if it is first node then i = 1, for second i = 2, for third i = 3 and so on.


### Reference
1. https://www.howtogeek.com/177621/the-beginners-guide-to-iptables-the-linux-firewall/
2. https://scalingo.com/blog/iptables
3. https://en.wikipedia.org/wiki/Iptables
4. man iptables
5. https://www.hostinger.com/tutorials/iptables-tutorial
6. man iptables-extensions
