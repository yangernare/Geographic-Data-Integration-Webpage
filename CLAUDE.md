# 大数据软件安装和基础编程实践指南

> 本指南基于厦门大学数据库实验室林子雨老师的系列教程整理，配套《大数据技术原理与应用》教材。
> 教程来源：https://dblab.xmu.edu.cn/blog/

---

## 目录

1. [环境准备：安装 VirtualBox 和 Ubuntu 虚拟机](#1-环境准备安装-virtualbox-和-ubuntu-虚拟机)
2. [Windows 与 Ubuntu 之间传输文件（FTP）](#2-windows-与-ubuntu-之间传输文件ftp)
3. [Hadoop 3.1.3 安装教程（单机/伪分布式）](#3-hadoop-313-安装教程单机伪分布式)
4. [HDFS 编程实践（Hadoop 3.1.3）](#4-hdfs-编程实践hadoop-313)
5. [HBase 2.2.2 安装和编程实践](#5-hbase-222-安装和编程实践)
6. [MapReduce 编程实践（Hadoop 3.3.5）](#6-mapreduce-编程实践hadoop-335)
7. [2023 版完整学习路线](#7-2023-版完整学习路线)
8. [Hive 3.1.3 安装和使用](#8-hive-313-安装和使用)
9. [Spark 3.4.0 安装和编程实践](#9-spark-340-安装和编程实践)
10. [Flink 1.16.2 安装与编程实践](#10-flink-1162-安装与编程实践)
11. [统一启动和关闭指南](#11-统一启动和关闭指南)

---

## 1. 环境准备：安装 VirtualBox 和 Ubuntu 虚拟机

### 1.1 前置条件

- 确保电脑已开启 CPU 虚拟化（进入 BIOS，将 Virtualization 设置为 Enabled）
- 关闭杀毒软件或添加信任
- 推荐电脑总内存 8GB 以上

### 1.2 安装 VirtualBox

1. 下载 VirtualBox（官网或百度网盘，提取码：lnwl）
2. 双击安装包，按向导完成安装
3. 安装过程中弹出的网络重置警告需确认

### 1.3 创建 Ubuntu 虚拟机

1. 打开 VirtualBox，点击"新建"
2. 设置名称（如 "Ubuntu"），文件夹设为本地磁盘目录（如 `D:\`），类型选 "Linux"，版本选 "Ubuntu (64 bit)"
3. 设置虚拟机内存：4GB 总内存分 1GB，8GB 总内存分 3GB
4. 创建虚拟硬盘：选 VDI，动态分配，大小 **30GB**（大数据软件需要较大空间）
5. 启动前先设置"存储"：加载 Ubuntu ISO 镜像文件（如 `ubuntukylin-16.04-desktop-amd64.iso` 或 `ubuntu-22.04`）
6. 启动虚拟机，按提示安装 Ubuntu

### 1.4 安装 Ubuntu 注意事项

- 安装类型选择"其他选项"进行手动分区
- 分区方案：
  - **交换空间**：512MB，主分区，空间起始位置
  - **根目录**：剩余全部空间，逻辑分区，EXT4日志文件系统，挂载点 `/`
- 用户名建议创建 `hadoop` 用户
- 安装完成后重启，安装增强功能以支持更高分辨率：

```bash
sudo apt-get install virtualbox-guest-dkms
```

### 1.5 Win11 + VirtualBox 7.2.8 + Ubuntu 22.04.5 快速流程

1. 下载 VirtualBox 7.2.8 和 Ubuntu 22.04.5 ISO（华为云镜像）
2. VirtualBox 中创建虚拟机，选择 ISO 映像，点击"自动安装"
3. 配置用户名密码，硬件和虚拟硬盘默认即可
4. 等待自动安装完成，重启进入系统

### 1.6 网络配置

- 安装完成后如无法联网，切换网络模式到**桥接模式**
- VirtualBox 设置 → 网络 → 连接方式选择"桥接网卡"

---

## 2. Windows 与 Ubuntu 之间传输文件（FTP）

### 2.1 设置网络连接方式

默认 NAT 模式下主机无法访问虚拟机，需改为**桥接网卡**：

1. 关闭虚拟机和 VirtualBox
2. 重新打开 VirtualBox → 设置 → 网络 → 连接方式选择"桥接网卡"
3. 在"界面名称"中选择当前连接互联网的网卡
4. 启动虚拟机

### 2.2 使用 FileZilla 进行文件传输

1. 在 Windows 上安装 FileZilla
2. 在 Ubuntu 终端中查看 IP 地址：
```bash
ifconfig
# 找到 inet 地址，如 192.168.0.104
```
3. 打开 FileZilla → 文件 → 站点管理器 → 新站点
4. 配置连接参数：
   - **主机**：Ubuntu 的 IP 地址
   - **协议**：SFTP - SSH File Transfer Protocol
   - **登录类型**：正常
   - **用户名/密码**：Ubuntu 的用户凭据
5. 连接后，左侧为 Windows 本地文件，右侧为 Ubuntu 远程文件
6. 右键文件选择"上传"或"下载"

> **注意**：Linux 文件权限限制，只能上传到用户有权限的目录（如 `/home/hadoop`）

---

## 3. Hadoop 3.1.3 安装教程（单机/伪分布式）

### 3.1 创建 hadoop 用户

```bash
sudo useradd -m hadoop -s /bin/bash
sudo passwd hadoop        # 设置密码，可设为 hadoop
sudo adduser hadoop sudo  # 添加管理员权限
```

注销当前用户，用 hadoop 用户登录。

### 3.2 更新 apt 并安装 vim

```bash
sudo apt-get update
sudo apt-get install vim
```

### 3.3 安装 SSH 并配置无密码登录

```bash
sudo apt-get install openssh-server
ssh localhost              # 首次连接输入 yes 和密码

# 配置无密码登录
exit
cd ~/.ssh/
ssh-keygen -t rsa         # 一路回车
cat ./id_rsa.pub >> ./authorized_keys
ssh localhost              # 验证无需密码
```

### 3.4 安装 Java 环境（JDK 1.8）

```bash
cd /usr/lib
sudo mkdir jvm
cd ~/Downloads
sudo tar -zxvf ./jdk-8u162-linux-x64.tar.gz -C /usr/lib/jvm
```

配置环境变量，编辑 `~/.bashrc`，添加：

```bash
export JAVA_HOME=/usr/lib/jvm/jdk1.8.0_162
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```

```bash
source ~/.bashrc
java -version   # 验证安装
```

### 3.5 安装 Hadoop 3.1.3

```bash
sudo tar -zxf ~/下载/hadoop-3.1.3.tar.gz -C /usr/local
cd /usr/local/
sudo mv ./hadoop-3.1.3/ ./hadoop
sudo chown -R hadoop ./hadoop
cd /usr/local/hadoop
./bin/hadoop version       # 验证安装
```

### 3.6 Hadoop 单机模式（非分布式）

```bash
cd /usr/local/hadoop
mkdir ./input
cp ./etc/hadoop/*.xml ./input
./bin/hadoop jar ./share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar grep ./input ./output 'dfs[a-z.]+'
cat ./output/*
rm -r ./output   # 清理
```

### 3.7 Hadoop 伪分布式配置

**编辑 `./etc/hadoop/core-site.xml`：**

```xml
<configuration>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>file:/usr/local/hadoop/tmp</value>
        <description>Abase for other temporary directories.</description>
    </property>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>
```

**编辑 `./etc/hadoop/hdfs-site.xml`：**

```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:/usr/local/hadoop/tmp/dfs/name</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:/usr/local/hadoop/tmp/dfs/data</value>
    </property>
</configuration>
```

**格式化 NameNode 并启动：**

```bash
cd /usr/local/hadoop
./bin/hdfs namenode -format    # 成功会显示 "successfully formatted"
./sbin/start-dfs.sh            # 启动
jps                            # 验证：应看到 NameNode, DataNode, SecondaryNameNode
```

**Web 界面**：访问 http://localhost:9870 查看 NameNode 和 DataNode 信息。

### 3.8 运行伪分布式实例

```bash
cd /usr/local/hadoop
./bin/hdfs dfs -mkdir -p /user/hadoop
./bin/hdfs dfs -mkdir input
./bin/hdfs dfs -put ./etc/hadoop/*.xml input
./bin/hadoop jar ./share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar grep input output 'dfs[a-z.]+'
./bin/hdfs dfs -cat output/*
./bin/hdfs dfs -get output ./output  # 取回本地
cat ./output/*
```

**停止 Hadoop：**
```bash
./sbin/stop-dfs.sh
```

> **注意**：下次启动无需重新格式化 NameNode，直接运行 `./sbin/start-dfs.sh`

### 3.9 常见问题

- **JAVA_HOME not set**：在 `/usr/local/hadoop/etc/hadoop/hadoop-env.sh` 中修改 `export JAVA_HOME=/usr/lib/jvm/jdk1.8.0_162`
- **Could not resolve hostname**：在 `~/.bashrc` 中添加：
  ```bash
  export HADOOP_HOME=/usr/local/hadoop
  export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
  ```
- **DataNode 无法启动**：删除 tmp 重新格式化
  ```bash
  ./sbin/stop-dfs.sh
  rm -r ./tmp
  ./bin/hdfs namenode -format
  ./sbin/start-dfs.sh
  ```

---

## 4. HDFS 编程实践（Hadoop 3.1.3）

### 4.1 启动 Hadoop

```bash
cd /usr/local/hadoop
./sbin/start-dfs.sh
```

### 4.2 Shell 命令与 HDFS 交互

三种 Shell 命令方式：
- `hadoop fs` — 适用于任何文件系统
- `hadoop dfs` — 仅适用于 HDFS
- `hdfs dfs` — 仅适用于 HDFS

**目录操作：**

```bash
cd /usr/local/hadoop
./bin/hdfs dfs -mkdir -p /user/hadoop    # 创建用户目录
./bin/hdfs dfs -ls .                      # 列出当前用户目录
./bin/hdfs dfs -mkdir input               # 创建 input 目录（相对路径）
./bin/hdfs dfs -mkdir /input              # 在根目录创建 input
./bin/hdfs dfs -rm -r /input              # 删除目录
```

**文件操作：**

```bash
# 上传文件
./bin/hdfs dfs -put /home/hadoop/myLocalFile.txt input

# 查看文件列表
./bin/hdfs dfs -ls input

# 查看文件内容
./bin/hdfs dfs -cat input/myLocalFile.txt

# 下载文件到本地
./bin/hdfs dfs -get input/myLocalFile.txt /home/hadoop/下载

# HDFS 内部拷贝
./bin/hdfs dfs -cp input/myLocalFile.txt /input
```

### 4.3 Web 界面管理 HDFS

访问 http://localhost:9870

### 4.4 Java API 与 HDFS 交互

**在 Eclipse 中创建项目 "HDFSExample"，添加 JAR 包：**

- `/usr/local/hadoop/share/hadoop/common` 下的 JAR 包（不含 jdiff、lib、sources、webapps 目录）
- `/usr/local/hadoop/share/hadoop/common/lib` 下所有 JAR 包
- `/usr/local/hadoop/share/hadoop/hdfs` 下的 JAR 包（不含 jdiff、lib、sources、webapps 目录）
- `/usr/local/hadoop/share/hadoop/hdfs/lib` 下所有 JAR 包

**示例代码 — 文件合并（MergeFile.java）：**

```java
import java.io.IOException;
import java.io.PrintStream;
import java.net.URI;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.*;

class MyPathFilter implements PathFilter {
    String reg = null;
    MyPathFilter(String reg) { this.reg = reg; }
    public boolean accept(Path path) {
        if (!(path.toString().matches(reg))) return true;
        return false;
    }
}

public class MergeFile {
    Path inputPath = null;
    Path outputPath = null;
    public MergeFile(String input, String output) {
        this.inputPath = new Path(input);
        this.outputPath = new Path(output);
    }
    public void doMerge() throws IOException {
        Configuration conf = new Configuration();
        conf.set("fs.defaultFS","hdfs://localhost:9000");
        conf.set("fs.hdfs.impl","org.apache.hadoop.hdfs.DistributedFileSystem");
        FileSystem fsSource = FileSystem.get(URI.create(inputPath.toString()), conf);
        FileSystem fsDst = FileSystem.get(URI.create(outputPath.toString()), conf);
        FileStatus[] sourceStatus = fsSource.listStatus(inputPath, new MyPathFilter(".*\\.abc"));
        FSDataOutputStream fsdos = fsDst.create(outputPath);
        PrintStream ps = new PrintStream(System.out);
        for (FileStatus sta : sourceStatus) {
            System.out.print("路径：" + sta.getPath() + " 文件大小：" + sta.getLen()
                + " 权限：" + sta.getPermission() + " 内容：");
            FSDataInputStream fsdis = fsSource.open(sta.getPath());
            byte[] data = new byte[1024];
            int read = -1;
            while ((read = fsdis.read(data)) > 0) {
                ps.write(data, 0, read);
                fsdos.write(data, 0, read);
            }
            fsdis.close();
        }
        ps.close();
        fsdos.close();
    }
    public static void main(String[] args) throws IOException {
        MergeFile merge = new MergeFile(
            "hdfs://localhost:9000/user/hadoop/",
            "hdfs://localhost:9000/user/hadoop/merge.txt");
        merge.doMerge();
    }
}
```

**其他常用 API 示例：**

写入文件：
```java
Configuration conf = new Configuration();
conf.set("fs.defaultFS","hdfs://localhost:9000");
conf.set("fs.hdfs.impl","org.apache.hadoop.hdfs.DistributedFileSystem");
FileSystem fs = FileSystem.get(conf);
byte[] buff = "Hello world".getBytes();
FSDataOutputStream os = fs.create(new Path("test"));
os.write(buff, 0, buff.length);
os.close();
fs.close();
```

判断文件是否存在：
```java
FileSystem fs = FileSystem.get(conf);
if(fs.exists(new Path("test"))){
    System.out.println("文件存在");
}else{
    System.out.println("文件不存在");
}
```

读取文件：
```java
FileSystem fs = FileSystem.get(conf);
Path file = new Path("test");
FSDataInputStream getIt = fs.open(file);
BufferedReader d = new BufferedReader(new InputStreamReader(getIt));
String content = d.readLine();
System.out.println(content);
d.close();
fs.close();
```

### 4.5 打包部署 JAR

```bash
cd /usr/local/hadoop
mkdir myapp
# 在 Eclipse 中：右键工程 → Export → Runnable JAR file
# Launch configuration: 选择主类
# Export destination: /usr/local/hadoop/myapp/HDFSExample.jar
# Library handling: Extract required libraries into generated JAR

# 运行
./bin/hadoop jar ./myapp/HDFSExample.jar
```

---

## 5. HBase 2.2.2 安装和编程实践

### 5.1 前置条件

- 已安装 Hadoop 3.1.3（伪分布式模式）
- 已安装 JDK 1.8 和 SSH

### 5.2 安装 HBase

```bash
cd ~
sudo tar -zxf ~/下载/hbase-2.2.2-bin.tar.gz -C /usr/local
cd /usr/local
sudo mv ./hbase-2.2.2 ./hbase
sudo chown -R hadoop ./hbase
```

**配置环境变量**，编辑 `~/.bashrc`，添加：

```bash
export PATH=$PATH:/usr/local/hbase/bin
```

```bash
source ~/.bashrc
/usr/local/hbase/bin/hbase version   # 验证安装
```

### 5.3 单机模式配置

**编辑 `/usr/local/hbase/conf/hbase-env.sh`：**

```bash
export JAVA_HOME=/usr/lib/jvm/jdk1.8.0_162
export HBASE_MANAGES_ZK=true
```

**编辑 `/usr/local/hbase/conf/hbase-site.xml`：**

```xml
<configuration>
    <property>
        <name>hbase.rootdir</name>
        <value>file:///usr/local/hbase/hbase-tmp</value>
    </property>
</configuration>
```

**启动测试：**

```bash
cd /usr/local/hbase
bin/start-hbase.sh
bin/hbase shell         # 进入 Shell 交互界面
# 输入 exit 退出 shell
bin/stop-hbase.sh       # 停止 HBase
```

### 5.4 伪分布式模式配置

**编辑 `/usr/local/hbase/conf/hbase-env.sh`：**

```bash
export JAVA_HOME=/usr/lib/jvm/jdk1.8.0_162
export HBASE_CLASSPATH=/usr/local/hbase/conf
export HBASE_MANAGES_ZK=true
```

**编辑 `/usr/local/hbase/conf/hbase-site.xml`：**

```xml
<configuration>
    <property>
        <name>hbase.rootdir</name>
        <value>hdfs://localhost:9000/hbase</value>
    </property>
    <property>
        <name>hbase.cluster.distributed</name>
        <value>true</value>
    </property>
    <property>
        <name>hbase.unsafe.stream.capability.enforce</name>
        <value>false</value>
    </property>
</configuration>
```

> **重要**：`hbase.unsafe.stream.capability.enforce` 必须设为 false，否则 HMaster 进程无法启动

**启动顺序：**

```bash
# 1. 先启动 Hadoop
ssh localhost
cd /usr/local/hadoop
./sbin/start-dfs.sh

# 2. 再启动 HBase
cd /usr/local/hbase
bin/start-hbase.sh
jps    # 应看到 HMaster, HRegionServer 等进程

# 3. 进入 Shell
bin/hbase shell
```

**关闭顺序（重要！）：**

```
启动 Hadoop → 启动 HBase → 关闭 HBase → 关闭 Hadoop
```

```bash
bin/stop-hbase.sh       # 关闭 HBase
cd /usr/local/hadoop
./sbin/stop-dfs.sh      # 关闭 Hadoop
```

### 5.5 HBase Shell 操作

**创建表：**
```bash
create 'student','Sname','Ssex','Sage','Sdept','course'
describe 'student'    # 查看表信息
```

**添加数据：**
```bash
put 'student','95001','Sname','LiYing'
put 'student','95001','course:math','80'
put 'student','95001','Ssex','Male'
```

**查看数据：**
```bash
get 'student','95001'       # 查看某一行
scan 'student'              # 查看全部数据
```

**删除数据：**
```bash
delete 'student','95001','Ssex'    # 删除一个数据
deleteall 'student','95001'        # 删除一行数据
```

**删除表：**
```bash
disable 'student'
drop 'student'
```

**查询历史版本：**
```bash
create 'teacher',{NAME=>'username',VERSIONS=>5}
put 'teacher','91001','username','Mary'
put 'teacher','91001','username','Mary1'
get 'teacher','91001',{COLUMN=>'username',VERSIONS=>5}
```

### 5.6 HBase Java API 编程

在 Eclipse 中创建项目 "HBaseExample"，添加 `/usr/local/hbase/lib` 下所有 JAR 包（不含 client-facing-thirdparty、ruby、shaded-clients、zkcli 目录），以及 `client-facing-thirdparty` 目录下的所有 JAR 包。

**示例代码（ExampleForHBase.java）：**

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.*;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.util.Bytes;
import java.io.IOException;

public class ExampleForHBase {
    public static Configuration configuration;
    public static Connection connection;
    public static Admin admin;

    public static void main(String[] args) throws IOException {
        init();
        createTable("student", new String[]{"score"});
        insertData("student", "zhangsan", "score", "English", "69");
        insertData("student", "zhangsan", "score", "Math", "86");
        insertData("student", "zhangsan", "score", "Computer", "77");
        getData("student", "zhangsan", "score", "English");
        close();
    }

    public static void init() {
        configuration = HBaseConfiguration.create();
        configuration.set("hbase.rootdir", "hdfs://localhost:9000/hbase");
        try {
            connection = ConnectionFactory.createConnection(configuration);
            admin = connection.getAdmin();
        } catch (IOException e) { e.printStackTrace(); }
    }

    public static void close() {
        try {
            if (admin != null) admin.close();
            if (null != connection) connection.close();
        } catch (IOException e) { e.printStackTrace(); }
    }

    public static void createTable(String myTableName, String[] colFamily) throws IOException {
        TableName tableName = TableName.valueOf(myTableName);
        if (admin.tableExists(tableName)) {
            System.println("table is exists!");
        } else {
            TableDescriptorBuilder tableDescriptor = TableDescriptorBuilder.newBuilder(tableName);
            for (String str : colFamily) {
                ColumnFamilyDescriptor family =
                    ColumnFamilyDescriptorBuilder.newBuilder(Bytes.toBytes(str)).build();
                tableDescriptor.setColumnFamily(family);
            }
            admin.createTable(tableDescriptor.build());
        }
    }

    public static void insertData(String tableName, String rowKey, String colFamily,
                                   String col, String val) throws IOException {
        Table table = connection.getTable(TableName.valueOf(tableName));
        Put put = new Put(rowKey.getBytes());
        put.addColumn(colFamily.getBytes(), col.getBytes(), val.getBytes());
        table.put(put);
        table.close();
    }

    public static void getData(String tableName, String rowKey,
                                String colFamily, String col) throws IOException {
        Table table = connection.getTable(TableName.valueOf(tableName));
        Get get = new Get(rowKey.getBytes());
        get.addColumn(colFamily.getBytes(), col.getBytes());
        Result result = table.get(get);
        System.out.println(new String(result.getValue(colFamily.getBytes(),
            col == null ? null : col.getBytes())));
        table.close();
    }
}
```

运行前需先启动 HDFS 和 HBase。成功后输出 "69"。

---

## 6. MapReduce 编程实践（Hadoop 3.3.5）

### 6.1 词频统计任务

创建测试文件 `wordfile1.txt` 和 `wordfile2.txt`：

```
# wordfile1.txt
I love Spark
I love Hadoop

# wordfile2.txt
Hadoop is good
Spark is fast
```

### 6.2 在 Eclipse 中创建项目

1. 创建 Java 工程 "WordCount"，JDK 选择 jdk1.8.0_371
2. 添加 JAR 包：
   - `/usr/local/hadoop/share/hadoop/common` 下的 hadoop-common-3.3.5.jar 和 hadoop-nfs-3.3.5.jar
   - `/usr/local/hadoop/share/hadoop/common/lib` 下所有 JAR 包
   - `/usr/local/hadoop/share/hadoop/mapreduce` 下所有 JAR 包（不含 jdiff、lib-examples、sources 目录）

### 6.3 WordCount.java 完整代码

```java
import java.io.IOException;
import java.util.Iterator;
import java.util.StringTokenizer;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;

public class WordCount {
    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        String[] otherArgs = (new GenericOptionsParser(conf, args)).getRemainingArgs();
        if (otherArgs.length < 2) {
            System.err.println("Usage: wordcount <in> [<in>...] <out>");
            System.exit(2);
        }
        Job job = Job.getInstance(conf, "word count");
        job.setJarByClass(WordCount.class);
        job.setMapperClass(WordCount.TokenizerMapper.class);
        job.setCombinerClass(WordCount.IntSumReducer.class);
        job.setReducerClass(WordCount.IntSumReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);
        for (int i = 0; i < otherArgs.length - 1; ++i) {
            FileInputFormat.addInputPath(job, new Path(otherArgs[i]));
        }
        FileOutputFormat.setOutputPath(job, new Path(otherArgs[otherArgs.length - 1]));
        System.exit(job.waitForCompletion(true) ? 0 : 1);
    }

    public static class TokenizerMapper extends Mapper<Object, Text, Text, IntWritable> {
        private static final IntWritable one = new IntWritable(1);
        private Text word = new Text();

        public void map(Object key, Text value, Context context)
                throws IOException, InterruptedException {
            StringTokenizer itr = new StringTokenizer(value.toString());
            while (itr.hasMoreTokens()) {
                word.set(itr.nextToken());
                context.write(word, one);
            }
        }
    }

    public static class IntSumReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
        private IntWritable result = new IntWritable();

        public void reduce(Text key, Iterable<IntWritable> values, Context context)
                throws IOException, InterruptedException {
            int sum = 0;
            for (IntWritable val : values) {
                sum += val.get();
            }
            result.set(sum);
            context.write(key, result);
        }
    }
}
```

### 6.4 打包并运行

```bash
# 在 Eclipse 中导出为 Runnable JAR
# Export destination: /usr/local/hadoop/myapp/WordCount.jar

# 启动 Hadoop
cd /usr/local/hadoop
./sbin/start-dfs.sh

# 准备 HDFS 数据
./bin/hdfs dfs -rm -r input
./bin/hdfs dfs -rm -r output
./bin/hdfs dfs -mkdir input
./bin/hdfs dfs -put ./wordfile1.txt input
./bin/hdfs dfs -put ./wordfile2.txt input

# 运行程序
./bin/hadoop jar ./myapp/WordCount.jar input output

# 查看结果
./bin/hdfs dfs -cat output/*
```

**预期输出：**
```
Hadoop 2
I      2
Spark  2
fast   1
good   1
is     2
love   2
```

> **注意**：再次运行前必须删除 HDFS 中的 output 目录

---

## 7. 2023 版完整学习路线

2023 年 7 月版教程（基于 Hadoop 3.3.5 / Ubuntu 22.04）包含以下模块：

| 序号 | 内容 | 说明 |
|------|------|------|
| 1 | 在 VMWare 中安装 Linux 虚拟机 | 环境准备 |
| 2 | Hadoop 3.3.5 安装（单机/伪分布式） | 核心组件 |
| 3 | Hadoop 集群安装配置 | 多节点部署 |
| 4 | HDFS 编程实践（Hadoop 3.3.5） | 分布式文件系统 |
| 5 | HBase 2.5.4 安装和编程实践 | NoSQL 数据库 |
| 6 | MapReduce 编程实践（Hadoop 3.3.5） | 分布式计算 |
| 7 | Hive 3.1.3 安装和使用 | SQL-on-Hadoop |
| 8 | Spark 3.4.0 安装和编程实践 | 内存计算 |
| 9 | Flink 1.16.2 安装与编程实践 | 流处理框架 |

2020 年 6 月版（基于 Hadoop 3.1.3 / Ubuntu 18.04）包含类似模块，版本较旧。

---

## 8. Hive 3.1.3 安装和使用

### 8.1 前置条件

- 已安装 Hadoop 3.1.3（伪分布式模式）
- 已安装 JDK 1.8 和 MySQL（或使用内置 Derby）

### 8.2 安装 Hive

```bash
cd ~
sudo tar -zxf ~/下载/apache-hive-3.1.3-bin.tar.gz -C /usr/local
cd /usr/local
sudo mv ./apache-hive-3.1.3-bin ./hive
sudo chown -R yang ./hive
```

**配置环境变量**，编辑 `~/.bashrc`，添加：

```bash
export HIVE_HOME=/usr/local/hive
export PATH=$PATH:$HIVE_HOME/bin
```

```bash
source ~/.bashrc
```

### 8.3 配置 Hive

**编辑 `/usr/local/hive/conf/hive-site.xml`：**

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:derby:;databaseName=/usr/local/hive/metastore_db;create=true</value>
        <description>JDBC connect string for a JDBC metastore</description>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionDriverName</name>
        <value>org.apache.derby.jdbc.EmbeddedDriver</value>
        <description>Driver class name for a JDBC metastore</description>
    </property>
    <property>
        <name>hive.metastore.schema.verification</name>
        <value>false</value>
    </property>
    <property>
        <name>datanucleus.schema.autoCreateAll</name>
        <value>true</value>
    </property>
</configuration>
```

### 8.4 初始化并启动 Hive

```bash
cd /usr/local/hive
schematool -dbType derby -initSchema    # 初始化元数据库
hive                                     # 启动 Hive Shell
```

### 8.5 Hive 基本操作

```sql
-- 创建数据库
CREATE DATABASE testdb;
USE testdb;

-- 创建表
CREATE TABLE student(id INT, name STRING, age INT)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';

-- 加载数据（从本地文件）
LOAD DATA LOCAL INPATH '/home/yang/student.csv' INTO TABLE student;

-- 查询
SELECT * FROM student;
SELECT name, age FROM student WHERE age > 20;

-- 聚合查询
SELECT age, COUNT(*) FROM student GROUP BY age;

-- 退出
EXIT;
```

---

## 9. Spark 3.4.0 安装和编程实践

### 9.1 Spark 简介

Spark 是基于内存的大数据计算框架，比 Hadoop MapReduce 快 10-100 倍。

**Spark 生态系统：**
- **Spark Core**：核心计算引擎，提供 RDD 编程模型
- **Spark SQL**：结构化数据查询（类似 SQL）
- **Spark Streaming**：实时流处理
- **MLlib**：机器学习库
- **GraphX**：图计算

**Spark vs Hadoop MapReduce：**
- Spark 使用内存计算，MapReduce 需要频繁读写磁盘
- Spark 支持 DAG 执行引擎，MapReduce 只有 Map 和 Reduce 两个操作
- Spark 支持多种编程模式（批处理、流处理、SQL、机器学习）

### 9.2 安装 Spark

```bash
cd ~
sudo tar -zxf ~/下载/spark-3.4.0-bin-hadoop3.tgz -C /usr/local
cd /usr/local
sudo mv ./spark-3.4.0-bin-hadoop3 ./spark
sudo chown -R yang ./spark
```

**配置环境变量**，编辑 `~/.bashrc`，添加：

```bash
export SPARK_HOME=/usr/local/spark
export PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin
```

```bash
source ~/.bashrc
```

### 9.3 配置 Spark

**编辑 `/usr/local/spark/conf/spark-env.sh`：**

```bash
cp /usr/local/spark/conf/spark-env.sh.template /usr/local/spark/conf/spark-env.sh
```

在文件末尾添加：

```bash
export JAVA_HOME=/usr/lib/jvm/jdk1.8.0_202
export HADOOP_HOME=/usr/local/hadoop
export HADOOP_CONF_DIR=/usr/local/hadoop/etc/hadoop
export SPARK_MASTER_HOST=localhost
export SPARK_MASTER_PORT=7077
```

**编辑 `/usr/local/spark/conf/spark-defaults.conf`：**

```bash
cp /usr/local/spark/conf/spark-defaults.conf.template /usr/local/spark/conf/spark-defaults.conf
```

添加：

```
spark.master                     spark://localhost:7077
spark.eventLog.enabled           true
spark.eventLog.dir               hdfs://localhost:9000/spark-logs
spark.history.fs.logDirectory    hdfs://localhost:9000/spark-logs
```

**创建 HDFS 日志目录：**

```bash
hdfs dfs -mkdir -p /spark-logs
```

### 9.4 启动 Spark

```bash
# 启动 Hadoop（如果未启动）
start-dfs.sh

# 启动 Spark Master
/usr/local/spark/sbin/start-master.sh

# 启动 Spark Worker
/usr/local/spark/sbin/start-work.sh

# 或者启动所有
/usr/local/spark/sbin/start-all.sh

# 验证
jps    # 应看到 Master, Worker
```

**Web 界面：** http://localhost:8080（Spark Master UI）

### 9.5 Spark Shell 交互

```bash
# 启动 Spark Shell（本地模式）
spark-shell --master local[*]

# 启动 Spark Shell（连接集群）
spark-shell --master spark://localhost:7077
```

**在 Scala Shell 中测试：**

```scala
// 从本地文件创建 RDD
val textFile = sc.textFile("file:///usr/local/spark/README.md")

// 统计行数
textFile.count()

// 查看第一行
textFile.first()

// 筛选包含"Spark"的行
val linesWithSpark = textFile.filter(line => line.contains("Spark"))
linesWithSpark.count()

// WordCount 示例
val wordCounts = textFile.flatMap(line => line.split(" "))
                         .filter(word => word.nonEmpty)
                         .map(word => (word, 1))
                         .reduceByKey(_ + _)
                         .sortBy(_._2, ascending = false)
wordCounts.take(10).foreach(println)

// 退出
:quit
```

### 9.6 Spark SQL 示例

```bash
# 启动 Spark SQL Shell
spark-sql
```

```sql
-- 创建表
CREATE TABLE IF NOT EXISTS student (id INT, name STRING, age INT)
USING csv OPTIONS (path '/home/yang/student.csv', header 'false');

-- 查询
SELECT * FROM student;
SELECT age, COUNT(*) FROM student GROUP BY age;

-- 退出
quit;
```

### 9.7 PySpark（Python 编程）

```bash
# 启动 PySpark
pyspark --master local[*]
```

```python
# 在 PySpark Shell 中
textFile = sc.textFile("file:///usr/local/spark/README.md")
textFile.count()

# WordCount
wordCounts = textFile.flatMap(lambda line: line.split(" ")) \
                     .filter(lambda word: word != "") \
                     .map(lambda word: (word, 1)) \
                     .reduceByKey(lambda a, b: a + b)
wordCounts.take(10)

exit()
```

### 9.8 提交 Spark 任务

```bash
# 运行自带的示例程序
spark-submit --class org.apache.spark.examples.SparkPi \
    --master local[*] \
    /usr/local/spark/examples/jars/spark-examples_2.12-3.4.0.jar 10

# 运行 Python 脚本
spark-submit --master local[*] wordcount.py
```

---

## 10. Flink 1.16.2 安装与编程实践

### 10.1 Flink 简介

Apache Flink 是流处理框架，支持实时流处理和批量处理。

**Flink vs Spark Streaming：**
- Flink 是真正的流处理（逐条处理），Spark Streaming 是微批处理
- Flink 支持事件时间（Event Time），Spark Streaming 主要支持处理时间
- Flink 延迟更低（毫秒级），Spark Streaming 延迟较高（秒级）

### 10.2 安装 Flink

```bash
cd ~
sudo tar -zxf ~/下载/flink-1.16.2-bin-scala_2.12.tgz -C /usr/local
cd /usr/local
sudo mv ./flink-1.16.2 ./flink
sudo chown -R yang ./flink
```

**配置环境变量**，编辑 `~/.bashrc`，添加：

```bash
export FLINK_HOME=/usr/local/flink
export PATH=$PATH:$FLINK_HOME/bin
```

```bash
source ~/.bashrc
```

### 10.3 配置 Flink

**编辑 `/usr/local/flink/conf/flink-conf.yaml`：**

```yaml
jobmanager.rpc.address: localhost
jobmanager.rpc.port: 6123
jobmanager.memory.process.size: 1600m
taskmanager.memory.process.size: 1728m
taskmanager.numberOfTaskSlots: 2
parallelism.default: 1
rest.port: 8082
env.java.home: /usr/lib/jvm/jdk1.8.0_202
```

> **重要**：`rest.port` 设为 8082，因为 Spark 默认占用 8081 端口。`env.java.home` 必须指定 JDK 1.8，否则 Flink 1.16.2 在 JDK 17 下会报模块访问错误。

### 10.4 启动 Flink

```bash
# 启动 Hadoop（如果未启动）
start-dfs.sh

# 启动 Flink
/usr/local/flink/bin/start-cluster.sh

# 验证
jps    # 应看到 StandaloneSessionClusterEntrypoint, TaskManagerRunner
```

**Web 界面：** http://localhost:8082（Flink Dashboard，端口改为 8082 避免与 Spark 冲突）

### 10.5 Flink Shell 交互

```bash
# Scala Shell
/usr/local/flink/bin/start-scala-shell.sh local

# SQL Shell
/usr/local/flink/bin/sql-client.sh
```

**在 Scala Shell 中测试：**

```scala
// 读取文件
val text = benv.readTextFile("/usr/local/flink/README.md")

// 统计行数
text.count()

// WordCount
val counts = text.flatMap(_.split(" "))
                 .filter(_.nonEmpty)
                 .map((_, 1))
                 .groupBy(0)
                 .sum(1)
counts.print()

// 退出
:quit
```

### 10.6 提交 Flink 任务

```bash
# 运行自带的 WordCount 示例
/usr/local/flink/bin/flink run /usr/local/flink/examples/batch/WordCount.jar

# 查看结果
cat /usr/local/flink/log/flink-*-taskexecutor-*.out
```

### 10.7 Maven 安装与 Flink WordCount 编程实践

#### 10.7.1 安装 Maven

```bash
sudo apt-get install -y maven
mvn --version    # 验证安装
```

#### 10.7.2 创建 Maven 项目

```bash
mkdir -p ~/WordCount/src/main/java/cn/xmu
mkdir -p ~/WordCount/src/main/resources
```

#### 10.7.3 编写 WordCountData.java

```java
package cn.xmu;

import org.apache.flink.api.java.DataSet;
import org.apache.flink.api.java.ExecutionEnvironment;

public class WordCountData {
    public static final String[] WORDS = new String[]{
        "To be, or not to be,--that is the question:--",
        "Whether 'tis nobler in the mind to suffer",
        "The slings and arrows of outrageous fortune",
        "Or to take arms against a sea of troubles,",
        "And by opposing end them?--To die,--to sleep,--",
        "No more; and by a sleep to say we end",
        "The heartache, and the thousand natural shocks",
        "That flesh is heir to,--'tis a consummation",
        "Devoutly to be wish'd. To die,--to sleep;--",
        "To sleep! perchance to dream:--ay, there's the rub;",
        "For in that sleep of death what dreams may come,",
        "When we have shuffled off this mortal coil,",
        "Must give us pause: there's the respect",
        "That makes calamity of so long life;",
        "For who would bear the whips and scorns of time,",
        "The oppressor's wrong, the proud man's contumely,",
        "The pangs of despis'd love, the law's delay,",
        "The insolence of office, and the spurns",
        "That patient merit of the unworthy takes,",
        "When he himself might his quietus make",
        "With a bare bodkin? who would these fardels bear,",
        "To grunt and sweat under a weary life,",
        "But that the dread of something after death,--",
        "The undiscover'd country, from whose bourn",
        "No traveller returns,--puzzles the will,",
        "And makes us rather bear those ills we have",
        "Than fly to others that we know not of?",
        "Thus conscience does make cowards of us all;",
        "And thus the native hue of resolution",
        "Is sicklied o'er with the pale cast of thought;",
        "And enterprises of great pith and moment,",
        "With this regard, their currents turn awry,",
        "And lose the name of action.--Soft you now!",
        "The fair Ophelia!--Nymph, in thy orisons",
        "Be all my sins remember'd."
    };

    public static DataSet<String> getDefaultTextLineDataset(ExecutionEnvironment env) {
        return env.fromElements(WORDS);
    }
}
```

#### 10.7.4 编写 WordCountTokenizer.java

```java
package cn.xmu;

import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.util.Collector;

public class WordCountTokenizer implements FlatMapFunction<String, Tuple2<String, Integer>> {
    @Override
    public void flatMap(String value, Collector<Tuple2<String, Integer>> out) throws Exception {
        String[] tokens = value.toLowerCase().split("\\W+");
        for (String token : tokens) {
            if (token.length() > 0) {
                out.collect(new Tuple2<>(token, 1));
            }
        }
    }
}
```

#### 10.7.5 编写 WordCount.java

```java
package cn.xmu;

import org.apache.flink.api.java.DataSet;
import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.api.java.utils.ParameterTool;

public class WordCount {
    public static void main(String[] args) throws Exception {
        ParameterTool params = ParameterTool.fromArgs(args);
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        env.getConfig().setGlobalJobParameters(params);

        DataSet<String> text;
        if (params.has("input")) {
            text = env.readTextFile(params.get("input"));
        } else {
            System.out.println("Executing WordCount with default input data set.");
            System.out.println("Use --input to specify file input.");
            text = WordCountData.getDefaultTextLineDataset(env);
        }

        DataSet<Tuple2<String, Integer>> counts = text
            .flatMap(new WordCountTokenizer())
            .groupBy(0)
            .sum(1);

        if (params.has("output")) {
            counts.writeAsCsv(params.get("output"), "\n", " ");
            env.execute("WordCount Example");
        } else {
            System.out.println("Printing result to stdout. Use --output to specify output path.");
            counts.print();
        }
    }
}
```

#### 10.7.6 编译打包

由于 Maven 中央仓库可能无法下载 Flink JAR，可使用 Flink 自带的 `flink-dist-1.16.2.jar` 编译：

```bash
cd ~/WordCount
export JAVA_HOME=/usr/lib/jvm/jdk1.8.0_202

# 编译
mkdir -p target/classes
$JAVA_HOME/bin/javac -encoding UTF-8 \
    -classpath /usr/local/flink/lib/flink-dist-1.16.2.jar \
    -d target/classes \
    src/main/java/cn/xmu/*.java

# 打包 JAR
cd target/classes
$JAVA_HOME/bin/jar cvf ~/WordCount/WordCount.jar cn/
```

#### 10.7.7 运行 WordCount

```bash
# 确保 Flink 已启动
/usr/local/flink/bin/start-cluster.sh

# 运行（使用默认数据）
export JAVA_HOME=/usr/lib/jvm/jdk1.8.0_202
/usr/local/flink/bin/flink run -c cn.xmu.WordCount ~/WordCount/WordCount.jar

# 使用自定义输入文件
/usr/local/flink/bin/flink run -c cn.xmu.WordCount ~/WordCount/WordCount.jar --input /path/to/input.txt

# 指定输出路径
/usr/local/flink/bin/flink run -c cn.xmu.WordCount ~/WordCount/WordCount.jar --output /path/to/output
```

项目路径：`~/WordCount/`，JAR 文件：`~/WordCount/WordCount.jar`

---

## 11. 统一启动和关闭指南

### 11.1 启动所有服务（按顺序）

```bash
# 1. 启动 Hadoop HDFS
start-dfs.sh

# 2. 启动 HBase（依赖 HDFS）
start-hbase.sh

# 3. 启动 Spark（依赖 HDFS）
/usr/local/spark/sbin/start-all.sh

# 4. 启动 Flink（独立运行）
/usr/local/flink/bin/start-cluster.sh

# 5. 检查所有进程
jps
```

**预期进程：**
- NameNode, DataNode, SecondaryNameNode（Hadoop）
- HQuorumPeer, HMaster, HRegionServer（HBase）
- Master, Worker（Spark）
- StandaloneSessionClusterEntrypoint, TaskManagerRunner（Flink）

### 11.2 关闭所有服务（反序关闭）

```bash
# 1. 关闭 Flink
/usr/local/flink/bin/stop-cluster.sh

# 2. 关闭 Spark
/usr/local/spark/sbin/stop-all.sh

# 3. 关闭 HBase
stop-hbase.sh

# 4. 关闭 Hadoop
stop-dfs.sh
```

### 11.3 启动 Hive

```bash
# Hive 依赖 Hadoop，先确保 HDFS 已启动
start-dfs.sh

# 启动 Hive Shell（需在 hive 目录下）
cd /usr/local/hive
hive
```

### 11.4 Web 管理界面汇总

| 服务 | 地址 | 说明 |
|------|------|------|
| HDFS NameNode | http://localhost:9870 | 文件系统管理 |
| HBase Master | http://localhost:16010 | HBase 数据库管理 |
| YARN ResourceManager | http://localhost:8088 | YARN 资源管理 |
| Spark Master | http://localhost:8080 | Spark 集群管理 |
| Flink Dashboard | http://localhost:8082 | Flink 任务管理 |
| SSH | localhost:2222 | 远程登录 |

### 11.5 VirtualBox 端口转发配置

由于虚拟机使用 NAT 网络模式，需要配置端口转发才能从 Windows 浏览器访问 Web 界面。在 Windows 命令行中执行：

```bash
"C:/Program Files/Oracle/VirtualBox/VBoxManage.exe" controlvm "Ubuntu22.04" natpf1 "hdfs,tcp,,9870,,9870"
"C:/Program Files/Oracle/VirtualBox/VBoxManage.exe" controlvm "Ubuntu22.04" natpf1 "hbase,tcp,,16010,,16010"
"C:/Program Files/Oracle/VirtualBox/VBoxManage.exe" controlvm "Ubuntu22.04" natpf1 "yarn,tcp,,8088,,8088"
"C:/Program Files/Oracle/VirtualBox/VBoxManage.exe" controlvm "Ubuntu22.04" natpf1 "spark,tcp,,8080,,8080"
"C:/Program Files/Oracle/VirtualBox/VBoxManage.exe" controlvm "Ubuntu22.04" natpf1 "flink,tcp,,8082,,8082"
"C:/Program Files/Oracle/VirtualBox/VBoxManage.exe" controlvm "Ubuntu22.04" natpf1 "ssh,tcp,,2222,,22"
```

验证端口转发配置：

```bash
"C:/Program Files/Oracle/VirtualBox/VBoxManage.exe" showvminfo "Ubuntu22.04" --machinereadable | grep -i "Forwarding"
```

配置完成后，所有服务均可从 Windows 浏览器直接访问：

| 服务 | Windows 访问地址 | 状态 |
|------|------------------|------|
| HDFS | http://localhost:9870 | OK |
| HBase | http://localhost:16010 | OK |
| YARN | http://localhost:8088 | OK |
| Spark | http://localhost:8080 | OK |
| Flink | http://localhost:8082 | OK |
| SSH | localhost:2222 | OK |

### 11.6 一键启动脚本

创建 `/home/yang/start_all.sh`：

```bash
#!/bin/bash
echo "=== 启动所有大数据服务 ==="

echo "启动 Hadoop..."
start-dfs.sh
sleep 3

echo "启动 HBase..."
start-hbase.sh
sleep 3

echo "启动 Spark..."
/usr/local/spark/sbin/start-all.sh
sleep 2

echo "启动 Flink..."
/usr/local/flink/bin/start-cluster.sh
sleep 2

echo ""
echo "=== 所有服务已启动 ==="
jps
echo ""
echo "Web 界面："
echo "  HDFS:    http://localhost:9870"
echo "  HBase:   http://localhost:16010"
echo "  YARN:    http://localhost:8088"
echo "  Spark:   http://localhost:8080"
echo "  Flink:   http://localhost:8082"
```

创建 `/home/yang/stop_all.sh`：

```bash
#!/bin/bash
echo "=== 关闭所有大数据服务 ==="

echo "关闭 Flink..."
/usr/local/flink/bin/stop-cluster.sh

echo "关闭 Spark..."
/usr/local/spark/sbin/stop-all.sh

echo "关闭 HBase..."
stop-hbase.sh

echo "关闭 Hadoop..."
stop-dfs.sh

echo ""
echo "=== 所有服务已关闭 ==="
```

```bash
chmod +x ~/start_all.sh ~/stop_all.sh

# 使用
~/start_all.sh    # 启动所有
~/stop_all.sh     # 关闭所有
```

---

## 附录：常用命令速查

### HDFS 命令

```bash
./bin/hdfs dfs -mkdir -p /user/hadoop   # 创建目录
./bin/hdfs dfs -ls                       # 列出文件
./bin/hdfs dfs -put localfile hdfsdir    # 上传文件
./bin/hdfs dfs -get hdfsfile localdir    # 下载文件
./bin/hdfs dfs -cat hdfsfile             # 查看文件内容
./bin/hdfs dfs -rm -r hdfsdir            # 删除目录/文件
./bin/hdfs dfs -cp src dst               # 拷贝文件
```

### Hadoop 启停命令

```bash
cd /usr/local/hadoop
./sbin/start-dfs.sh     # 启动 HDFS
./sbin/stop-dfs.sh      # 停止 HDFS
jps                     # 查看 Java 进程
```

### HBase 启停命令

```bash
start-hbase.sh          # 启动 HBase
stop-hbase.sh           # 停止 HBase
hbase shell             # 进入 Shell
```

### Hive 命令

```bash
hive                         # 启动 Hive Shell
SHOW DATABASES;              # 查看数据库
USE database_name;           # 切换数据库
SHOW TABLES;                 # 查看表
DESCRIBE table_name;         # 查看表结构
SELECT * FROM table LIMIT 10; # 查询数据
EXIT;                        # 退出
```

### Spark 命令

```bash
spark-shell                  # 启动 Spark Shell (Scala)
pyspark                      # 启动 PySpark (Python)
spark-sql                    # 启动 Spark SQL Shell
spark-submit --class MainClass --master local[*] app.jar  # 提交任务
/usr/local/spark/sbin/start-all.sh   # 启动 Spark 集群
/usr/local/spark/sbin/stop-all.sh    # 停止 Spark 集群
```

### Flink 命令

```bash
/usr/local/flink/bin/start-cluster.sh   # 启动 Flink
/usr/local/flink/bin/stop-cluster.sh    # 停止 Flink
/usr/local/flink/bin/flink run app.jar  # 提交任务
/usr/local/flink/bin/sql-client.sh      # SQL Shell
```

### HBase Shell 命令

```bash
create 'table','cf1','cf2'              # 创建表
put 'table','row','cf:col','value'      # 插入数据
get 'table','row'                       # 查询一行
scan 'table'                            # 扫描全表
delete 'table','row','cf:col'           # 删除数据
deleteall 'table','row'                 # 删除一行
disable 'table'                         # 禁用表
drop 'table'                            # 删除表
exit                                    # 退出 Shell
```

### 关键配置文件路径

| 文件 | 路径 |
|------|------|
| Hadoop 核心配置 | `/usr/local/hadoop/etc/hadoop/core-site.xml` |
| HDFS 配置 | `/usr/local/hadoop/etc/hadoop/hdfs-site.xml` |
| Hadoop 环境变量 | `/usr/local/hadoop/etc/hadoop/hadoop-env.sh` |
| HBase 环境变量 | `/usr/local/hbase/conf/hbase-env.sh` |
| HBase 配置 | `/usr/local/hbase/conf/hbase-site.xml` |
| Hive 配置 | `/usr/local/hive/conf/hive-site.xml` |
| Spark 配置 | `/usr/local/spark/conf/spark-defaults.conf` |
| Spark 环境 | `/usr/local/spark/conf/spark-env.sh` |
| Flink 配置 | `/usr/local/flink/conf/flink-conf.yaml` |
| 用户环境变量 | `~/.bashrc` |

### Web 管理界面（需先配置 VirtualBox 端口转发，见 11.5 节）

| 服务 | 地址 |
|------|------|
| HDFS NameNode | http://localhost:9870 |
| YARN ResourceManager | http://localhost:8088 |
| HBase Master | http://localhost:16010 |
| Spark Master | http://localhost:8080 |
| Flink Dashboard | http://localhost:8082 |
| SSH | localhost:2222 |

---

## 12. 地理空间数据导航与预处理平台（Geographic Data Integration Webpage）

### 12.1 项目简介（中文）

**地理空间数据导航与预处理平台**是一个现代化的 Web 应用，致力于解决地理空间数据获取与预处理的痛点问题。

**核心功能：**
- 🔍 **一站式数据导航** - 聚合 50+ 全球地理空间数据平台，中国平台优先展示
- ⚡ **浏览器端预处理** - 基于 GDAL WebAssembly，数据不上云，隐私安全
- 🤖 **本地 AI 助手** - WebLLM 驱动，自然语言查询，离线可用
- 🌐 **完全离线可用** - 核心功能无需后端服务，纯前端运行

**数据源覆盖：**
- 🇨🇳 中国平台 32 个（地理空间数据云、天地图、风云卫星等）
- 🌍 国际平台 19 个（USGS、NASA、ESA、Google Earth Engine 等）

**技术栈：** GDAL WebAssembly、WebLLM、geotiff.js、proj4js、Tailwind CSS

**GitHub 仓库：** https://github.com/yangernare/Geographic-Data-Integration-Webpage

### 12.2 Project Introduction (English)

**Geographic Data Integration Webpage** is a modern web application designed to solve the pain points of geospatial data acquisition and preprocessing.

**Core Features:**
- 🔍 **One-stop Data Navigation** - Aggregates 50+ global geospatial data platforms, with Chinese platforms prioritized
- ⚡ **Browser-based Preprocessing** - Based on GDAL WebAssembly, data stays local, privacy secured
- 🤖 **Local AI Assistant** - Powered by WebLLM, natural language queries, works offline
- 🌐 **Fully Offline Capable** - Core features require no backend, pure frontend operation

**Data Sources:**
- 🇨🇳 32 Chinese platforms (Geospatial Data Cloud, Tianditu, FengYun Satellite, etc.)
- 🌍 19 International platforms (USGS, NASA, ESA, Google Earth Engine, etc.)

**Tech Stack:** GDAL WebAssembly, WebLLM, geotiff.js, proj4js, Tailwind CSS

**GitHub Repository:** https://github.com/yangernare/Geographic-Data-Integration-Webpage

### 12.3 项目结构（Project Structure）

```
Geographic-Data-Integration-Webpage/
├── index.html          # 主页面（Main page）
├── README.md           # 项目说明文档（Project documentation）
└── CLAUDE.md           # Claude 配置文件（Claude configuration）
```

### 12.4 部署说明（Deployment）

**本地开发（Local Development）：**
```bash
# 直接在浏览器中打开 index.html
# Simply open index.html in your browser
open index.html
```

**GitHub Pages 部署（GitHub Pages Deployment）：**
```bash
# 推送到 GitHub 后，在仓库设置中启用 GitHub Pages
# After pushing to GitHub, enable GitHub Pages in repository settings
# 选择 main 分支作为源
# Select main branch as source
```
