基于wifi探针的商业大数据分析
================

##一、项目结构：
整个项目分为三个模块：
* **数据采集与存储**：WiFi探针采集到实时数据发送到数据缓冲服务器。
* **数据分析**：利用分布式计算引擎处理实时流数据。
* **数据可视化**：将分析好的数据制作成图表显示在Web端。

##二、设计思路
本课题最大的难点就是对海量数据进行实时数据分析，WiFi探针可以精确定位到的区域为半径100米内，考虑到商业地段人流量很大，扫描到的mac最少1000以上，一个探针1s发送的数据包在80k左右，一分钟的数据量在1600k左右，需要做到最少1000个并发，也就是说一分钟内最少需要对1.6G的数据进行数据分析并且实时在web端显示计算的结果，即使设置一定的延迟处理，一个小时也需要处理接近100G数据，是一个经典的大数据问题。
* *Hadoop Yarn + HDFS + Spark Streaming + MongoDB的分布式架构*：
WiFi探针采集到的原始数据通过数据缓存服务器存储到HDFS分布式文件系统，Spark Streaming进行实时流计算，计算完成后存入MongoDB数据库。
* *数据库设计*：
考虑到客流量分析的多样性，使用没有结构的文档型数据库可以很方便的扩展将来新的数据分析需求。
* *数据库设计优化-利用富文档结构进行分桶*：
该数据库主要用来存储计算完后的数据，数据形式为以分钟为最小单位的时间序列数据、店铺信息与当前在线mac的信息和所有曾经来过区域的mac历史行为记录，所以体量非常大，如果以普通设计将每分钟数据作为一条文档存入，则查询起来性能太差，同时也会占用大量的存储，所以可以采用**分桶设计**来获得几十倍的性能增长。具体来说就是把分析后的数据按小时为一个桶，然后一个店铺一天的数据为一个桶，聚合到一个文档里，每分钟（每小时）的值用子文档的一个字段来表示。这样的好处就是大量减少文档的数量，相应的索引数量也会减少，总体写入OI将会大幅度降低并得到性能提升。使用这种分时我们还可以把一些统计需要的数值，如每小时的平均值预先就作为一个字段存进去，需要的时候不需要现场计算，只要从文档里读出来即可，即将大部分计算任务都放在集群分布式计算从而提高整个系统的性能。
* *业务扩展*：
需要扩展新的数据分析业务比如用机器学习算法预测客流量等，只需将新的任务部署到Yarn集群即可。
* *架构可行性分析*：
所运用的技术都是时下最流行的分布式大数据技术，已经有很多大型企业使用Spark+MongoDB架构取得了成功。
![成功案例](http://opkumgn53.bkt.clouddn.com/spark%E6%88%90%E5%8A%9F%E6%A1%88%E4%BE%8B.png)


##三、系统架构图
#####系统整体和部分模块架构架构图
![系统整体架构](http://opkumgn53.bkt.clouddn.com/wifiAnalysis.png)
![数据分析架构](http://opkumgn53.bkt.clouddn.com/%E6%95%B0%E6%8D%AE%E5%88%86%E6%9E%90%E6%9E%B6%E6%9E%84.png)
![实时流计算流程](http://opkumgn53.bkt.clouddn.com/Streaming%E6%9E%B6%E6%9E%843.png)
![web框架](http://opkumgn53.bkt.clouddn.com/web%E7%B3%BB%E7%BB%9F%E6%A1%86%E6%9E%B6.png)
##四、部署应用
1.将数据分析应用部署到Yarn集群

```python
./bin/spark-submit \
--master yarn \
--deploy-mode client \
--name "WifiAnalysis" \
--queue exampleQueue \
--num-executors  1 \
--executor-memory 1g \
Domain.py
```
2.Web端部署

    将eclipse中导出的war包，放到tomcat目录下。

##五、项目文件说明
* *Document*：项目的所有文档
* *Source*：项目源代码
    * Domain.py：数据分析源代码
    * xxx.py：服务器并发压力测试代码
        * 利用一台现有的wifi探针采集到的一个半小时数据，模拟出一千台wifi探针向服务器发送数据，用来测试服务器并发压力。
    * xxxxx：Web端源代码
* *PPT*：项目PPT
* *Vedio*：演示视频链接
##六、演示材料
* 视频地址：[基于Wifi探针的商业大数据分析演示视频](https://segmentfault.com/markdown#)
* PPT：（写PPT文件位置）


  [1]: http://opkumgn53.bkt.clouddn.com/web%E7%B3%BB%E7%BB%9F%E6%A1%86%E6%9E%B6.png
