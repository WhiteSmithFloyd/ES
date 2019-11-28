# Elastic Search Installation

### 安装ES实例

1. Download ES

   <https://www.elastic.co/cn/downloads/elasticsearch>

2. Run  `elasticSearch`

   ```shell
   bin/elasticsearch 
   ```

3.  Access `http://localhost:9200/` to check the ES status, than get info below 

   ```json
   {
     "name" : "h3",
     "cluster_name" : "elasticsearch",
     "cluster_uuid" : "UwzKM-KpSYGWWC7eKPKt0g",
     "version" : {
       "number" : "7.4.2",
       "build_flavor" : "default",
       "build_type" : "tar",
       "build_hash" : "2f90bbf7b93631e52bafb59b3b049cb44ec25e96",
       "build_date" : "2019-10-28T20:40:44.881551Z",
       "build_snapshot" : false,
       "lucene_version" : "8.2.0",
       "minimum_wire_compatibility_version" : "6.8.0",
       "minimum_index_compatibility_version" : "6.0.0-beta1"
     },
     "tagline" : "You Know, for Search"
   }
   ```



### 插件安装

```shell
# 查看已经安装的插件
bin/elasticsearch-plugin list

# 安装插件
bin/elasticsearch-plugin install analysis-icu
```

通过  `http://h2:9200/_cat/plugins?v` 可以在浏览器上查询插件



###  集群（多实例 ）

方法1. 直接运行集群 (本机)： 指定节点名称、集群名称、数据存储路径 

```shell
# h2
bin/elasticsearch -E node.name=node3 -E cluster.name=esCluster1 -E path.data=/extDisk/home/data/ES/node3_data -d
# h3
bin/elasticsearch -E node.name=node2 -E cluster.name=esCluster1 -E path.data=/extDisk/home/data/Es/node2_data -d
# h4
bin/elasticsearch -E node.name=node1 -E cluster.name=esCluster1 -E path.data=/extDisk/home/data/Es/node1_data -d
```



**方法2. 修改集群配置文件 (推荐)**

> 有三台主机：h2, h3, h4

1. 修改h2的配置文件 `elasticsearch.yml`

   ```yaml
   # ======================== Elasticsearch Configuration =========================
   #
   # NOTE: Elasticsearch comes with reasonable defaults for most settings.
   #       Before you set out to tweak and tune the configuration, make sure you
   #       understand what are you trying to accomplish and the consequences.
   #
   # The primary way of configuring a node is via this file. This template lists
   # the most important settings you may want to configure for a production cluster.
   #
   # Please consult the documentation for further information on configuration options:
   # https://www.elastic.co/guide/en/elasticsearch/reference/index.html
   #
   # ---------------------------------- Cluster -----------------------------------
   #
   # Use a descriptive name for your cluster:
   #
   #cluster.name: my-application
   cluster.name: esCluster1
   #
   # ------------------------------------ Node ------------------------------------
   #
   # Use a descriptive name for the node:
   #
   #node.name: node-1
   node.name: node2
   #
   # Add custom attributes to the node:
   #
   #node.attr.rack: r1
   #
   # ----------------------------------- Paths ------------------------------------
   #
   # Path to directory where to store the data (separate multiple locations by comma):
   #
   #path.data: /path/to/data
   path.data: /extDisk/home/data/ES/node2_data
   #
   # Path to log files:
   #
   #path.logs: /path/to/logs
   path.logs: /extDisk/home/data/ES/node2_log
   #
   # ----------------------------------- Memory -----------------------------------
   #
   # Lock the memory on startup:
   #
   #bootstrap.memory_lock: true
   #
   # Make sure that the heap size is set to about half the memory available
   # on the system and that the owner of the process is allowed to use this
   # limit.
   #
   # Elasticsearch performs poorly when the system is swapping the memory.
   #
   # ---------------------------------- Network -----------------------------------
   #
   # Set the bind address to a specific IP (IPv4 or IPv6):
   #
   # network.host: 192.168.0.1
   network.host: 10.16.13.186
   #
   # Set a custom port for HTTP:
   #
   #http.port: 9200
   #
   # For more information, consult the network module documentation.
   #
   # --------------------------------- Discovery ----------------------------------
   #
   # Pass an initial list of hosts to perform discovery when this node is started:
   # The default list of hosts is ["127.0.0.1", "[::1]"]
   #
   #discovery.seed_hosts: ["host1", "host2"]
   discovery.seed_hosts: ["h4","h3","h2"]
   #
   # Bootstrap the cluster using an initial set of master-eligible nodes:
   #
   #cluster.initial_master_nodes: ["node-1", "node-2"]
   cluster.initial_master_nodes: ["node1","node2","node3"]
   #
   # For more information, consult the discovery and cluster formation module documentation.
   #
   # ---------------------------------- Gateway -----------------------------------
   #
   # Block initial recovery after a full cluster restart until N nodes are started:
   #
   #gateway.recover_after_nodes: 3
   #
   # For more information, consult the gateway module documentation.
   #
   # ---------------------------------- Various -----------------------------------
   #
   # Require explicit names when deleting indices:
   #
   #action.destructive_requires_name: true
   ```

2. 分别修改 `h3`和`h4`上`ES`的配置文件

   ```yaml
   # 仅修改如下面信息，其他不变
   network.host: # IP of h2 or h3
   node.name: # node name
   ```

3. 启动 ES 集群

   ```shell
   nohup bin/elasticsearch -d 
   ```



#### 集群状态查询

```shell
# 前缀 http://h4:9200/  CASE:  http://h4:9200/_cat/nodes?v
/_cat/allocation

/_cat/shards

/_cat/shards/{index}

/_cat/master

/_cat/nodes

/_cat/tasks

/_cat/indices

/_cat/indices/{index}

/_cat/segments

/_cat/segments/{index}

/_cat/count

/_cat/count/{index}

/_cat/recovery

/_cat/recovery/{index}

/_cat/health

/_cat/pending_tasks

/_cat/aliases

/_cat/aliases/{alias}

/_cat/thread_pool

/_cat/thread_pool/{thread_pools}

/_cat/plugins

/_cat/fielddata

/_cat/fielddata/{fields}

/_cat/nodeattrs

/_cat/repositories

/_cat/snapshots/{repository}

/_cat/templates
```





### Q&A

ERROR: [5] bootstrap checks failed

#### [1]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65535]

**最大文件描述符**

切换root用户 `vi /etc/security/limits.conf` 

```conf
# 在倒数第二行
* soft nofile 65536
* hard nofile 65536
```



#### [2]: max number of threads [1024] for user [es] is too low, increase to at least [4096]

**最大线程数**

切换root用户 `vi /etc/security/limits.conf` 

```conf
# 在倒数第二行
* soft    nproc   2048
* hard    nproc   4096
```



#### [3]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]

最大虚拟内存

切换root用户 `vi /etc/sysctl.conf`

```conf
vm.max_map_count=262144
```



#### [4]: failed to install; check the logs and fix your configuration or disable system call filters at your own risk

**系统调用过滤器**

修改`elasticsearch.yml` 

```yaml
bootstrap.memory_lock: false
bootstrap.system_call_filter: false
```



#### [5]: the default discovery settings are unsuitable for production use; at least one of [discovery.seed_hosts, discovery.seed_providers, cluster.initial_master_nodes] must be configured

修改 `elasticsearch.yml`

```yaml
# 取消注释保留一个节点
cluster.initial_master_nodes: ["node-1"]
```



