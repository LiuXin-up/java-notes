# docker 部署es与kibana

## 注意事项：es与kibana的版本需要一致

## 一、拉取镜像

```bash
docker pull elasticsearch:8.12.0

docker pull kibana:8.12.0
```

## 二、部署es

`--network es-net `:要加入的网络名称

```bash
docker run -d \
  --name es \
    -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" \
    -e "discovery.type=single-node" \
    -v es-data:/usr/local/elasticsearch7.12.1/data \
    -v es-plugins:/usr/local/elasticsearch7.12.1/plugins \
    -v es-logs:/usr/local/elasticsearch7.12.1/logs \
    --privileged \
    --network es-net \
    -p 9200:9200 \
    -p 9300:9300 \
elasticsearch:8.12.0
```

## 三、配置es

使用root用户进入es容器

```bash
 docker exec -u 0 -it elasticsearch /bin/bash
```

在es容器中，安装vim编辑器

```bash
apt-get update 
apt-get install -y vim
```

进入config目录，编辑yml配置文件

```bash
cd /config
vim elasticsearch.yml
```

在yml中，添加以下配置

```yaml
# =================== 是否将GeoIp功能开启采集。在默认的启动下是会去官网的默认地址下获取最新的Ip的GEO信息 ===================
ingest.geoip.downloader.enabled: false

# =================== 是否在集群中使用机器学习功能 ===================
xpack.ml.enabled: false

# =================== 是否开启跨域访问 ===================
http.cors.enabled: true

# =================== 能访问es的地址限制,*号表示无限制 ===================
http.cors.allow-origin: "*"
```

配置完成后，重新启动es

## 四、部署kibana

```bash
docker run -d --name kibana   --network=es-net   -p 5601:5601  kibana:8.12.0
```

## 五、配置kibana（可选项）

使用root用户进入kibana容器

```bash
 docker exec -u 0 -it kibana /bin/bash
```

在es容器中，安装vim编辑器

```bash
apt-get update 
apt-get install -y vim
```

进入config目录，编辑yml配置文件

```bash
cd /config
vim kibana.yml
```

在yml中添加以下配置

```yaml
# =================== 设置kibana为中文 ===================
i18n.locale: "zh-CN"
```

配置后，重新启动kibana