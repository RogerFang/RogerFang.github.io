---
title: Elasticsearch5 集群部署
date: 2017-06-20 09:53:06
categories:
- Elasticsearch
---
# 配置说明
![](/images/es/es-network-setting.png)

| 配置 | 描述 |
|--------|--------|
|http.port|配置对外提供服务的http端口，默认取9200~9299之间，如果端口冲突会自动使用其他端口。|
|transport.tcp.port|配置ES集群内数据通讯的端口，默认是9300，端口被占用会自动使用其他端口。|
|network.host|default为`_local_`，还可设为`_site_`、`_global_`|
|network.bind_host|可以设为`[_site_, _global_]`，监听请求的地址，**可以有多个**，默认为network.host。|
|network.publish_host|可以设为`[_site_]`，节点之间相互通信的地址，**只有一个**，默认会从network.host里选一个最好的。|
|discovery.zen.ping.unicast.hosts|节点加入集群所需。2.x 版本后，ES取消了默认的广播模式来发现master节点，需要使用该配置来指定发现master节点，default为`["127.0.0.1", "[::1]"]`。|

* `http.port`: 配置对外提供服务的http端口，默认取9200~9299之间，如果端口冲突会自动使用其他端口。
* `transport.tcp.port`: 配置ES集群内数据通讯的端口，默认是9300，端口被占用会自动使用其他端口。
* `discovery.zen.ping.unicast.hosts: ["x.x.x.x:9300"]`: 2.x 版本后，ES取消了默认的广播模式来发现master节点，需要使用该配置来指定发现master节点，default为`["127.0.0.1", "[::1]"]`。
* `network.host`: default为`_local_`，还可设为`_site_`、`_global_`。
* `network.bind_host: [_site_, _global_]`: 监听请求的地址，可以有多个，默认为network.host。
* `network.publish_host: [_site_]`: 节点之间相互通信的地址，只有一个，默认会从network.host里选一个最好的。

# 单机多节点
1. **node-1** yml配置
	```
    	cluster.name: mycluster
        node.name: ${HOSTNAME}-node-1
        network.bind_host: [_site_, _global_]
        network.publish_host: [_site_]
        http.port:9201
        transport.tcp.port: 9301
        discovery.zen.ping.unicast.hosts: ["x.x.x.x:9302"]
        http.cors.enabled: true
        http.cors.allow-origin: "*"
    ```
2. **node-2** yml配置
	```
    	cluster.name: mycluster
        node.name: ${HOSTNAME}-node-2
        network.bind_host: [_site_, _global_]
        network.publish_host: [_site_]
        http.port:9202
        transport.tcp.port: 9302
        discovery.zen.ping.unicast.hosts: ["x.x.x.x:9301"]
        http.cors.enabled: true
        http.cors.allow-origin: "*"
    ```

