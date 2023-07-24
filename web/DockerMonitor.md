# `Docker` 监控

## 1、启动`elasticsearch`并指定单节点模式

````bash
docker network create elastic

docker run -d --name elasticsearch \
  -p 9200:9200 \
  --net elastic \
  -e "discovery.type=single-node" \
  -e ES_JAVA_OPTS="-Xms1024m -Xmx1024m" \
  elasticsearch:8.5.0
````

## 2、启动`kibana`

````bash
docker run -d --name kibana \
  --net elastic \
  --link elasticsearch:elasticsearch \
 -p 5601:5601 kibana:8.5.0
````

## 3、获取模板配置

````bash
curl -L -O <https://raw.githubusercontent.com/elastic/beats/8.5/deploy/docker/metricbeat.docker.yml>

# 调整配置
output.elasticsearch:
  hosts: ["<es_url>"]
  username: "elastic"
  password: "<password>"
  # If using Elasticsearch's default certificate
  ssl.ca_trusted_fingerprint: "<es cert fingerprint>"
setup.kibana:
  host: "<kibana_url>"
````

## 4、启动`metricbeat`

````bash
docker run -d \
  --name=metricbeat \
  --net elastic \
  --user=root \
  --volume="$(pwd)/metricbeat.docker.yml:/usr/share/metricbeat/metricbeat.yml:ro" \
  --volume="/var/run/docker.sock:/var/run/docker.sock:ro" \
  --volume="/sys/fs/cgroup:/hostfs/sys/fs/cgroup:ro" \
  --volume="/proc:/hostfs/proc:ro" \
  --volume="/:/hostfs:ro" \
  docker.elastic.co/beats/metricbeat:8.5.0
````

## 5、进入`metricbeat`容器配置 `elasticsearch`和`kibana`

````bash
docker exec -it metricbeat bash
# 启用和配置 docker 模块
./metricbeat modules enable docker

# 加载kibana仪表盘
./metricbeat setup
./metricbeat -e
````

## 6、重启`metricbeat`并前往`kibana`查看数据

````bash
docker restart metricbeat
````

## 遇到异常问题

### 1、Kibana server is not ready yet

+ 解决方式：在 `kibana.yml` 配置 `kibana_system` 用户信息

````bash
elasticsearch.username: "kibana_system"
elasticsearch.password: "{password}"
````

### 2、dashboard 没有加载成功；报错lock by path.data

+ 解决方式：在对应 `path.data` 下删除 `xxx.lock` 文件
