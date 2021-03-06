# ls
Tomcat 8 优化
编辑配置文件：server.xml
1.线程池优化：
默认值：
1.	<!--
2.	<Executor name="tomcatThreadPool" namePrefix="catalina-exec-"
3.	  maxThreads="150" minSpareThreads="4"/>
4.	-->
修改为：
5.	<Executor
6.	      name="tomcatThreadPool"
7.	      namePrefix="catalina-exec-"
8.	      maxThreads="500"
9.	      minSpareThreads="30"
10.	      maxIdleTime="60000"
11.	      prestartminSpareThreads = "true"
12.	      maxQueueSize = "100"
13.	/>
重点参数解释：
name	线程池的名字，如启用线程池，需要在 <Connector port="8080" …… /> 中配置
maxThreads	最大线程数，默认为200, 一般建议设置500~ 800 ，要根据自己的硬件设施条件和实际业务需求而定。
minSpareThreads	最小活跃线程数，默认25
maxIdleTime	空闲线程等待时间，默认 60000 ms（1 min）
maxQueueSize	最大的等待队列数（超出线程池数量的请求会进入排队），超过则请求拒绝，默认值为 java Integer 的最大值 2147483647（暗指不限）
prestartminSpareThreads	是否需要在启动时就生成minSpareThreads这个数量的线程，默认是false

2.连接器优化：
默认值：
14.	<Connector 
15.	  port="8080" 
16.	  protocol="HTTP/1.1" 
17.	  connectionTimeout="20000" 
18.	  redirectPort="8443" 
19.	/>
修改为：
20.	<Connector 
21.	 executor="tomcatThreadPool"
22.	 port="8080" 
23.	 protocol="org.apache.coyote.http11.Http11Nio2Protocol" 
24.	 connectionTimeout="20000" 
25.	 maxConnections="10000" 
26.	 redirectPort="8443" 
27.	 enableLookups="false" 
28.	 acceptCount="100" 
29.	 maxPostSize="10485760" 
30.	 maxHttpHeaderSize="8192" 
31.	 compression="on" 
32.	 disableUploadTimeout="true" 
33.	 compressionMinSize="2048" 
34.	 acceptorThreadCount="2" 
35.	 compressableMimeType="text/html,text/xml,text/plain,text/css,text/javascript,application/javascript" 
36.	 URIEncoding="utf-8"
37.	/>
重点参数解释：
protocol	如果这个用不了，就用org.apache.coyote.http11.Http11NioProtocol
enableLookups	禁用DNS查询
acceptCount	指定当所有可以使用的处理请求的线程数都被使用时，可以放到处理队列中的请求数，超过这个数的请求将不予处理，默认设置 100
maxPostSize	以 FORM URL 参数方式的 POST 提交方式，限制提交最大的大小，默认是 2097152(2兆)，它使用的单位是字节。10485760 为 10M。如果要禁用限制，则可以设置为 -1
acceptorThreadCount	用于接收连接的线程的数量，默认值是1。一般这个指需要改动的时候是因为该服务器是一个多核CPU，如果是多核 CPU 一般配置为 2
maxHttpHeaderSize	http请求头信息的最大程度，超过此长度的部分不予处理。一般8K
禁用 AJP（如果你服务器没有使用 Apache）,把下面这一行注释掉，默认 Tomcat 是开启的。
<!-- <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" /> -->
3.tomcat压缩：

<Connector      port="8080"protocol="HTTP/1.1"

connectionTimeout="20000"

redirectPort="8181"compression="500"

compressableMimeType="text/html,text/xml,text/plain,application/octet-stream"/>
在前面的配置中，当文件的大小大于等于500bytes时才会压缩。如果当文件达到了大小但是却没有被压缩，那么设置属性compression=”on”。否则Tomcat默认设置是“off”。
JVM 优化
Linux 修改 catalina.sh 文件
如果服务器只运行一个 Tomcat，机子内存如果是 4G：
JAVA_OPTS="-Xms2G -Xmx2G -Xmn512m -XX:MetaspaceSize=512M -XX:MaxMetaspaceSize=512M -XX:+UseConcMarkSweepGC -XX:+CMSClassUnloadingEnabled -XX:+HeapDumpOnOutOfMemoryError -verbose:gc -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps -Xloggc:/appl/gc.log -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSInitiatingOccupancyOnly"
参数说明：
38.	 –Xss128k 应该是128k 够用的，大的应用建议使用 256k 或 512K，默认值linux/arm（32位）为320k，linux/i386( 32位)为320k，linux/x64（64位）为1024k
39.	 -server 服务器模式，启动慢，运行快，多个cpu性能佳，反之-client
40.	 -Xms2G 初始分配的堆内存,最多不超过80%
41.	 -Xmx2G 最大允许分配的堆内存，这两个配成一样节省堆内存分配耗时。
42.	 -Xmn512m 年轻代大小，Sun官方推荐配置为整个堆的3/8
43.	 -XX:MetaspaceSize=512M 初始元空间大小，达到该值就会触发垃圾收集进行类型卸载，同时GC会对该值进行调整：如果释放了大量的空间，就适当降低该值；如果释放了很少的空间，那么在不超过MaxMetaspaceSize时，适当提高该值。元空间的本质和永久代类似，都是对JVM规范中方法区的实现。不过元空间与永久代之间最大的区别在于：元空间并不在虚拟机中，而是使用本地内存。 
44.	 -XX:MaxMetaspaceSize=512M 
45.	 -XX:+UseConcMarkSweepGC 并发标记清除（CMS）收集器，垃圾收集器的选择：CMS收集器（并发回收）：暂停时间优先，Parallel收集器（并行回收）：吞吐量优先
46.	 -XX:+CMSClassUnloadingEnabled 
47.	 -XX:+HeapDumpOnOutOfMemoryError 表示当JVM发生OOM时，自动生成DUMP文件。 
48.	 -XX:HeapDumpPath=${目录}参数表示生成DUMP文件的路径，也可以指定文件名称，例如：-XX:HeapDumpPath=${目录}/java_heapdump.hprof。如果不指定文件名，默认为：java_<pid>_<date>_<time>_heapDump.hprof。
49.	 -verbose:gc 输出GC日志 ，  -XX:+PrintGC 与 -verbose:gc 是一样的，可以认为-verbose:gc 是 -XX:+PrintGC的别名.
50.	 -XX:+PrintGCDetails 打印GC详细信息
51.	 -XX:+PrintGCTimeStamps 打印gc时间戳
52.	 -XX:+PrintGCDateStamps 
53.	 -Xloggc:/appl/gc.log  定义gc日志目录
54.	 -XX:CMSInitiatingOccupancyFraction=75 是指设定CMS在对内存占用率达到75%的时候开始GC(因为CMS会有浮动垃圾,所以一般都较早启动GC); 
55.	 -XX:+UseCMSInitiatingOccupancyOnly 只是用设定的回收阈值(上面指定的75%),如果不指定,JVM仅在第一次使用设定值,后续则自动调整
TCP 优化
修改 /etc/sysctl.conf
生效： sysctl –p
参数说明：
56.	 net.ipv4.tcp_mem = 196608  262144  393216
57.	 #（4G 内存机器 使用，TCP连接最多约使用1.6GB内存 ， 393216*4096/1024/1024=1.6G）
58.	 #内核分配给TCP连接的内存，单位是Page，1 Page = 4096 Bytes
59.	 net.ipv4.tcp_mem = 524288  699050  1048576 
60.	 #（8G 内存使用，TCP连接最多约使用4GB内存） 
61.	 #为每个TCP连接分配的读、写缓冲区内存大小，单位是Byte
62.	 net.ipv4.tcp_rmem = 4096      8192    4194304 
63.	 net.ipv4.tcp_wmem = 4096      8192    4194304 
64.	 #                  最小内存  缺省内存  最大内存
65.	 # 一般按照缺省值分配，上面的例子就是读写均为8KB，共16KB
66.	 #1.6G 内存服务器， TCP内存能容纳的连接数，约为  1600MB/16KB = 100K = 10万
67.	 #4.G TCP内存能容纳的连接数，约为  4000MB/16KB = 250K = 25万
68.	 net.core.somaxconn= 4000 
69.	 #（端口最大的监听队列的长度）
70.	 #同时，修改下全局配置 
71.	 # echo 4000 > /proc/sys/net/core/somaxconn 定义了系统中每一个端口最大的监听队列的长度,这是个全局的参数,默认值为128
72.	 net.ipv4.tcp_syncookies = 1 
73.	 #表示开启SYN Cookies。当出现SYN等待队列溢出时，启用cookies来处理，可防范少量SYN攻击，默认为0，表示关闭；
74.	 net.ipv4.tcp_tw_reuse = 1 
75.	 #表示开启重用。允许将TIME-WAIT sockets重新用于新的TCP连接，默认为0，表示关闭； 
76.	 net.ipv4.tcp_fin_timeout = 30
77.	 #修改系統默认的 TIMEOUT 时间。 
78.	 net.ipv4.tcp_keepalive_time = 1200   
79.	 #表示当keepalive起用的时候，TCP发送keepalive消息的频度。缺省是2小时，改为20分钟。
80.	 net.ipv4.ip_local_port_range = 10000 65000   
81.	 #表示用于向外连接的端口范围。缺省情况下很小：32768到61000，改为10000到65000。 
82.	 #（注意：这里不要将最低值设的太低，否则可能会占用掉正常的端口！）
83.	 net.ipv4.tcp_max_syn_backlog = 8192 
84.	 #表示SYN队列的长度，默认为1024，加大队列长度为8192，可以容纳更多等待连接的网络连接数。
85.	 net.ipv4.tcp_max_tw_buckets = 5000 
86.	 #表示系统同时保持TIME_WAIT的最大数量，如果超过这个数字，TIME_WAIT将立刻被清除并打印警告信息。默 认为180000，改为5000。
87.	 net.ipv4.tcp_max_orphans = 65536 
88.	 #当orphans达到32768个时，会报Out of socket memory，此时占用内存 32K*64KB=2048MB=2GB
89.	 #（每个孤儿socket可占用多达64KB内存），实际可能小一些
90.	 net.ipv4.tcp_orphan_retries = 1
91.	 #孤儿socket废弃前重试的次数，重负载web服务器建议调小，设置较小的数值，可以有效降低orphans的数量 
92.	 net.ipv4.tcp_retries = 2
93.	 #活动TCP连接重传次数，超过次数视为掉线，放弃连接。缺省值：15，建议设为 2或者3. 
94.	 net.ipv4.tcp_synack_retries
95.	 #TCP三次握手的syn/ack阶段，重试次数，缺省5，设为2-3 
96.	 net.core.netdev_max_backlog = 2048
97.	 # 网络设备的收发包的队列大小 
Tomcat数据库调优
Tomcat性能在等待数据库查询被执行期间会降低。如今大多数应用程序都是使用可能包含“命名查询”的关系型数据库。如果是那样的话，Tomcat会在启动时默认加载命名查询，这个可能会提升性能。另一件重要事是确保所有数据库连接正确地关闭。给数据库连接池设置正确值也是十分重要的。我所说的值是指Resource要素的最大空闲数（maxIdle），最大连接数（maxActive）,最大建立连接等待时间（maxWait）属性的值。因为配置依赖与应用要求，我也不能在本文指定正确的值。你可以通过调用数据库性能测试来找到正确的值。

Jvm参考配置
配置	线程池	连接器	jvm参数
8G4核	name="tomcatThreadPool"	      namePrefix="catalina-exec-"
maxThreads="500"	      minSpareThreads="30"
maxIdleTime="60000"     prestartminSpareThreads = "true"
maxQueueSize = "100"	executor="tomcatThreadPool"
port="8080" 
protocol="org.apache.coyote.http11.Http11Nio2Protocol" 
connectionTimeout="20000" 
maxConnections="10000" 
redirectPort="8443" 
enableLookups="false" 
acceptCount="100" 
maxPostSize="10485760" 
maxHttpHeaderSize="8192" 
compression="on" 
disableUploadTimeout="true" 
compressionMinSize="2048" 
acceptorThreadCount="2"
compressableMimeType="text/html,text/xml,text/plain,
text/css,text/javascript,application/javascript" 
URIEncoding="utf-8"	-server –Xms4G –Xmx4G –Xmn1024m -XX:MetaspaceSize=1024M -XX:MaxMetaspaceSize=1024M -XX:+UseConcMarkSweepGC -XX:+CMSClassUnloadingEnabled -XX:+HeapDumpOnOutOfMemoryError -verbose:gc -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps -Xloggc:/appl/gc.log -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSInitiatingOccupancyOnly


