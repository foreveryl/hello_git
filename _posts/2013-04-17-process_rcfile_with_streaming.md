---
layout: post
title: Hadoop Streaming方式处理RCFile数据
---

0, 利用Hadoop Streaming方式, 可以跨语言来写Map-Reduce程序. 当数据格式是非text format时 如RCFile(关于RCFile, 参见wiki文档:
<a href='http://en.wikipedia.org/wiki/RCFile'>http://en.wikipedia.org/wiki/RCFile </a>) 如何使用Streaming方式呢?

1, Streaming官网文档: http://hadoop.apache.org/docs/r1.1.2/streaming.html 中明确指出, 可以通过:
"
-inputformat JavaClassName	 Optional	 Class you supply should return key/value pairs of Text class. If not specified, TextInputFormat is used as the default

"
参数来制定Input的Format

blog: http://www.linuxidc.com/Linux/2012-04/57829.htm 中很直接的给出自定义format的解析方法: 

原理如下:
1, 写解析的java类
2, 编译出class, 注意: package 必须是hadoop-streaming-1.0.1.jar, 因为hadoop streaming是在该路径下寻找解析类
3, 增加至hadoop-streaming-1.0.1.jar 



2, 对RCFile而言, hive中已经造出了轮子. 而自己写有明显太麻烦. 
在路径: org/apache/hadoop/hive/ql/io 下. 
所以需要移植下. 

我对java不是很熟. 但如下的做法是可以的. 
步骤如下:
a, copy org/apache/hadoop/hive/ql/io/*.java 到 org/apache/hadoop/streaming 下
b, 修改一下文件package名, 在每个文件中,import org.apache.hadoop.hive.ql.io*(重用hive的jar包, 省去编译hive)  编译RCFile.java, RCFileInputFormat.java 文件, 生成相应的class

javac -classpath path-to-hadoop-core-0.20.2-cdh3u1.jar:path-to/hive-exec-0.7.0-cdh3u0.jar:path-to/hadoop-/lib/* ./org/apache/hadoop/streaming/RCFileInputFormat.java 

c, 将class文件"加入"到hadoop-streaming.jar中
jar uf hadoop-streaming-0.20.2-cdh3u1.jar org/apache/hadoop/streaming/*.class

这时候运行streaming的话, 发现还缺少很多模块. 
解决办法:

解压出hive中的class
jar zvf hive....jar

将缺少的class 在一次加入到hadoo-streaming中. 

经尝试, 加入以下的依赖:
jar uf hadoop-streaming-0.20.2-cdh3u1.jar org/apache/hadoop/hive/serde2/*.class
jar uf hadoop-streaming-0.20.2-cdh3u1.jar org/apache/hadoop/hive/ql/io/*.class
jar uf hadoop-streaming-0.20.2-cdh3u1.jar org/apache/hadoop/hive/conf/HiveConf*.class
jar uf hadoop-streaming-0.20.2-cdh3u1.jar org/apache/hadoop/hive/serde2/*/*.class
jar uf hadoop-streaming-0.20.2-cdh3u1.jar org/apache/hadoop/hive/common/*.class
jar uf hadoop-streaming-0.20.2-cdh3u1.jar org/apache/hadoop/hive/common/io/NonSyncByteArray*.class


3, 使用python mrjob 来编写python 的mr. 
需要MRJob继承类中重定义:
def hadoop_input_format(self):
    return 'RCFileInputFormat'

即可

Job is Done!





