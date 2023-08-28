# 作業 - 14. 架設 Nginx 反向代理伺服器

## Load Balance

https://stackoverflow.com/questions/29910778/load-balancing-with-nginx-using-hash-method
https://stackoverflow.com/questions/21173496/haproxy-vs-nginx

免費版有些功能做不到

### Round Robin

預設的 loadbalance 是這個，會平均分配 Request

##### Requests are distributed evenly across the servers, with server weights taken into consideration. This method is used by default (there is no directive for enabling it)

#### weight

default 是 1，如果設定為 5，有 6 個 Request 的話，會有五個都連到 weight=5 的

```
    upstream app {
        server 192.168.50.131:80;
        server 192.168.50.132:80 weight=5;
    }

    server {
        listen 80;
	server_name et.localhost;
        location / {
            proxy_pass http://app;
        }
    }
```

### Least Connections

會先優先把 Request 分配給連線數少的

##### A request is sent to the server with the least number of active connections, again with server weights taken into consideration

```
    upstream app {
	ip_hash;
        server 192.168.50.131:80;
        server 192.168.50.132:80;
    }

    server {
        listen 80;
	server_name et.localhost;
        location / {
            proxy_pass http://app;
        }
    }

```

### IP Hash

用前三組 IP(IPV4)或全部 IP(IPv4)來計算 hash，讓同個 hash 連到同個 server

##### The server to which a request is sent is determined from the client IP address. In this case, either the first three octets of the IPv4 address or the whole IPv6 address are used to calculate the hash value. The method guarantees that requests from the same address get to the same server unless it is not available.

```
    upstream app {
	    ip_hash;
        server 192.168.50.131:80;
        server 192.168.50.132:80;
    }

    server {
        listen 80;
	server_name et.localhost;
        location / {
            proxy_pass http://app;
        }
    }

```

### Generic Hash

根據 $request_uri 或其他變數，會指到固定的 Server

##### The server to which a request is sent is determined from a user‑defined key which can be a text string, variable, or a combination. For example, the key may be a paired source IP address and port, or a URI as in this example

```
    upstream app {
	    hash $request_uri consistent;
        server 192.168.50.131:80;
        server 192.168.50.132:80;
    }

    server {
        listen 80;
	server_name et.localhost;
        location / {
            proxy_pass http://app;
        }
    }

```
