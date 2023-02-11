````conf
user www-data;          # 运行用户
worker_processes auto;  # 启动进程,通常设置成和cpu的数量相等
pid /run/nginx.pid;     # PID文件
include /etc/nginx/modules-enabled/*.conf; # 包含另一个 file 或匹配指定 mask 的文件到配置中。包含的文件应该由语法正确的指令和块组成。
# 工作模式及连接数上限
events {
    # 指定要使用的连接处理方式。通常不需要明确指定它，因为 nginx 将默认使用最有效的方式。
    use epoll;                  # epoll是多路复用IO(I/O Multiplexing)中的一种方式,但是仅用于linux2.6以上内核,可以大大提高nginx的性能
    worker_connections 1024;    # 设置工作进程可以打开的同时连接的最大数量。
    #multi_accept on;           # 如果 multi_accept 被禁用，工作进程将一次接受一个新连接。否则，工作进程将一次接受所有新连接。
}

# 设定http服务器，利用它的反向代理功能提供负载均衡支持 
http {
    server_tokens off;          # 隐藏Nginx版本号

    sendfile on;                # 必须设为 on,如果用来进行下载等应用磁盘IO重负载应用，可设置为 off，以平衡磁盘与网络I/O处理速度，降低系统的uptime.       
    tcp_nopush on;              # 只有在启用了 sendfile 之后才生效。启用它之后，数据包会累计到一定大小之后才会发送，减小了额外开销，提高网络效率。
    tcp_nodelay on;             # 启用后会禁用 Nagle 算法，尽快发送数据；Nginx 只会针对处于 keep-alive 状态的 TCP 连接才会启用 tcp_nodelay。
    keepalive_timeout 65;       # 连接超时时间
    types_hash_max_size 2048;   # 影响散列表的冲突率。此值越大，消耗内存则越多，散列key冲突率就越低，检索速度就更快；值越小，消耗内存就越少，冲突率相对就会高一些。

    include /etc/nginx/mime.types;          # 设定mime类型,类型由mime.type文件定义
    default_type application/octet-stream;  # 定义响应的默认 MIME 类型。可以使用 types 指令对 MIME 类型的文件扩展名进行映射。
    gzip on;                                # 启用gzip功能，对响应数据进行在线实时压缩，减少数据传输量。

    server {
        listen 80;                              # 此server块监听的端口号。
        #listen [::]:80;                        # IPv6 地址在方括号中指定
        server_name server1.example.com;        # 你的域名

        location / {                            # 默认路径
            proxy_redirect off;                 # off 参数取消所有 proxy_redirect 指令对当前级别的影响：
            proxy_pass http://127.0.0.1:8080;   # 设置代理服务器的协议、地址以及应映射位置的可选 URI。协议可以指定 http 或 https。可以将地址指定为域名或 IP 地址，以及一个可选端口号：
            proxy_http_version 1.1;             # 设置代理的 HTTP 协议版本。默认情况下，使用 1.0 版本。建议将 1.1 版与 keepalive 连接和 NTLM 身份验证配合使用。
            # 允许将字段重新定义或附加到传递给代理服务器的请求 header。
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }

    server {
        listen 443 ssl;                         # 启用https方式，监听 443 端口
        #listen [::]:443;                       # IPv6 地址在方括号中指定
        server_name server2.example.com;        # 你的域名

        ssl_certificate /etc/skey/a.crt;        # 公钥位置
        ssl_certificate_key /etc/skey/a.key;    # 私钥位置
        ssl_session_timeout 1d;                 # session超时时间。
        ssl_session_cache shared:MozSSL:10m;    # 配置加密session重用。
        ssl_session_tickets off;                # 通过 TLS 会话票证启用或禁用会话复用。
        ssl_protocols TLSv1.2 TLSv1.3;          # 启用指定的协议。
        ssl_prefer_server_ciphers off;          # 指定当使用 SSLv3 和 TLS 协议时，服务器加密算法应优先于客户端加密算法。

        location / {
            proxy_redirect off;
            proxy_pass http://127.0.0.1:8080;
            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }

    server {
        listen 80;
        #listen [::]:80;
        erver_name server2.example.com;
        location / {
            rewrite ^(.*)$ https://$host$1 permanent;   # 重定向到https访问
        }          
    }

}
````