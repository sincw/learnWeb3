# 分布式哈希表DHT

[分布式哈希表DHT（Kademlia算法）——通俗易懂_qq_26720653的博客-CSDN博客](https://blog.csdn.net/qq_26720653/article/details/106496916)

##### 概述

Kademlia算法是一种分布式存储及路由的算法。什么是分布式存储？试想一下，一所1000人的学校，现在学校突然决定拆掉图书馆（不设立中心化的服务器），将图书馆里所有的书都分发到每位学生手上（所有的文件分散存储在各个节点上）。即是所有的学生，共同组成了一个分布式的图书馆。

![](.\image\aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy85NDcyMDktNzk1ZjNhZjcyMzExMDhiNy5wbmc_aW1hZ2VNb2dyMi9hdXRvLW9yaWVudC9zdHJpcHxpbWFnZVZpZXcyLzIvdy84MDAvZm9ybWF0L3dlYnA.png)

首先我们来看看每个同学（节点）都有哪些属性：

- 学号（Node ID，2进制，160位）
- 手机号码（节点的IP地址及端口）

每个同学会维护以下内容：

- 从图书馆分发下来的书本（被分配到需要存储的内容），每本书当然都有书名和书本内容（内容以<key, value>对的形式存储，可以理解为文件名和文件内容）；
- 一个通讯录，包含一小部分其他同学的学号和手机号，通讯录按学号分层（一个路由表，称为“k-bucket”，按Node ID分层，记录有限个数的其他节点的ID和IP地址及端口）。

根据上面那个类比，可以看看这个表格：

![](image\aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy85NDcyMDktYWMwMzM4MTAwYTM4MGM2MS5wbmc_aW1hZ2VNb2dyMi9hdXRvLW9yaWVudC9zdHJpcHxpbWFnZVZpZXcyLzIvdy82MDEvZm9ybWF0L3dlYnA.png)

原来收藏在图书馆里，按索引号码得整整齐齐的书，以一种什么样的方式分发到同学们手里呢？大致的原则，包括：1）书本能够比较均衡地分布在同学们的手里，不会出现部分同学手里书特别多、而大部分同学连一本书都没有的情况；2）同学想找一本特定的书的时候，能够一种相对简单的索引方式找到这本书。  
Kademlia作了下面这种安排：  
假设《分布式算法》这本书的书名的hash值是 *00010000*，那么这本书就会被要求存在学号为*00010000*的同学手上。（这要求hash算法的值域与node ID的值域一致。Kademlia的Node ID是160位2进制。这里的示例对Node ID进行了简略）  
但还得考虑到会有同学缺勤。万一*00010000*今天没来上学（节点没有上线或彻底退出网络），那《分布式算法》这本书岂不是谁都拿不到了？那算法要求这本书不能只存在一个同学手上，而是被要求同时存储在学号最接近*00010000*的k位同学手上，即*00010001*、*00010010*、*00010011*…等同学手上都会有这本书。

同样地，当你需要找《分布式算法》这本书时，将书名hash一下，得到 *00010000*，这个便是索书号，你就知道该找哪（几）位同学了。剩下的问题，就是找到这（几）位同学的手机号。

![](D:\Learn\web3\note\3\image\aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy85NDcyMDktNTRkZmNiMTY1MzkyNjM4ZS5wbmc_aW1hZ2VNb2dyMi9hdXRvLW9yaWVudC9zdHJpcHxpbWFnZVZpZXcyLzIvdy82MjgvZm9ybWF0L3dlYnA.png)

##### 节点的异或距离

由于你手上只有一部分同学的通讯录，你很可能并没有*00010000*的手机号（IP地址）。那如何联系上目标同学呢？

通讯录上并没有目标同学的情况

一个可行的思路就是在你的通讯录里找到一位拥有目标同学的联系方式的同学。前面提到，每位同学手上的通讯录都是按距离分层的。算法的设计是，如果一个同学离你越近，你手上的通讯录里存有ta的手机号码的概率越大。而算法的核心的思路就可以是：当你知道目标同学Z与你之间的距离，你可以在你的通讯录上先找到一个你认为与同学Z最相近的同学B，请同学B再进一步去查找同学Z的手机号。

上文提到的距离，是学号（Node ID）之间的异或距离(XOR distance）。异或是针对yes/no或者二进制的运算.

举2个例子：  
*01010000*与*01010010*距离（即是2个ID的异或值）为*00000010*（换算为十进制即为2）；  *01000000*与*00000001*距离为*01000001*（换算为十进制即为26+1，即65）；

通讯录是如何按距离分层呢？下面的示例会告诉你，按异或距离分层，基本上可以理解为按位数分层。设想以下情景：  
以*0000110*为基础节点，如果一个节点的ID，前面所有位数都与它相同，只有最后1位不同，这样的节点只有1个——*0000111*，与基础节点的异或值为*0000001*，即距离为1；对于*0000110*而言，这样的节点归为**“k-bucket 1”**；  
如果一个节点的ID，前面所有位数相同，从倒数第2位开始不同，这样的节点只有2个：*0000101*、*0000100*，与基础节点的异或值为*0000011*和*0000010*，即距离范围为3和2；对于*0000110*而言，这样的节点归为**“k-bucket 2”**；  
……  
如果一个节点的ID，前面所有位数相同，从倒数第n位开始不同，这样的节点只有2(i-1)个，与基础节点的距离范围为[2(i-1), 2i）；对于*0000110*而言，这样的节点归为**“k-bucket i”**；

![](D:\Learn\web3\note\3\image\aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy85NDcyMDktNmJkZDZlOTZhODBkMDc4MC5wbmc_aW1hZ2VNb2dyMi9hdXRvLW9yaWVudC9zdHJpcHxpbWFnZVZpZXcyLzIvdy80MzAvZm9ybWF0L3dlYnA.png)

按位数区分k-bucket

对上面描述的另一种理解方式：如果将整个网络的节点梳理为一个按节点ID排列的二叉树，树最末端的每个叶子便是一个节点，则下图就比较直观的展现出，节点之间的距离的关系。

![](image\aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy85NDcyMDktYzEyODQ2OTAwYjA1MjVkYi5wbmc_aW1hZ2VNb2dyMi9hdXRvLW9yaWVudC9zdHJpcHxpbWFnZVZpZXcyLzIvdy85MTMvZm9ybWF0L3dlYnA.png)

k-bucket示意图：右下角的黑色实心圆，为基础节点（按wiki百科的配图修改）

回到我们的类比。每个同学只维护一部分的通讯录，这个通讯录按照距离分层（可以理解为按学号与自己的学号从第几位开始不同而分层），即k-bucket1, k-bucket 2, k-bucket 3…虽然每个k-bucket中实际存在的同学人数逐渐增多，但每个同学在它自己的每个k-bucket中只记录k位同学的手机号（k个节点的地址与端口，这里的k是一个可调节的常量参数）。  
由于学号（节点的ID）有160位，所以每个同学的通讯录中共分160层（节点共有160个k-bucket）。整个网络最多可以容纳2^160个同学（节点），但是每个同学（节点）最多只维护160 * k 行通讯录（其他节点的地址与端口）。

##### 节点定位

我们现在来阐述一个完整的索书流程。

A同学（学号*00000110*）想找《分布式算法》，A首先需要计算书名的哈希值，hash(《分布式算法》) = *00010000*。那么A就知道ta需要找到*00010000*号同学（命名为Z同学）或学号与Z邻近的同学。  
Z的学号*00010000*与自己的异或距离为 *00010110*，距离范围在[24, 25)，所以这个Z同学可能在k-bucket 5中（或者说，Z同学的学号与A同学的学号从第5位开始不同，所以Z同学可能在k-bucket 5中）。  
然后A同学看看自己的k-bucket 5有没有Z同学：

- 如果有，那就直接联系Z同学要书；
- 如果没有，在k-bucket 5里随便找一个B同学（注意任意B同学，它的学号第5位肯定与Z相同，即它与Z同学的距离会小于24，相当于比Z、A之间的距离缩短了一半以上），请求B同学在它自己的通讯录里按同样的查找方式找一下Z同学：  
  -- 如果B知道Z同学，那就把Z同学的手机号（IP Address）告诉A；  
  -- 如果B也不知道Z同学，那B按同样的搜索方法，可以在自己的通讯录里找到一个离Z更近的C同学（Z、C之间距离小于23），把C同学推荐给A；A同学请求C同学进行下一步查找。

![](D:\Learn\web3\note\3\image\aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy85NDcyMDktMTM5Njc2NWU0ZTBhZmIxMi5wbmc_aW1hZ2VNb2dyMi9hdXRvLW9yaWVudC9zdHJpcHxpbWFnZVZpZXcyLzIvdy84NjYvZm9ybWF0L3dlYnA.png)

Kademlia的这种查询机制，有点像是将一张纸不断地对折来收缩搜索范围，保证对于任意n个学生，最多只需要查询log2(n)次，即可找到获得目标同学的联系方式（即在对于任意一个有[2(n−1), 2n)个节点的网络，最多只需要n步搜索即可找到目标节点）。

![](D:\Learn\web3\note\3\image\aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy85NDcyMDktMTE0MzE2OWM4MzE4YTJmZi5wbmc_aW1hZ2VNb2dyMi9hdXRvLW9yaWVudC9zdHJpcHxpbWFnZVZpZXcyLzIvdy82NjYvZm9ybWF0L3dlYnA.png)

每次搜索都将距离至少收缩一半
