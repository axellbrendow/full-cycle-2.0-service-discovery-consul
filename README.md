# full-cycle-2.0-service-discovery-consul

Files I produced during the "Service Discovery and Consul" classes of my [Microservices Full Cycle 2.0 course](https://drive.google.com/file/d/1MdN-qK_8Pfg6YI3TSfSa5_2-FHmqGxEP/view?usp=sharing).

## HashiCorp Consul

![Consul multicloud example having APM, Logging, Gateways and Load Balancers](https://www.datocms-assets.com/2885/1622152328-control-plane.png?fit=max&fm=webp&q=80&w=1500)

## Running consul in Dev mode

```sh
docker-compose up -d

# After consul container startup, go inside it:
docker-compose exec consulserver01 sh

# Then run:
consul agent -dev
```

### Open another terminal and run:

```sh
docker-compose exec consulserver01 sh

consul members  # You should see the agent started above

# To see your nodes metadata:
curl localhost:8500/v1/catalog/nodes
```

### As Consul comes with a DNS Server, we will install the dig command in Alpine to test it:

```sh
apk add -U bind-tools

dig @localhost -p 8600 consulserver01.node.consul  # Consult the DNS server
```

### You should see something like that:

```
; <<>> DiG 9.16.20 <<>> @localhost -p 8600 consulserver01.node.consul
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 40676
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 2
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;consulserver01.node.consul.          IN      A

;; ANSWER SECTION:
consulserver01.node.consul.   0       IN      A       127.0.0.1

;; ADDITIONAL SECTION:
consulserver01.node.consul.   0       IN      TXT     "consul-network-segment="

;; Query time: 27 msec
;; SERVER: 127.0.0.1#8600(127.0.0.1)
;; WHEN: Fri Aug 20 12:07:43 UTC 2021
;; MSG SIZE  rcvd: 101
```

## Running consul in Server mode

This should be the final result after following this README:

http://localhost:8500

![Showing Consul UI with all services and nodes](./consul-ui.gif)

### Starting consulserver01

```sh
docker-compose up -d

# After consul container startup, go inside it:
docker-compose exec consulserver01 sh

# And run:
consul keygen
# Generates a cryptographic key that everyone should use to communicate in the cluster
# Output should be like: YGsICA9Fwq6TmFQI/qm4qIbdITrhvnHAsfZElu2czlk=

ifconfig  # Get your ip address from the docker interface, usually eth0. Mine is 172.25.0.2

mkdir /etc/consul.d
mkdir /var/lib/consul

consul agent -server \
    -bootstrap-expect=3 \
    -node=consulserver01 \
    -bind=172.25.0.2 \
    -data-dir=/var/lib/consul \
    -config-dir=/etc/consul.d \
    -encrypt=YGsICA9Fwq6TmFQI/qm4qIbdITrhvnHAsfZElu2czlk= \
    -ui \
    -client=0.0.0.0
```

### Open another terminal and start consulserver02:

```sh
docker-compose exec consulserver02 sh

ifconfig  # Get your ip address from the docker interface, usually eth0. Mine is 172.25.0.3

mkdir /etc/consul.d
mkdir /var/lib/consul

consul agent -server \
    -bootstrap-expect=3 \
    -node=consulserver02 \
    -bind=172.25.0.3 \
    -data-dir=/var/lib/consul \
    -config-dir=/etc/consul.d \
    -encrypt=YGsICA9Fwq6TmFQI/qm4qIbdITrhvnHAsfZElu2czlk=
```

### Open another terminal, go to the consulserver01 and join the consulserver02:

```sh
docker-compose exec consulserver01 sh

consul join 172.25.0.3  # This is the ip of consulserver02

consul members  # You should see two members now
```

### Open another terminal and start consulserver03:

```sh
docker-compose exec consulserver03 sh

ifconfig  # Get your ip address from the docker interface, usually eth0. Mine is 172.25.0.4

mkdir /etc/consul.d
mkdir /var/lib/consul

consul agent -server \
    -bootstrap-expect=3 \
    -node=consulserver03 \
    -bind=172.25.0.4 \
    -data-dir=/var/lib/consul \
    -config-dir=/etc/consul.d \
    -encrypt=YGsICA9Fwq6TmFQI/qm4qIbdITrhvnHAsfZElu2czlk=
```

### Open another terminal, go to the consulserver03 and join the consulserver02:

```sh
docker-compose exec consulserver03 sh

consul join 172.25.0.3  # This is the ip of consulserver02

consul members  # You should see three members now
```

