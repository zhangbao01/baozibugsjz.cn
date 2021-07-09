## 字符串

![image-20200817080405387](https://note.youdao.com/yws/api/personal/file/WEBe8600f0f5873c43dd3fec45672fc759b?method=download&shareKey=ce9e8cd0669b45f0940a12e658ccef3f)

1. c原生字符串
   - 获取长度时间复杂度O(n)
   - 缓冲区溢出:strcat(a,"Cluster")覆盖a元素
   - 字符串截取:字符串截取需要重新分配同时回收旧空间(否则内存泄漏)
   - 不能存储二进制:需要通过"\0"知道字符串是否结束
2. 应用场景
   - 比如通知的详情存储 key:id value:详细的json格式数据



## 链表

![image-20200101111300452](https://note.youdao.com/yws/public/resource/4762addbbb207565dafe6a1264ea04a1/xmlnote/8BEF876A201741DB9A871C36764653D8/8988)

1. 多态:
   - dup:复制链表节点所保存的值
   - free:释放节点所保存的值
   - match:对比链表节点所保存的值和另外一个输入的值是否相等
2. 场景
   - 异步化处理:比如消息上报时放入到异步队列中
   - 后台工作线程消费队列中的数据



## 字典

![image-20200101115037847](https://note.youdao.com/yws/public/resource/4762addbbb207565dafe6a1264ea04a1/xmlnote/2C24DE4DD6F34E8D8821F04A7D95CA46/8995)

1. 字典(dict)
   - type:类型特定函数
   - privdata:私有数据
   - ht[2]:哈希表  ht[0]:存储数据  当需要rehash时使用ht[1]:当rehash完成之后再交换0和1
   - rehashidx:rehash索引,不进行rehash是为-1
2. 哈希表(dictht)
   - table:哈希表数组
   - size:数组长度
   - sizemask:数组长度掩码=数组长度-1
   - used:哈表表已有节点的数量
3. 哈希表节点
   - key:键
   - value:值
   - next:下一个dictEntry构成链表结构
4. 哈希算法
   - hash=dict->type->hashFunction(key) 通过hash函数计算key对应的hash值
   - index=hash&dict->hx[x].sizemask 同构hash值得到字典表的数组下标为hx[0]或者hx[1],在求hash表数组对应的index 如果index下存储dictEntry字典键值对则添加到链表头部
5. 场景
   - 比如redisson的分布式锁
   - 比如记录用户每天签到的礼品信息 entry的key:时间 value:次数



## 跳表

![image-20200103195654345](https://note.youdao.com/yws/public/resource/4762addbbb207565dafe6a1264ea04a1/xmlnote/0D45891126414D11B5303011FB133B17/8997)

1. zskiplist
   - 一个跳跃表由多个跳跃表节点组成
   - header:跳跃表的表头节点
   - tail:指向跳跃表的表尾节点
   - level:记录目前跳跃表内,层数最大的那个节点的层数比如tail的层数为L5
   - length:记录目前跳跃表的长度,目前3个跳跃表节点则length=3
2. zskiplistNode
   - Level[] :标记L1 L2 L3 L4标记节点的各个层
   - BW:backward 后退指针,通过tail遍历bw为空为止即为header
   - 前进指针:level[i].forward 从表头向表尾方向访问节点
   - score:各个跳跃表中1.0 2.0 3.0,即通过查找时层的跨度,比如从L5层通过前驱指正找到尾节点跨度为3
   - obj:成员变量,存储是一个指针,指向一个字符串对象,而字符串对象保存的是一个SDS值
3. 注意事项
   - 当分数相同时,排序按照字典的顺序排列,成员变量较小的排在前面
4. 场景
   - 延时队列:当队列中消费失败后放入到延时队列中
   - 分页排序:比如拉取消息时根据上一次的消息id来进行拉取
   - 时间磋增量查询:根据时间磋来判断是否有新的消息



## 整数集合

1. 整数集合(intset)当集合中底层存储数据都是整数类型切无重复元素时使用

   ```redis
   sadd numbers 1 100 1001 111111
   ```

2. 数据存储分为三部分

   - encodeing:编码方式,每个元素的编码方式与其相同,例如:intset_enc_int16  32 64 对应的大小范围
   - length:集合中元素的个数
   - contents[]:存储元素的数组,元素存储按从小到大存储

3. 升级

   - 当元素添加的值超过了旧的范围时进行升级,升级步骤
   - 根据新元素的类型,扩展整数集合数组的空间大小,比如添加65535是对应的类型为int32_t,32*4=128 之前存储3个元素为int16_t  3*16=48 所以分配的空间为 48-127
   - 将旧的元素的类型转为新的元素的类型,同时保证下标顺序一致 如图中3保持下标为2位置变为(64,95),同理1和2
   - 最后将添加的元素放入到数组的尾部,同时length+1

4. 问题

   - 为啥所有的元素不都存储int64_t,因为数据较小是会浪费内存的空间 需要合理的分配
   - 不支持降级操作
   
5. 场景

   - 比如消息已读通过set集合来存储
   - 每次判断一个消息是否已读时进行sismember操作



## 压缩列表

![image-20200107064019335](https://note.youdao.com/yws/public/resource/0913f7b1edd0a739cddfc0fb16cfac5b/xmlnote/9C1E98609513422C9EA33F4C51F7CC96/9043)

1. zlbytes:指压缩列表占的内存字节数
2. zltail:代表列表的尾指针到头指针的长度,通过头指针(zlbytes)+zltail得到尾指针的位置
3. zlend:压缩列表的结尾特殊值:255
4. Entry1....entry3:压缩列表的节点元素,节点分为三部分
   - previous_entry_length:如果前一个节点长度小于254字节,则通过1个字节存储前一个节点的长度,如果大于254个字节,通过5个字节来存储长度,其中第一个为0XFE(254),后4个字节为前一个节点的长度,这样计算前一个节点的地址 preEntry = currentEntry-previous_entry_length
   - encoding:当前节点元素使用的编码
   - content:存储节点的内容
5. 连续更新问题
   - 比如节点e1-eN对应的节点字节数都在250-253个字节,如果新加入的节点大于254个字节,此时会加入到头部节点的位置;当时由于e1存储的previous_entry_length为1则存储不下需要通过5个字节存储同理存储5个节点后e2存储不下e1依次类推到eN节点,每次分配的最坏时间复杂度为O(N)  N次则O(N^2)

![image-20200107064859834](https://note.youdao.com/yws/public/resource/0913f7b1edd0a739cddfc0fb16cfac5b/xmlnote/B3AFABBDA7274147ADBCD422AF241B61/9083)

