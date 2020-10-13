# Hadoop-
经验总结

## 大数据数据仓库实战项目-电商数仓教程V3.0最新版

### 1. 项目经验之 HDFS 存储多目录
若 HDFS 存储空间紧张，需要对 DataNode 进行磁盘扩展
1. 在 DataNode 节点增加磁盘并进行挂载。 

2. 在 hdfs-site.xml 文件中配置多目录，注意新挂载磁盘的访问权限问题。 
`<property>
  <name>dfs.datanode.data.dir</name>
  <value>file:///${hadoop.tmp.dir}/dfs/data1,file:///hd2/dfs/ data2,file:///hd3/dfs/data3,file:///hd4/dfs/data4</value>
</property>`

3. 增加磁盘后，保证每个目录数据均衡 
开启数据均衡命令：
`bin/start-balancer.sh –threshold 10 `
对于参数 10，代表的是集群中各个节点的磁盘空间利用率相差不超过 10%，可根据实际情况进行调整。 

停止数据均衡命令：
`bin/stop-balancer.sh`
