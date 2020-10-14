# Hadoop-
经验总结

## 大数据数据仓库实战项目-电商数仓教程V3.0最新版

### 1. 项目经验之 HDFS 存储多目录
若 HDFS 存储空间紧张，需要对 DataNode 进行磁盘扩展
  1. 在 DataNode 节点增加磁盘并进行挂载。 

  2. 在 hdfs-site.xml 文件中配置多目录，注意新挂载磁盘的访问权限问题。 
```
<property>
  <name>dfs.datanode.data.dir</name>
  <value>file:///${hadoop.tmp.dir}/dfs/data1,file:///hd2/dfs/ data2,file:///hd3/dfs/data3,file:///hd4/dfs/data4</value>
</property>
```


### 2. 项目经验之 集群数据均衡
#### 1. 增加磁盘后，保证集群每个机器间数据均衡 
1. 开启数据均衡命令：
```
bin/start-balancer.sh –threshold 10 
```
对于参数 10，代表的是集群中各个节点的磁盘空间利用率相差不超过 10%，可根据实际情况进行调整。 

2. 停止数据均衡命令：
```
bin/stop-balancer.sh
```
 
#### 2. 集群机器每个磁盘间数据均衡
1. 生成均衡计划
```
hdfs diskbalancer -plan hadoop103
````

2. 执行均衡计划
```
hdfs diskbalancer -execute hadoop103.plan.json
```

3. 查看当前均衡计划的执行情况
```
hdfs diskbalancer -query hadoop103
```

4. 取消均衡计划
```
hdfs diskbalancer -cancel hadoop103.plan.json
```


### 3. 项目经验之 支持LZO压缩配置
将编译好后的 hadoop-lzo-0.4.20.jar 放入 hadoop-3.1.2/share/hadoop/common/

修改core-site.xml 增加配置支持 LZO 压缩
```
<property>
  <name>io.compression.codecs</name>
  <value> org.apache.hadoop.io.compress.GzipCodec, org.apache.hadoop.io.compress.DefaultCodec, org.apache.hadoop.io.compress.BZip2Codec, org.apache.hadoop.io.compress.SnappyCodec, com.hadoop.compression.lzo.LzoCodec, com.hadoop.compression.lzo.LzopCodec </value>
</property>
```


### 4. 项目经验之 LZO 创建索引
1. 创建 LZO 文件的索引，LZO 压缩文件的可切片特性依赖于其索引，故我们需要手动为 LZO 压缩文件创建索引。若无索引，则 LZO 文件的切片只有一个
2. 测试
   1. 将 bigtable.lzo（150M）上传到集群的根目录
    ```
    hadoop fs -put bigtable.lzo /input
    ```
   2. 执行 wordcount 程序
    ```
    hadoop jar /opt/module/hadoop-3.1.3/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar wordcount -Dmapreduce.job.inputformat.class=com.hadoop.mapreduce.LzoTextInputFormat /input /output1
    ```
    ![lzo_split_1](https://github.com/caocong192/Hadoop-/blob/main/pics/lzo_split_1.jpg)
    
   3. 对上传的 LZO 文件建索引
    ```
    hadoop jar /opt/module/hadoop-3.1.3/share/hadoop/common/hadoop-lzo-0.4.20.jar com.hadoop.compression.lzo.DistributedLzoIndexer /input/bigtable.lzo
    ```
   4. 再次执行 WordCount 程序
    ```
    hadoop jar /opt/module/hadoop-3.1.3/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar wordcount -Dmapreduce.job.inputformat.class=com.hadoop.mapreduce.LzoTextInputFormat /input /output2
    ```


### 4. 项目经验之 基准测试


