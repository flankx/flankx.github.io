# `Docker`部署`elasticsearch`集群及`kibana`监控

## 1、修改配置`sysctl.conf`

```bash
root@ubuntu:~# vi /etc/sysctl.conf
# 添加配置
root@ubuntu:~# vm.max_map_count=262144
# 重新加载：
root@ubuntu:~# sysctl -p
```

## 2、`docker`安装及新增统一访问网路

### 2.1、`docker` 及 `docker-compose` 安装

+ [Docker 安装文档](https://docs.docker.com/engine/install/)
+ [Docker Compose 安装文档](https://docs.docker.com/compose/install/)

### 2.2、创建共享网络

```bash
root@ubuntu:~# docker network create flankx
root@ubuntu:~# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
dea464a9158c   bridge    bridge    local
f266d09eaa06   flankx    bridge    local
cf6a6a2681a3   host      host      local
e63d21b2e585   none      null      local
```

## 3、编排`docker`容器

### 3.1、容器服务及配置文件目录结构

```bash
# 目录结构
root@ubuntu:~/elasticsearch# tree -L 2
.
├── conf
│   ├── elastic-certificates.p12
│   ├── es-master.yml
│   ├── es-slave1.yml
│   ├── es-slave2.yml
│   ├── kibana.yml
│   └── metricbeat.docker.yml
├── docker-compose.yml
├── master
│   ├── data
│   ├── logs
│   └── plugins
├── slave1
│   ├── data
│   ├── logs
│   └── plugins
└── slave2
    ├── data
    ├── logs
    └── plugins
# 服务列表
root@ubuntu:~/elasticsearch# docker compose ps
NAME                IMAGE                                        COMMAND                  SERVICE             CREATED             STATUS    
es-master           elasticsearch:7.17.7                         "/bin/tini -- /usr/l…"   es-master           3 hours ago         Up 3 hours
es-slave1           elasticsearch:7.17.7                         "/bin/tini -- /usr/l…"   es-slave1           3 hours ago         Up 3 hours
es-slave2           elasticsearch:7.17.7                         "/bin/tini -- /usr/l…"   es-slave2           3 hours ago         Up 3 hours
kibana              kibana:7.17.7                                "/bin/tini -- /usr/l…"   kibana              3 hours ago         Up 3 hours
metricbeat          docker.elastic.co/beats/metricbeat:7.17.12   "/usr/bin/tini -- /u…"   metricbeat          3 hours ago         Up 3 hours
```

+ `docker-compose.yml`

```yml
version: "3"
services:
  es-master:
    container_name: es-master
    hostname: es-master
    networks:
      - flankx
    image: elasticsearch:7.17.7
    restart: always
    ports:
      - 9200:9200
      - 9300:9300
    volumes:
      - /root/elasticsearch/conf/es-master.yml:/usr/share/elasticsearch/config/elasticsearch.yml
      - /root/elasticsearch/master/data:/usr/share/elasticsearch/data
      - /root/elasticsearch/slave2/logs:/usr/share/elasticsearch/logs
      - /root/elasticsearch/master/plugins:/usr/share/elasticsearch/plugins
    environment:
      - "ES_JAVA_OPTS=-Xms1G -Xmx1G"

  es-slave1:
    container_name: es-slave1
    networks:
      - flankx
    image: elasticsearch:7.17.7
    restart: always
    ports:
      - 9201:9200
      - 9301:9300
    volumes:
      - /root/elasticsearch/conf/es-slave1.yml:/usr/share/elasticsearch/config/elasticsearch.yml
      - /root/elasticsearch/slave1/data:/usr/share/elasticsearch/data
      - /root/elasticsearch/slave2/logs:/usr/share/elasticsearch/logs
      - /root/elasticsearch/slave1/plugins:/usr/share/elasticsearch/plugins
    environment:
      - "ES_JAVA_OPTS=-Xms1G -Xmx1G"

  es-slave2:
    container_name: es-slave2
    image: elasticsearch:7.17.7
    networks:
      - flankx
    restart: always
    ports:
      - 9202:9200
      - 9302:9300
    volumes:
      - /root/elasticsearch/conf/es-slave2.yml:/usr/share/elasticsearch/config/elasticsearch.yml
      - /root/elasticsearch/slave2/data:/usr/share/elasticsearch/data
      - /root/elasticsearch/slave2/logs:/usr/share/elasticsearch/logs
      - /root/elasticsearch/slave2/plugins:/usr/share/elasticsearch/plugins
    environment:
      - "ES_JAVA_OPTS=-Xms1G -Xmx1G"

  kibana:
    container_name: kibana
    hostname: kibana
    networks:
      - flankx
    image: kibana:7.17.7
    restart: always
    ports:
      - 5601:5601
    volumes:
      - /root/elasticsearch/conf/kibana.yml:/usr/share/kibana/config/kibana.yml
    environment:
      - elasticsearch.hosts=http://es-master:9200
    depends_on:
      - es-master
      - es-slave1
      - es-slave2

  metricbeat:
    container_name: metricbeat
    hostname: metricbeat
    networks:
      - flankx
    user: root
    image:  docker.elastic.co/beats/metricbeat:7.17.12
    restart: always
    volumes:
      - /root/elasticsearch/conf/metricbeat.docker.yml:/usr/share/metricbeat/metricbeat.yml:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - /sys/fs/cgroup:/hostfs/sys/fs/cgroup:ro
      - /proc:/hostfs/proc:ro
      - /:/hostfs:ro
    depends_on:
      - kibana
      - es-master
      - es-slave1
      - es-slave2

networks:
  flankx:
    external: true
```

+ `es-master.yml`

```yml
# 集群名称
cluster.name: es-cluster
# 节点名称
node.name: es-master
# 是否可以成为master节点
node.master: true
# 是否允许该节点存储数据,默认开启
node.data: false
# 网络绑定
network.host: 0.0.0.0
# 设置对外服务的http端口
http.port: 9200
# 设置节点间交互的tcp端口
transport.port: 9300
# 集群发现
discovery.seed_hosts:
  - es-master
  - es-slave1
  - es-slave2
# 手动指定可以成为 mater 的所有节点的 name 或者 ip，这些配置将会在第一次选举中进行计算
cluster.initial_master_nodes:
  - es-master
```

+ `es-slave1.yml`

```yml
# 集群名称
cluster.name: es-cluster
# 节点名称
node.name: es-slave1
# 是否可以成为master节点
node.master: true
# 是否允许该节点存储数据,默认开启
node.data: true
# 网络绑定
network.host: 0.0.0.0
# 设置对外服务的http端口
http.port: 9201
# 设置节点间交互的tcp端口
#transport.port: 9301
# 集群发现
discovery.seed_hosts:
  - es-master
  - es-slave1
  - es-slave2
# 手动指定可以成为 mater 的所有节点的 name 或者 ip，这些配置将会在第一次选举中进行计算
cluster.initial_master_nodes:
  - es-master
```

+ `es-slave2.yml`

```yml
# 集群名称
cluster.name: es-cluster
# 节点名称
node.name: es-slave2
# 是否可以成为master节点
node.master: true
# 是否允许该节点存储数据,默认开启
node.data: true
# 网络绑定
network.host: 0.0.0.0
# 设置对外服务的http端口
http.port: 9202
# 设置节点间交互的tcp端口
#transport.port: 9302
# 集群发现
discovery.seed_hosts:
  - es-master
  - es-slave1
  - es-slave2
# 手动指定可以成为 mater 的所有节点的 name 或者 ip，这些配置将会在第一次选举中进行计算
cluster.initial_master_nodes:
  - es-master
```

+ `kibana.yml`

```yml
# 服务端口
server.port: 5601
# 服务IP
server.host: "0.0.0.0"
# ES
elasticsearch.hosts: ["http://es-master:9200"]
# 汉化
i18n.locale: "zh-CN"
```

+ `metricbeat.docker.yml` 详见 `Docker` 监控

### 3.2、安装分词插件

+ [下载对应版本分次插件](https://github.com/medcl/elasticsearch-analysis-ik/releases)
+ 解压后指定文件夹`ik`中分别挂载到各个`es node`的`plugins`文件夹中

```bash
root@ubuntu:~/elasticsearch# cp -r ik/ elasticsearch/master/plugins/
root@ubuntu:~/elasticsearch# cp -r ik/ elasticsearch/slave1/plugins/
root@ubuntu:~/elasticsearch# cp -r ik/ elasticsearch/slave2/plugins/
```

+ 在kibana控制台的开发工具下测试IK分词器

```json
POST _analyze
{
  "analyzer":"ik_smart",
  "text":"以下判断正确的是2"
}

// 返回
{
  "tokens" : [
    {
      "token" : "以下",
      "start_offset" : 0,
      "end_offset" : 2,
      "type" : "CN_WORD",
      "position" : 0
    },
    {
      "token" : "判断",
      "start_offset" : 2,
      "end_offset" : 4,
      "type" : "CN_WORD",
      "position" : 1
    },
    {
      "token" : "正确",
      "start_offset" : 4,
      "end_offset" : 6,
      "type" : "CN_WORD",
      "position" : 2
    },
    {
      "token" : "的",
      "start_offset" : 6,
      "end_offset" : 7,
      "type" : "CN_CHAR",
      "position" : 3
    },
    {
      "token" : "是",
      "start_offset" : 7,
      "end_offset" : 8,
      "type" : "CN_CHAR",
      "position" : 4
    },
    {
      "token" : "2",
      "start_offset" : 8,
      "end_offset" : 9,
      "type" : "ARABIC",
      "position" : 5
    }
  ]
}

```

### 3.3、启用安全

+ 生成 .p12 文件

```bash
# 进入 ES 容器
root@ubuntu:~/elasticsearch# docker exec -it es-master bash
# 生成 .p12 文件
root@ubuntu:~/elasticsearch# bin/elasticsearch-certutil ca
root@ubuntu:~/elasticsearch# bin/elasticsearch-certutil cert --ca elastic-stack-ca.p12
# 将 .p12 拷贝到宿主机的 ~/elasticsearch/config/ 目录下
root@ubuntu:~/elasticsearch# docker cp -a es:/usr/share/elasticsearch/elastic-certificates.p12 /root/elasticsearch/config/
# 将 .p12 推送到每个节点
root@ubuntu:~/elasticsearch# ...
```

+ 开启安全配置;每个node节点的配置文件追加安全配置

```bash
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.keystore.type: PKCS12
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.keystore.path: elastic-certificates.p12
xpack.security.transport.ssl.truststore.path: elastic-certificates.p12
xpack.security.transport.ssl.truststore.type: PKCS12
```

+ 重启容器并设置密码

```bash
root@ubuntu:~/elasticsearch# docker compose down
root@ubuntu:~/elasticsearch# docker compose up -d
root@ubuntu:~/elasticsearch# docker exec -it es-master bash
root@ubuntu:~/elasticsearch# bin/elasticsearch-setup-passwords auto
```
