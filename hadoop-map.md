# 从 HDFS 中使用分布式的 MAP REDUCE JOB 写入 CASSANDRA   --   王运里

作者：MANU MUKERJI  

时间：2015年7月17日

原文地址：[https://www.packtpub.com/books/content/writing-cassandra-hdfs-using-hadoop-map-reduce-job](https://www.packtpub.com/books/content/writing-cassandra-hdfs-using-hadoop-map-reduce-job)
## 关于本文

文章开头介绍了 Cassandra 中表的创建等基本知识。随后介绍了java 中对 Cassandra 的操作的库 CassandraHelper.java，CassandraTester.java，MapReduceExample.java，还有其中的 getSession()，createConnection(String)，closeConnection()，prepareQueries()，addKey(String) 等方法。文章最后，作者给出了一个他修改过的标准单词计数的示例，他将计数的数据写入到 Cassandra 中，并在 cqlsh 中查看了运行结果。作者在文章中详细地给出了运行此示例的步骤。此外，作者还在文章中提供了他对该示例测试时所用的数据集，您可以从 git 上获取它。

## 文章内容

在这篇文章中我会讲解如何设置允许您写入 Cassandra 的 Map Reduce Job。这里介绍的用例将包括串流分析到 Cassandra 中。 

我想在我们开始之前，你有可用的 Cassandra 集群和 Hadoop 集群，甚至单个实例或本地主机就足够了。用于此示例的代码可在 [https://github.com/manum/mr-cassandra](https://github.com/manum/mr-cassandra) 中获得。

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

我们可以从看 CassandraHelper.java 和 CassandraTester.java 开始。

**CassandraHelper** 方法：

getSession():检索当前会话对象以确保没有其他会话对象被创建。


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

createConnection(String): 为 Cassandra 服务器传递 host。


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

closeConnection(): 在一切都完成之后关闭连接。


```
public void closeConnection() {
        cluster.close();
    }
```

prepareQueries():此方法准备的查询在服务器端进行了优化。如果您经常执行相同的查询或查询不会更改但数据可能改变，例如在插入操作时，它推荐您使用预查询。

```
private void prepareQueries()  {
        LOG.info("Starting prepareQueries()");
        this.preparedStatement = this.session.prepare(this.query);
    }
```

addKey(String):该方法将数据添加到群集，它还有try catch块捕获异常并告诉你正在发生什么。

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

CassandraTester: 该类有一个 void main 方法，您需要提供想要连接的主机并且它会将值 "test1234" 写入到 Cassandra 中。

MapReduceExample.java 是这里的感兴趣的文件。它有一个 Mapper 类和 Reducer 类和 main 方法来初始化工作。在 Mapper 下，你会发现 setup() 和 cleanup() 方法——它们会被 Map Reduce 框架自动调用来处理设置和清理操作——您将使用连接到 Cassandra 和之后的清理连接。

我修改了标准的单词计数的例子，现在的方案对行计数，并且会将它们都写入 Cassandra。reducer 的输出基本上是各行和行数。

若要运行本示例，您需要做以下几点：

1. 从 [https://github.com/manum/mr-cassandra](https://github.com/manum/mr-cassandra) 复制 repo。

2. 运行 mvn install 来在目标的 /folder 中创建一个 jar。

3. 用 scp 命令将 jar 复制到您的分布式集群中。

4. 复制测试输入(对于这个测试，我使用了 git 上莎士比亚的所有作品 all-shakespeare.txt)

5. 使用以下命令运行 jar：hadoop jar mr_cassandra-0.0.1-SNAPSHOT-jar-with-dependencies.jar com.example.com.mr_cassandra.MapReduceExample /user/ubuntu/all-shakespeare.txt /user/ubuntu/output/

如果您运行上述步骤，它应该启动这份工作。在工作完成之后，转到 cqlsh 并运行 select * from keytable limit 10;

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

Manu Mukerji 有云计算和大数据方面的背景，实时处理数以亿计的每天的交易。他喜欢建筑和设计可扩展、高可用性数据的解决方案，并在网络广告和社交媒体方面有丰富的工作经验。

[twitter:@next2manu](twitter:@next2manu)

LinkedIn: [http://www.linkedin.com/in/manumukerji/](http://www.linkedin.com/in/manumukerji/)
