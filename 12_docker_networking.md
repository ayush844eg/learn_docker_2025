
# 🧠 Overview: What is Docker Networking?

Docker networking is the system by which **containers** communicate:

* With each other
* With the **host**
* With the **outside world**

It's made possible through **network drivers**, **virtual interfaces**, **bridges**, **namespaces**, and **iptables rules** managed by Docker.

---

## 🔌 1. What Are Docker Network Drivers?

A **network driver** is a plugin that implements a specific **networking strategy** or configuration in Docker. It determines how containers connect to each other and the outside world.

Docker includes **built-in drivers** (out-of-the-box), but also supports **custom drivers** via plugins (for example, Calico, Weave, etc.).

### 🛠 Built-in Network Drivers:

| Driver      | Description                                                                  |
| ----------- | ---------------------------------------------------------------------------- |
| **bridge**  | Default driver for standalone containers; creates a private internal network |
| **host**    | Removes isolation between container and host network                         |
| **overlay** | Creates networks spanning multiple Docker hosts (Swarm only)                 |
| **macvlan** | Assigns MAC addresses to containers, making them appear as physical hosts    |
| **none**    | Disables all networking for a container                                      |

---

## 🏗️ 2. The Default Docker Network Setup

When Docker is installed and started:

* Docker creates a **default bridge network** called `bridge`
* Also creates the **`docker0`** bridge interface on the host
* This is visible by running:

  ```bash
  docker network ls
  ```

### Default Networks Docker Creates:

```bash
NETWORK ID     NAME      DRIVER    SCOPE
a1b2c3d4e5f6   bridge    bridge    local
b2c3d4e5f6g7   host      host      local
c3d4e5f6g7h8   none      null      local
```

### Default Bridge (`bridge`)

* This is used when no `--network` is specified.
* It:

  * Creates a **NAT**-based network using **iptables**
  * Assigns IP addresses from a private subnet (e.g., `172.17.0.0/16`)
  * Connects containers to the `docker0` bridge interface

---

## 🔗 3. How Docker Networking Works Internally

Docker relies on core Linux features:

### 🧱 Linux Network Namespaces

Each container runs in its own **network namespace**, which isolates:

* IP addresses
* Routing tables
* Interfaces
* DNS settings

### 🧶 Virtual Ethernet Interfaces (veth pairs)

* A **veth pair** is like a virtual cable connecting the container to the host.
* One end is inside the container (e.g., `eth0`)
* One end is on the host and connected to the bridge (`vethXYZ` → `docker0`)

### 🔧 Example:

For a container connected to the bridge network:

* Host:

  * Interface: `docker0`
  * IP: `172.17.0.1`
* Container:

  * Interface: `eth0`
  * IP: `172.17.0.2`
* Communication:

  * Packets from container go through `veth`, hit `docker0`, routed/NATed to the outside world.

---

## 🔐 4. Docker's Use of iptables

Docker dynamically configures **iptables** rules to:

* Allow/deny traffic
* Handle **NAT (masquerading)** for internet access
* Perform **port forwarding** (for `-p` or `--publish`)

Example:

```bash
docker run -d -p 8080:80 nginx
```

Creates a rule like:

```
iptables -t nat -A DOCKER -p tcp --dport 8080 -j DNAT --to-destination 172.17.0.2:80
```

---

## 🌉 5. User-Defined Bridge Networks vs Default Bridge

### Default `bridge`:

* Limited DNS-based name resolution
* No automatic service discovery

### User-Defined `bridge`:

```bash
docker network create my_bridge
```

* Allows **container name resolution via internal DNS**
* Better isolation and easier management

```bash
docker run -d --network my_bridge --name web nginx
docker run -d --network my_bridge --name app alpine
```

Now `app` can ping `web` by name.

---

## 🌐 6. Host Network Driver

* Shares host's IP and ports
* No network namespace isolation

```bash
docker run --network host nginx
```

### Pros:

* Zero overhead
* Ideal for performance-sensitive applications

### Cons:

* No container isolation
* Can conflict with host services

---

## 🌍 7. Overlay Networks (for Docker Swarm)

Overlay networks allow containers on **different Docker hosts** to communicate as if on the same network.

* Used with **Swarm mode**
* Built using **VXLAN** tunnels
* Traffic is encrypted by default (if set)

```bash
docker network create \
  --driver overlay \
  --attachable \
  my_overlay
```

This enables scalable, distributed microservices.

---

## 📶 8. Macvlan Networks

Lets containers appear as **physical devices on the LAN**.

### Setup:

* Assign MAC & IP from your actual network
* Can talk to other devices on your LAN (e.g., routers, printers)

```bash
docker network create -d macvlan \
  --subnet=192.168.1.0/24 \
  --gateway=192.168.1.1 \
  -o parent=eth0 macvlan_net
```

### Use case:

* Containers that need to be on the same level as physical servers
* Often used in **legacy systems** or when container IPs need to be reachable externally

---

## 🚫 9. None Network

* Completely disables networking
* Useful for security, testing, or when you manually configure networking

```bash
docker run --network none alpine
```

---

## 🧠 10. DNS and Service Discovery

* Docker has a built-in **DNS server** that works inside user-defined networks
* Containers can use service/container **names** to reach each other (like `redis`, `db`, etc.)
* Works only in **user-defined networks** (not the default `bridge`)

---

## 📦 11. Docker Compose and Networking

Docker Compose creates a **default network** for your services, unless you specify otherwise.

```yaml
version: '3'
services:
  web:
    image: nginx
  app:
    image: alpine
    command: ping web
```

* Compose will create a network: `projectname_default`
* `web` and `app` can talk using names: `web`, `app`

You can also define custom networks:

```yaml
networks:
  mynet:
services:
  web:
    networks:
      - mynet
```

---

## 🧾 Summary

| Component                 | Description                                                               |
| ------------------------- | ------------------------------------------------------------------------- |
| **Drivers**               | Plugins that define the networking behavior (bridge, host, overlay, etc.) |
| **Default bridge**        | Created on install, used if no custom network is specified                |
| **Namespaces**            | Isolate containers' networking                                            |
| **veth pairs**            | Virtual cables connecting containers to host bridge                       |
| **iptables**              | Manages NAT, port forwarding, isolation                                   |
| **Docker DNS**            | Internal DNS for container name resolution                                |
| **User-defined networks** | Allow name resolution, better management, isolation                       |

---
---
---


## creating a custom network

### terminal 1:

```shell

ayush@lucas:~$ docker network create -d bridge my_custome_network
334adde777bac345bbe51093a49e375f5fa1999675b594b0666aa80025af3157
ayush@lucas:~$ docker network ls
NETWORK ID     NAME                          DRIVER    SCOPE
2d4e6a0bceb6   bridge                        bridge    local
31dc71cc6803   eg-face-recognition_default   bridge    local
f6ae8728cab0   eg-image-resizer_default      bridge    local
db68933b2a18   egreact_default               bridge    local
188a34dbc327   eventgraphia-apis_default     bridge    local
a92f935ab0d0   eventgraphia_net              bridge    local
6097da4e9840   host                          host      local
334adde777ba   my_custome_network            bridge    local
fc38592d04db   none                          null      local
ayush@lucas:~$ docker run -it --network=my_custome_network --name tony_stark ubuntu
root@e1801f1863eb:/# 
exit
ayush@lucas:~$ docker network inspect my_custome_network 
[
    {
        "Name": "my_custome_network",
        "Id": "334adde777bac345bbe51093a49e375f5fa1999675b594b0666aa80025af3157",
        "Created": "2025-08-17T14:08:09.792352736+05:30",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv4": true,
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.23.0.0/16",
                    "Gateway": "172.23.0.1"
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
            "0160208ed5d9747eee4959645656f786f9a4b917acc50d4fc7e82de9c4c3eb7a": {
                "Name": "hulk",
                "EndpointID": "1f5b0fb4e79e152abe805f425609f84467f8002621283a14c25b55fe8d4d1f51",
                "MacAddress": "a6:26:e7:81:6b:b1",
                "IPv4Address": "172.23.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
ayush@lucas:~$ 


```



### terminal 2:

```shell

ayush@lucas:~$ docker run -it --network=my_custome_network --name hulk busybox
/ # ping tony_stark
PING tony_stark (172.23.0.2): 56 data bytes
64 bytes from 172.23.0.2: seq=0 ttl=64 time=0.198 ms
64 bytes from 172.23.0.2: seq=1 ttl=64 time=0.151 ms
64 bytes from 172.23.0.2: seq=2 ttl=64 time=0.142 ms
64 bytes from 172.23.0.2: seq=3 ttl=64 time=0.255 ms
64 bytes from 172.23.0.2: seq=4 ttl=64 time=0.190 ms
64 bytes from 172.23.0.2: seq=5 ttl=64 time=0.148 ms
64 bytes from 172.23.0.2: seq=6 ttl=64 time=0.156 ms

```