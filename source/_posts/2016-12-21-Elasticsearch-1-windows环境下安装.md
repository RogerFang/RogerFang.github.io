---
title: 'Elasticsearch(1): windows环境下安装'
date: 2016-12-21 16:13:17
categories:
- Elasticsearch
tags:
---

# Elasticsearch安装
1. 官网下载: `https://www.elastic.co/downloads/elasticsearch`
2. 解压，修改配置文件
在config/elasticsearch.yml配置文件中添加以下参数:
> http.cors.enabled:true
> http.cors.allow-origin:"*"

3. 启动
运行bin/elasticsearch.bat, 访问`http://localhost:9200`

# head插件安装
在5.0版本中不支持直接安装head插件，需要启动一个服务

1. 下载插件并解压
`https://github.com/mobz/elasticsearch-head`

2. 安装grunt
grunt是一个很方便的构建工具，可以进行打包压缩、测试、执行等等的工作，5.0里的head插件就是通过grunt启动的。
`npm install grunt-cli`
查看是否安装: `grunt -version`

3. 修改head配置
目录：`head/Gruntfile.js`，增加`hostname: '*'`实现跨机器访问。
```javascript
connect: {
			server: {
				options: {
					port: 9100,
					base: '.',
					hostname: '*',
					keepalive: true
				}
			}
		}
```
目录: `head/_site/app.js`，修改head的连接地址:`this.base_uri = this.config.base_uri || this.prefs.get("app-base_uri") || "http://localhost:9200";`

4. 安装依赖包
在head目录中，执行`npm install`下载依赖的包。

5. 启动nodejs
`grunt server`

6. 访问
访问head插件：`http://localhost:9100`