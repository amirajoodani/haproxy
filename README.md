# HAProxy - High Availability Proxy

**HAProxy** (High Availability Proxy) is a free, open-source **load balancer** and **proxy server** designed to improve the performance, reliability, and security of web applications. It distributes traffic across servers to ensure high availability and scalability.

## 🔥 Key Features

### 1. **Load Balancing**
   - Distributes traffic across backend servers (HTTP/TCP/UDP).
   - Algorithms: **Round Robin**, Least Connections, Source IP Hash, and more.

### 2. **High Availability (HA)**
   - Automatic failover with **health checks** (active/passive).
   - Removes failed servers from the pool.

### 3. **Reverse Proxy & SSL Termination**
   - Acts as an intermediary for clients/servers.
   - Offloads SSL/TLS encryption (**HTTPS termination**).

### 4. **Layer 4 (TCP) & Layer 7 (HTTP) Support**
   - Balances traffic at both transport and application layers.

### 5. **Advanced Traffic Routing**
   - Uses **ACLs (Access Control Lists)** for rules based on:
     - URL paths, headers, cookies, etc.

### 6. **Security & DDoS Protection**
   - Rate limiting, connection throttling.
   - Basic **WAF (Web Application Firewall)** capabilities.

### 7. **Logging & Monitoring**
   - Detailed logs for debugging.
   - Integrates with **Prometheus**, **StatsD**, and **Syslog**.

### 8. **Performance & Scalability**
   - Handles **millions of requests/sec** with low latency.
   - Multithreading for optimal CPU usage.

### 9. **Modern Protocol Support**
   - **HTTP/2**, **WebSockets**, **gRPC**.

### 10. **Dynamic Configuration (Runtime API)**
   - Modify settings **without restarting** (via CLI/API).

## 🚀 Common Use Cases
- **Web Application Load Balancing**
- **API Gateway & Microservices**
- **Database Load Balancing** (MySQL, PostgreSQL)
- **Kubernetes Ingress Controller**
- **DDoS Protection & Traffic Filtering**

## 📜 Example Config Snippet
```haproxy
frontend http_front
   bind *:80
   mode http
   default_backend http_back

backend http_back
   balance roundrobin
   server server1 192.168.1.10:80 check
   server server2 192.168.1.11:80 check
```
## Scenario one : HTTP LoadBalancing

assume that we have below test environments : <br>
![haproxy1](https://github.com/user-attachments/assets/4190d7a9-ad6e-4607-88a7-1567741a681c) <br>
first we install nginx on both two ubuntu 24.04 servers and install haproxy on haproxy node . <br>
now we copy config file for backup and then change the config file : <br>
```bash
# sudo cp /etc/haproxy/haproxy.conf /etc/haproxy/haproxy.conf.back
# sudo vi /etc/haproxy/haproxy.conf
```
haproxy config has <b>four</b> main parts : <br>
1- Global: <br>
the main part for haproxy proccessing <br>
2- defaults: <br>
some defaults configuaraion like mode is store here . if in that file , we use other mode like tcp , <b>tcp</b> is the main mode and overwrite the default config on top of the file <br>
3- frontend: <br>
the part to define reverse proxy . <br>
4- backend: <br>
the pool of backnd server for answering requests . <br>

note : for checking haproxy config we can use : <br>
```bash
# sudo haproxy -c -f /etc/haproxy/haproxy.cfg
```
<b>note:</b> we have three timeout in default config: <br>
1- <b>timeout connect : </b><br>
the time that haproxy check the backend server for up and running . if the backend server does not answer to haproxy , server failed and haproxy does not send data to it . <br>
2- <b>timeout client : </b><br>
time wait for answering client to proccessed data to haproxy . <br>
3- <b>timeout server : </b><br>
time wait for answering backend server to proccesses data to haproxy . <br>

now we import below config and reload haproxy :
```haproxy
frontend http_front
   bind *:80
   mode http
   default_backend http_back

backend http_back
   balance roundrobin
   server server1 192.168.1.59:80 check
   server server2 192.168.1.12:80 check
```
```bash
# sudo systemctl reload haproxy
```
now based on roundrobin algorithm one request send to server 1 and next one send to server 2 <br>
![2](https://github.com/user-attachments/assets/303599c1-dee2-4f8c-870e-30a76d2034f8) <br>



