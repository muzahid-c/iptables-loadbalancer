# IPTables as a Load Balancer
 In this tutotial we will learn how to use IPTables as a TCP load balancer

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

## IPTables Chain
 There are three chain (policy):

### 1. Input
 This chain is used to control the behavior for incoming connections.

### 2. Forward
 This chain is used for incoming connections that aren't actually being delivered locally. Like a router - data is always being sent to it but rarely actually destined for the router itself; the data is just forwarded to its target.

### 3. Output
This chain is used for outgoing connections.  

## IPTables Policy chain default behaviour
`iptables -L | grep policy`

## IPTables Response
There are three responses

### 1. Accept
Allow the connection.

### 2. Drop
Drop the connection, act like it never happened.

### 3. Reject
Don't allow the connection, but send back an error. It is like *drop* but with response.

## Save IPTables rules
For ubunutu
`sudo /sbin/iptables-save`

## TCP Load Balancer using IPTables


### Step1 (Create Custom docker network)
docker network create --subnet=11.11.11.0/24 lbnet0

### Step2 (Create nodes)

*Node1*
`docker build -t node1:v1 .`

`docker run --net lbnet0 --ip 11.11.11.7 --name node1 node1:v1`

*Node2*
`docker build -t node2:v1 .`

`docker run --net lbnet0 --ip 11.11.11.8 --name node2 node2:v1`

*Node3*

`docker build -t node3:v1 .`

`docker run --net lbnet0 --ip 11.11.11.9 --name node3 node3:v1`

### Step3 ()

### Reference
1. https://www.howtogeek.com/177621/the-beginners-guide-to-iptables-the-linux-firewall/
2. https://scalingo.com/blog/iptables
3. https://en.wikipedia.org/wiki/Iptables
4. man iptables
5. https://www.hostinger.com/tutorials/iptables-tutorial
6. man iptables-extensions