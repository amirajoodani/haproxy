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

## Scenario one : HTTP LoadBalancing

assume that we have below test environments : <br>
![haproxy1](https://github.com/user-attachments/assets/4190d7a9-ad6e-4607-88a7-1567741a681c)
first we install nginx on both two ubuntu 24.04 servers and install haproxy on haproxy node . <br>


