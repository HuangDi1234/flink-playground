# Flink Playground

Flink Playground 是用来进行 Flink 任务调试的容器环境，内部包含消息队列、消息生产者、Flink JobManager、Flink TaskManager 等调试的必备服务。

该环境基于 [docker-compose](https://docs.docker.com/compose/) 支持一键拉起。由于使用了 host network mode 等特性，所以该环境无法在 Docker Desktop 等不支持 host network mode 的环境中使用。

## 用户权限

Flink 官方镜像会在 Dockerfile 中创建一个用户 flink:flink 并使用该用户运行容器，这会导致 volume 的目录出现权限异常。目前有两种解决方案：

*   在 docker-compose.yml 中指定启动 Flink JobManager 和 Flink TaskManager 的用户为 "1000"。如果挂载进容器的路径属于其他用户，也可以将用户 uid 传入，以解决权限问题。可以使用 mount volume，无法通过环境变量安装插件

*   不在 docker-compose.yml 中指定用户。可以通过环境变量安装插件，无法使用 mount volume

请用户根据需求自行选择，修改配置

## 启动

本环境完全基于开源镜像，均可通过 Dockerhub 下载，为了加快速度，目前都来自位于 docker.shiyou.kingsoft.com 的镜像仓库。

根目录的 docker-compose.yml 文件即可拉起全部环境，第一次启动可能需要较长时间用于下载镜像。

```bash
cd flink-playground
docker-compose up -d
```

## 配置变动

需要注意 Flink 会自动修改 config 目录下的两个配置文件，在文件的末尾追加一些配置信息，某些情况下可能导致无法重新启动，配置文件中已经做出了标注，附加信息均可以手动删除。

## 目录结构

```
.
├── README.md
├── config
│   ├── flink-jobmanager-conf.yaml  
│   ├── flink-taskmanager-conf.yaml 
│   └── fluent-bit.conf （在此修改写入 Kafka 的数据格式）
├── data
│   ├── fluent-bit      
│   │   └── output.txt  （所有 fluent-bit 写入 Kafka 的数据均会有一份在此文件中，可以删除）
│   ├── flink
│   │   ├── ha          （文件夹中的内容可删）
│   │   ├── jars        （保存上传至 JobManager 的 Jar 文件，内容可删）
│   │   ├── logs        （日志文件，可删）
│   │   ├── checkpoints （状态文件，可删）
│   │   └── savepoints  （状态文件，可删）
│   └── output          （建议的 Flink 输出根目录，内容可删）
└── docker-compose.yml
```

## 可观察性

### Flink Web UI 

在启动之后，可以通过 http://localhost:8081 进行访问

### Cluster Manager for Apache Kafka

Kafka 的消息状态可以打开 http://localhost:9000/ 通过 [CMAK](https://github.com/yahoo/CMAK) 进行观察，但需要添加以下配置，每次重启之后需要重新设置

*   打开 http://localhost:9000/
*   点击 Cluster 右侧的下拉按钮，选择 "Add Cluster" 
*   Cluster Name 随意，但必须输入
*   Cluster Zookeeper Hosts 填写 localhost:2181
*   勾选 "Enable JMX Polling (Set JMX_PORT env variable before starting kafka server)"
*   勾选 "Poll consumer information (Not recommended for large # of consumers if ZK is used for offsets tracking on older Kafka versions)"
*   点击 Save 即可开始观察集群的状态

## 关闭

```
docker-compose down
```

关闭之后 volume 的文件并不会被清除，可以手动清理。

## 参考文献

[Flink Operations Playground](https://github.com/apache/flink-playgrounds/tree/master/operations-playground)

[Confluentinc cp-all-in-one-community](https://github.com/confluentinc/cp-all-in-one/tree/6.2.1-post/cp-all-in-one-community)
