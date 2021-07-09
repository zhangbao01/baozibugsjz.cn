## redis应用场景

1. 比如有一个消息服务 很多业务调用rest接口来上报消息给你
2. 上报接口你可以将json格式数据放入到一个redis的list队列中(异步化)
3. 后台线程池定时的lpop取消息(优雅停机)
4. 保存消息到数据库中  存储消息的详情到redis中string类型  id:消息id  value:json格式的数据
5. 还可以存储一份数据到zset中  比如 zadd notice_id_queue_key  id 当前时间
6. set集合的作用如果一个消息已读了在set中  用来判断消息是否已读或未读
7. zset主要用来进行增量的查询  比如客户端通过时间戳来查询这个时间点之后的消息 然后你取到消息的id再批量去string类型的redis中查询详情
8. hash的场景我直接说的redisson的分布式锁

