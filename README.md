# centOS7 搭建 单机版hadoop2.6.5

#### 自己下载  `VMware Workstation Pro`
##### CentOs7.ios  `https://pan.baidu.com/s/1O_0ZU8gNd54MMWM7b6Bd8A 提取码：99f2`
##### hadoop2.6.5  `https://pan.baidu.com/s/1h2kM1sl08wwkqYDcI0h-NA 提取码：enim `
##### jdk-8u201  `https://pan.baidu.com/s/1EGRj1xqHKdvfNCGfgsQesg 提取码：cdon `

## 1.下载VMware虚拟机安装CentOs7.ios虚拟机
* 虚拟机联网执行 `vi /etc/sysconfig/network-scripts/ifcfg-ens33` 修改`ONBOOT=yes`
* 虚拟机拿到`hadoop-2.6.5.tar.gz`和`jdk-8u201-linux-x64.tar.gz`文件后，假设在/opt目录下

## 2.解压java jdk，配置java环境变量
* 执行解压 `tar -xzvf /opt/jdk-8u201-linux-x64.tar.gz` 得到 `/opt/jdk1.8.0_201`
* 执行 `vi /etc/profile` 打开设置环境变量文件 新增
```javascript
export JAVA_HOME=/opt/jdk1.8.0_201
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```

## 3。解压hadoop，并配置
* 执行解压 `tar -xzvf /opt/hadoop-2.6.5.tar.gz` 得到 `/opt/hadoop-2.6.5`
* 执行 `vi /etc/profile` 打开设置环境变量文件 新增
```javascript 
export HADOOP_HOME=/opt/hadoop-2.6.5
export PATH=$PATH:$HADOOP_HOME/bin
```
* 修改hadoop的java环境变量 `vi /opt/hadoop-2.6.5/etc/hadoop/hadoop-env.sh` 新增 `export JAVA_HOME=/opt/jdk1.8.0_201
`
* 修改hadoop配置 `vi /opt/hadoop-2.6.5/etc/hadoop/core-site.xml`新增
```html
    <configuration>
        <property>
            <name>fs.defaultFS</name>
            <value>hdfs://localhost:9000</value>
            <!--9000是随便写的 随便写都行-->
        </property>
    </configuration>
```

* 修改hadoop配置 `vi /opt/hadoop-2.6.5/etc/hadoop/hdfs-site.xml`新增
```html
    <configuration>
        <property>
            <name>dfs.replication</name>
            <value>1</value>
        </property>
    </configuration>
```

* `cd /opt/hadoop-2.6.5` 执行 `./bin/hadoop namenode -format` 格式化hdfs

* `cd /opt/hadoop-2.6.5` 执行 `./sbin/start-dfs.sh` 启动hadoop; `./sbin/stop-dfs.sh`是暂停命令
* 执行`jps`检查是否执行成功,下面则证明执行成功
```javascript
    34208 SecondaryNameNode
    34002 NameNode
    38020 Jps
    33848 DataNode
```

## 4.编写`MapReduce`
### "任何可以使用JavaScript来编写的应用，最终会由JavaScript编写。"

### 1. centOs 安装node环境

##### 1. 在`/opt`下执行 `wget http://nodejs.org/dist/v8.12.0/node-v8.12.0.tar.gz`获取node包
##### 2. 执行`tar zxvf node-v8.12.0.tar.gz`解压
##### 3. 执行`vi /etc/profile`添加`node`环境变量
```javascript
export NODE_HOME=/opt/node-v8.12.0
export PATH=$NODE_HOME/bin:$PATH
```

### 2. 目录结构
```javascript
 ./map.js
 ./reduce.js
 ./wordcount.txt
```

### 3. `wordcount.txt` 内容
```javascript
    js java python js java js go react vue vue
```

### 4. `map.js` 内容
```javascript
const readline = require('readline')
const rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout
})
rl.on('line', line => {
    line.aplit(' ').map(word => {
        console.log(`${word}\tl`)
    })
})
rl.on('close', () => {
    process.exit(0)
})
```

### 5. `reduce.js` 内容
```javascript
const reduline = require('readline')
const rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout,
    terminal: false
})

let words = new Map()

rl.on('line', line => {
    const [word, count] = line.split('\t')
    if (!words.has(word)) {
        words.set(word, parseInt(count))
    } else {
        words.set(word, words.get(word) + 1)
    }
})

rl.on('clise', () => {
    words.forEach((v, k) => {
        console.log(`${k}\t${v}`)
    })
    process.exit(0)
})
```

## 5. 执行`MapReduce`
* 假设文件路径为
```javascript
 /opt/test/map.js
 /opt/test/reduce.js
 /opt/test/wordcount.txt
```
* 执行`chmod +x /opt/test/map.js /opt/test/reduce.js`将`map.js`和`reduce.js`变成可执行文件
* 执行 `hadoop fs -mkdir /input`给`hadoop`文件系统根目录创建`input`文件夹
* 执行 `hadoop fs -put /opt/test/wordcount.txt /input` 将`wordcount.txt`放在`input`文件夹内
* 进入`test`文件夹`cd /opt/test` 执行
```javascript
hadoop jar $HADOOP_HOME/share/hadoop/tools/lib/hadoop-streaming-2.6.5.jar -input /input/wordcount.txt -output /output -mapper "node ./map.js" -reducer "node ./reduce.js"
```
* 截至目前 任务已经提交运行完

* 执行`hadoop fs -ls /output` 查看hadoop文件系统`/output`目录
```txt
    -rw-r--r--   1 root supergroup          0 2019-04-11 14:33 /output/_SUCCESS
    -rw-r--r--   1 root supergroup         40 2019-04-11 14:33 /output/part-00000
```

* 执行`hadoop df -cat /output/part-00000`查看输出文件

```javascript
go	1
java	2
js	3
python	1
react	1
vue	2

```
