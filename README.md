# ClickHouse集群一键安装

基于ansible快速安装ClickHouse相关组件

**项目地址：**

- Github: https://github.com/tangyibo/clickhouse_installer

- Gitee: https://gitee.com/inrgihc/clickhouse_installer


## 一、安装准备

- (1)  配置hosts.ini配置文件

安装文件格式配置主机IP及密码等，各个组件安装的节点等，示例如下：

```
[nodes]
172.17.203.92 hostname=node1 ansible_ssh_user=root ansible_ssh_port=22 ansible_ssh_pass='123321'
172.17.203.93 hostname=node2 ansible_ssh_user=root ansible_ssh_port=22 ansible_ssh_pass='123321'
172.17.203.94 hostname=node3 ansible_ssh_user=root ansible_ssh_port=22 ansible_ssh_pass='123321'

[zookeeper]
node1
node2
node3

[clickhouse]
node1
node2
node3
```

## 二、一键安装

### 1、安装

```
cd clickhouse_installer/
sh install.sh
```

### 2、卸载

```
cd clickhouse_installer/
sh uninstall.sh
```

## 三、分步安装

### 1、安装

- (1) 部署机安装ansible：

使用yum安装ansible:

```
# yum install -y epel-release
# yum install -y ansible
```

- (2) root免密及集群主机hosts配置

执行如下命令所有集群服务器/etc/hosts及root免密配置:

```
# ansible-playbook -i hosts.ini prepare.yaml
```

- (3) 安装Zookeeper

 查看 vars/var_zookeeper.yml的参数配置，尤其根据版本号检查下载地址是否可用，然后执行shell命令安装zookeep集群:

```
# ansible-playbook -i hosts.ini  zookeeper.yaml
```

- (4) 安装ClickHouse集群

查看 vars/var_clickhouse.yml的参数配置，尤其是确认数据安装目录，然后执行shell命令安装ClickHouse集群:

```
# ansible-playbook -i hosts.ini  clickhouse.yaml
```

### 2、卸载

- (1) 执行如下命令卸载：

```
# ansible-playbook -i hosts.ini cleanup.yaml
```

## 四、集群验证

登录clickhouse集群的任意一台机器节点，执行如下命令：

```
clickhouse-client -h 127.0.0.1
```

然后执行如下SQL语句查看集群节点配置信息：

```
select * from system.clusters;
```

我的信息如下：

```
[root@node1 ~]# clickhouse-client -h 127.0.0.1
ClickHouse client version 21.1.2.15 (official build).
Connecting to 127.0.0.1:9000 as user default.
Connected to ClickHouse server version 21.1.2 revision 54443.

node1 :) select * from system.clusters;

SELECT *
FROM system.clusters

Query id: 4cdcd3d4-855b-4774-8669-e873bd33bf73

┌─cluster───────────────────────────┬─shard_num─┬─shard_weight─┬─replica_num─┬─host_name─┬─host_address──┬─port─┬─is_local─┬─user────┬─default_database─┬─errors_count─┬─estimated_recovery_time─┐
│ perftest_3shards_1replicas        │         1 │            1 │           1 │ node1     │ 172.17.203.92 │ 9000 │        1 │ default │                  │            0 │                       0 │
│ perftest_3shards_1replicas        │         2 │            1 │           1 │ node2     │ 172.17.203.93 │ 9000 │        0 │ default │                  │            0 │                       0 │
│ perftest_3shards_1replicas        │         3 │            1 │           1 │ node3     │ 172.17.203.94 │ 9000 │        0 │ default │                  │            0 │                       0 │
│ test_cluster_two_shards           │         1 │            1 │           1 │ 127.0.0.1 │ 127.0.0.1     │ 9000 │        1 │ default │                  │            0 │                       0 │
│ test_cluster_two_shards           │         2 │            1 │           1 │ 127.0.0.2 │ 127.0.0.2     │ 9000 │        0 │ default │                  │            0 │                       0 │
│ test_cluster_two_shards_localhost │         1 │            1 │           1 │ localhost │ ::1           │ 9000 │        1 │ default │                  │            0 │                       0 │
│ test_cluster_two_shards_localhost │         2 │            1 │           1 │ localhost │ ::1           │ 9000 │        1 │ default │                  │            0 │                       0 │
│ test_shard_localhost              │         1 │            1 │           1 │ localhost │ ::1           │ 9000 │        1 │ default │                  │            0 │                       0 │
│ test_shard_localhost_secure       │         1 │            1 │           1 │ localhost │ ::1           │ 9440 │        0 │ default │                  │            0 │                       0 │
│ test_unavailable_shard            │         1 │            1 │           1 │ localhost │ ::1           │ 9000 │        1 │ default │                  │            0 │                       0 │
│ test_unavailable_shard            │         2 │            1 │           1 │ localhost │ ::1           │    1 │        0 │ default │                  │            0 │                       0 │
└───────────────────────────────────┴───────────┴──────────────┴─────────────┴───────────┴───────────────┴──────┴──────────┴─────────┴──────────────────┴──────────────┴─────────────────────────┘

11 rows in set. Elapsed: 0.008 sec.

```