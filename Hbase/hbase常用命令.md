# HBase常用的命令

### HBase快照

**对某张表生成快照：**

 ```shell
> snapshot 'namespace:tableName', 'snapshotName'
 ```

**列出所有快照：**

   ```shell
> list_snapshots
   ```

**删除指定快照：**

 ```shell
> delete_snapshot 'snapshotName'
 ```

**从指定快照生成新表:**

 ```shell
> clone_snapshot 'snapshotName', 'namespace:newTableName'
 ```

**将指定快照内容替换生成快照的表的结构/数据，需要先disable当前表：**

```shell
> disable 'tableName'
> restore_snapshot 'snapshotName'
> enable 'tableName'
```

**使用ExportSnapshot工具类将现有快照导出至其他集群:**

 ```shell
$ hbase org.apache.hadoop.hbase.snapshot.ExportSnapshot -snapshot <snapshotName> -copy-to <hdfs:///ip_address:port/hbase>
 ```



