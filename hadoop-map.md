# 从 HDFS 中使用分布式的 MAP REDUCE JOB 写入 CASSANDRA

> 这篇文章是 MANU MUKERJI 于2015年7月17日所写。文章介绍了 Cassandra 的相关操作。原文地址：[https://www.packtpub.com/books/content/writing-cassandra-hdfs-using-hadoop-map-reduce-job](https://www.packtpub.com/books/content/writing-cassandra-hdfs-using-hadoop-map-reduce-job)

在这篇文章中我会讲解如何设置允许您写入 Cassandra 的 Map Reduce Job。这里介绍的用例将把串流分析包括到 Cassandra 中。

我想在我们开始之前，您有可用的 Cassandra 集群和 Hadoop 集群，甚至单个实例或本地主机就足够了。用于此示例的代码可在 [https://github.com/manum/mr-cassandra](https://github.com/manum/mr-cassandra) 中获得。

让我们创建我们将要使用的 Cassandra Keyspace 和 Table。您可以在 cqlsh 中运行以下代码(命令行实用程序，可以让你跟 Cassandra 交互)。

表 keytable 只有一列叫作 key；它将是我们存储数据的地方。

```
CREATE KEYSPACE keytest WITH REPLICATION = { 'class' : 'NetworkTopologyStrategy', 'datacenter1' : 3 };

CREATE TABLE keytable (
  key varchar,
PRIMARY KEY (key)
);
```

这是它在运行之后的样子：

```
cqlsh> USE keytest;
cqlsh:keytest> select * from keytable;
 key
----------
 test1234

(1 rows)
```

我们可以从浏览 CassandraHelper.java 和 CassandraTester.java 源码开始。

**CassandraHelper** 方法：

getSession()：检索当前会话对象以确保没有其他会话对象被创建。


```
public Session getSession()  {
         LOG.info("Starting getSession()");
        if (this.session == null && (this.cluster == null || this.cluster.isClosed())) {
            LOG.info("Cluster not started or closed");
        } else if (this.session.isClosed()) {
            LOG.info("session is closed. Creating a session");
            this.session = this.cluster.connect();
        }

        return this.session;
    }
```

createConnection(String)：为 Cassandra 服务器传递 host。


```
public void createConnection(String node)  {

        this.cluster = Cluster.builder().addContactPoint(node).build();

        Metadata metadata = cluster.getMetadata();
            
        System.out.printf("Connected to cluster: %s\n",metadata.getClusterName());
        
        for ( Host host : metadata.getAllHosts() ) {
            System.out.printf("Datatacenter: %s; Host: %s; Rack: %s\n", host.getDatacenter(), host.getAddress(), host.getRack());
        }
        this.session = cluster.connect();

        
        this.prepareQueries();

    }
```

closeConnection()：在一切都完成之后关闭连接。


```
public void closeConnection() {
        cluster.close();
    }
```

prepareQueries()：此方法准备的查询在服务器端进行了优化。如果您经常执行相同的查询或查询不更改但仅仅改变数据，例如在插入操作时，它推荐您使用预查询。

```
private void prepareQueries()  {
        LOG.info("Starting prepareQueries()");
        this.preparedStatement = this.session.prepare(this.query);
    }
```

addKey(String)：该方法将数据添加到集群，它还有 try catch 捕获异常区域并且告诉您正在发生什么。

```
public void addKey(String key) {
        Session session = this.getSession();
        
        if(key.length()>0) {
            try {
                session.execute(this.preparedStatement.bind(key) );
                //session.executeAsync(this.preparedStatement.bind(key));
            } catch (NoHostAvailableException e) {
                System.out.printf("No host in the %s cluster can be contacted to execute the query.\n", 
                        session.getCluster());
                Session.State st = session.getState();
                for ( Host host : st.getConnectedHosts() ) {
                    System.out.println("In flight queries::"+st.getInFlightQueries(host));
                    System.out.println("open connections::"+st.getOpenConnections(host));
                }

            } catch (QueryExecutionException e) {
                System.out.println("An exception was thrown by Cassandra because it cannot " +
                        "successfully execute the query with the specified consistency level.");
            }  catch (IllegalStateException e) {
                System.out.println("The BoundStatement is not ready.");
            }
        }
    }
```

CassandraTester：该类有一个 void main 方法，您需要提供想要连接的主机并且它会将值 "test1234" 写入到 Cassandra 中。

MapReduceExample.java 是这里比较有趣的一个文件。它使用 Mapper 类和 Reducer 类以及 main 方法来初始化工作。在 Mapper 下，你会发现 setup() 和 cleanup() 方法——它们会被 Map Reduce 框架自动调用来进行处理设置和清理操作——您将使用它来完成连接到 Cassandra 并且在连接之后对连接的清理的工作。

我修改了标准的单词计数的例子，现在的方案对行计数，并且会将它们都写入 Cassandra。reducer 的输出基本上是行数和总数。

若要运行本示例，您需要做以下几点：

1. 从 [https://github.com/manum/mr-cassandra](https://github.com/manum/mr-cassandra) 复制 repo。

2. 运行 mvn install 来在目标的 /folder 中创建一个 jar。

3. 用 scp 命令将 jar 复制到您的分布式集群中。

4. 复制测试输入(对于这个测试，我使用了 git 上莎士比亚的所有作品 all-shakespeare.txt)

5. 使用以下命令运行 jar：hadoop jar mr_cassandra-0.0.1-SNAPSHOT-jar-with-dependencies.jar com.example.com.mr_cassandra.MapReduceExample /user/ubuntu/all-shakespeare.txt /user/ubuntu/output/

如果您运行上述步骤，它应该可以启动这份工作了。在工作完成之后，转到 cqlsh 并运行 select * from keytable limit 10;

```
cqlsh:keytest> select * from keytable limit 10;

 key
----------------------------------------------------------------
          REGAN\tGood sir, no more; these are unsightly tricks:
                   KING\tWe lost a jewel of her; and our esteem
                                        ROSALIND\tAy, but when?
                                              \tNow leaves him.
                           \tThy brother by decree is banished:
 DUCHESS OF YORK\tI had a Richard too, and thou didst kill him;
             JULIET\tWho is't that calls? is it my lady mother?
           ARTHUR\tO, save me, Hubert, save me! my eyes are out
                 \tFull of high feeding, madly hath broke loose
                     \tSwift-winged with desire to get a grave,

(10 rows)

cqlsh:keytest>
```

## 关于作者

Manu Mukerji 有云计算和大数据方面的知识背景，每天实时处理数以亿计的交易。他喜欢构建和设计可扩展、具有高可用性数据的解决方案。并且他在网络广告和社交媒体方面有丰富的工作经验。

[twitter:@next2manu](twitter:@next2manu)

LinkedIn: [http://www.linkedin.com/in/manumukerji/](http://www.linkedin.com/in/manumukerji/)
