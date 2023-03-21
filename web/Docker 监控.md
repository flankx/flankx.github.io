# `Docker` 监控

## 1、启动`elasticsearch`并指定单节点模式

````bash
docker run -d --name elasticsearch \
  -p 9200:9200 \
  -e "discovery.type=single-node" \
  -e ES_JAVA_OPTS="-Xms256m -Xmx1024m" \
  elasticsearch:8.5.0
````

## 2、启动`kibana`

````bash
docker run -d --name kibana \
  --link elasticsearch:elasticsearch\
 -p 5601:5601 kibana:8.5.0
````

## 3、获取模板配置

````bash
curl -L -O <https://raw.githubusercontent.com/elastic/beats/8.5/deploy/docker/metricbeat.docker.yml>
````

## 4、启动`metricbeat`

````bash
docker run -d \
  --name=metricbeat \
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

metricbeat -e \
  -E output.elasticsearch.hosts=["<elasticsearch Ip>:9200"] \
  -E setup.kibana.host=<kibana Ip>:5601 \
  -E output.elasticsearch.username=elastic \
  -E output.elasticsearch.password=<password>
````

## 6、重启`metricbeat`并前往`kibana`查看数据

````bash
docker restart metricbeat
````
