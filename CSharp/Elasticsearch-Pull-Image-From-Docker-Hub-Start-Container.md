# Elasticsearch 系列 - 在 Windows 作業系統上安裝 Docker


docker network create elastic

docker run -d -e ELASTIC_PASSWORD=elastic --name elasticsearch --net elastic -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:8.11.1

https://localhost:9200/

user: elastic
password: elastic

docker run --name kib01 --net elastic -p 5601:5601 docker.elastic.co/kibana/kibana:8.11.1

docker exec -it elasticsearch /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana

docker exec -it kib01 bash
cd bin/
sh kibana-verification-code

http://localhost:5601/












```
docker network create elastic
```

https://hub.docker.com/_/elasticsearch

```
docker pull docker.elastic.co/elasticsearch/elasticsearch:8.11.1
```

docker run -d -e ELASTIC_PASSWORD=elastic --name elasticsearch --net elastic -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:8.11.1

user: elastic
password: elastic


docker pull docker.elastic.co/kibana/kibana:8.11.1

docker run --name kib01 --net elastic -p 5601:5601 docker.elastic.co/kibana/kibana:8.11.1

docker exec -it elasticsearch /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana


```
# Remove the Elastic network
docker network rm elastic

# Remove Elasticsearch containers
docker rm elasticsearch

# Remove the Kibana container
docker rm kib01
```










```
wget https://artifacts.elastic.co/cosign.pub
cosign verify --key cosign.pub docker.elastic.co/elasticsearch/elasticsearch:8.11.1
```

```
docker run --name es01 --net elastic -p 9200:9200 -it -m 1GB docker.elastic.co/elasticsearch/elasticsearch:8.11.1
```

```
docker exec -it es01 /usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic
docker exec -it es01 /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana
```

```
export ELASTIC_PASSWORD="your_password"
```

```
docker cp es01:/usr/share/elasticsearch/config/certs/http_ca.crt .
```

```
curl --cacert http_ca.crt -u elastic:$ELASTIC_PASSWORD https://localhost:9200
```

```
docker pull docker.elastic.co/kibana/kibana:latest
```

```
docker run --name kib01 --net elastic -p 5601:5601 docker.elastic.co/kibana/kibana:latest
```

```
docker exec -it es01 /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana
```

```
docker exec -it es01 /usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic
```

```
# Remove the Elastic network
docker network rm elastic

# Remove Elasticsearch containers
docker rm es01
docker rm es02

# Remove the Kibana container
docker rm kib01
```








