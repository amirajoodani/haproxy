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


# ## Scenario Two : HAProxy ACL Configuration Examples

## Base Scenario
You have two Nginx servers (`web1` and `web2`) behind an HAProxy load balancer.

## 1. Path-Based Routing
**Scenario**: Route `/blog` to web1 and `/shop` to web2

```haproxy
frontend http_front
    bind *:80
    acl is_blog path_beg /blog
    acl is_shop path_beg /shop
    
    use_backend blog_servers if is_blog
    use_backend shop_servers if is_shop
    default_backend web_servers

backend blog_servers
    server web1 192.168.1.10:80 check

backend shop_servers
    server web2 192.168.1.11:80 check

backend web_servers
    server web1 192.168.1.10:80 check
    server web2 192.168.1.11:80 check
```

## 2. Client IP-Based Routing
**Scenario**: Route internal network (192.168.1.0/24) to web1 only

```haproxy
frontend http_front
    bind *:80
    acl is_internal src 192.168.1.0/24
    
    use_backend internal_servers if is_internal
    default_backend web_servers

backend internal_servers
    server web1 192.168.1.10:80 check

backend web_servers
    server web1 192.168.1.10:80 check
    server web2 192.168.1.11:80 check
```

## 3. Blocking Bad User Agents
**Scenario**: Block scanner user agents

```haproxy
frontend http_front
    bind *:80
    acl is_scanner hdr_sub(User-Agent) -i scanner
    
    http-request deny if is_scanner
    default_backend web_servers

backend web_servers
    server web1 192.168.1.10:80 check
    server web2 192.168.1.11:80 check
```
How It Works:
hdr_sub(User-Agent) examines the User-Agent header

-i makes the match case-insensitive

Looks for the substring "scanner" in User-Agent

Common patterns to block:

acl is_scanner hdr_sub(User-Agent) -i (nmap|wget|curl|nikto|dirbuster)

acl is_bot hdr_sub(User-Agent) -i (bot|crawl|spider)

## 4. Cookie-Based Routing
**Scenario**: Route based on `preferred_server` cookie

```haproxy
frontend http_front
    bind *:80
    acl prefers_web1 cook(preferred_server) -m str -i web1
    
    use_backend web1_servers if prefers_web1
    default_backend web_servers

backend web1_servers
    server web1 192.168.1.10:80 check

backend web_servers
    server web1 192.168.1.10:80 check
    server web2 192.168.1.11:80 check
```

## 5. Combined ACL Conditions
**Scenario**: Block specific IP from /admin

```haproxy
frontend http_front
    bind *:80
    acl is_bad_ip src 192.168.1.100
    acl is_admin path_beg /admin
    
    http-request deny if is_bad_ip is_admin
    default_backend web_servers

backend web_servers
    server web1 192.168.1.10:80 check
    server web2 192.168.1.11:80 check
```

## 6. ACL For Url Parameters
**Scenario**: send request with special url param to one server <br>
in this example if search ?region=east request goes to server 2 and search ?region=west request goes to server one

```haproxy
frontend http_front
    bind *:80
    acl urlparam1 url_param(region) -i -m str west
    acl urlparam2 url_param(region) -i -m str east
    use_backend web1 if urlparam1
    use_backend web2 if urlparam2
    default_backend web_servers
backend web1
    server web1 192.168.1.10:80 check
backend web2
    server web2 192.168.1.11:80 check
backend web_servers
    server web1 192.168.1.10:80 check
    server web2 192.168.1.11:80 check
```
## Key Features of HAProxy ACLs

✔ Define true/false conditions for traffic handling  
✔ Combine multiple conditions with logical operators  
✔ Support various matching criteria:  
  - IP addresses  
  - HTTP headers  
  - URL paths  
  - Cookies  
  - and more...  
✔ Enable smart routing decisions  
✔ Provide security controls  

## scenario3 : # HAProxy URL Redirection Examples

## 🔁 Redirect HTTP to HTTPS

```haproxy
frontend http_in
    bind *:80
    mode http
    redirect scheme https code 301 if !{ ssl_fc }
```
Listens on port 80 (HTTP).<br>

Redirects all traffic to HTTPS.<br>

Uses HTTP status code 301 (permanent redirect). <br>

if !{ ssl_fc } ensures the redirect only applies if the request is not already SSL/TLS. <br>

## 🔁 Redirect Specific URL to Another Domain
```haproxy
frontend http_in
    bind *:80
    mode http

    acl is_old_url path_beg /oldpath
    redirect location https://newdomain.com/newpath code 301 if is_old_url
```
Redirects requests from /oldpath to https://newdomain.com/newpath.<br>

Uses an Access Control List (ACL) to match paths beginning with /oldpath. <br>

## 🔁 Redirect Domain to Another Domain

```haproxy
frontend http_in
    bind *:80
    mode http

    acl is_old_domain hdr(host) -i olddomain.com
    redirect prefix https://newdomain.com code 301 if is_old_domain
```
Redirects all requests from olddomain.com to https://newdomain.com.<br>

Preserves the original URI path.<br>

Matches requests using the Host header. <br>

we can use drop query to do this (prefix-drop-query): <br>
nextsysadmin.com/path  --> nextsysadmin.ir <br>

it means all the path after url removed . <br>



