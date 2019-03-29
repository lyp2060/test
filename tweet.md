#this is to add some good SDI docs

1. tweet design, from https://www.1point3acres.com/bbs/thread-498444-1-1.html
用户可以follow其它用户。被follow的用户如果发贴，则要按照时序出现在foller的timeline里。很明显这是一个写频繁（考虑到帖子字数要求140以下），读更频繁的分布式系统。

考虑到很多的小的写操作，一开始我的思路是纯粹按照用户来sharding。但是看过参考资料才知道这是不可取的。因为会有两个因素导致冷热不均：水车型用户（大量的写和存储）以及网红型用户（大量的读操作）

否决掉这个后，自然想到的就是按照unique id来sharding。

unique id的构成由
userid前缀
发帖时间戳，

组成。不特别把所有某个用户的帖子全部集中于一个sharding。可能是存储于连续的若干shardings里。

我这个版本，不同于参考资料的，有几个好处：
写操作可以分布到用户区间涵盖该用户的服务器上，插入。
浏览他人的tweets，往往要倒序查阅所有的该用户的帖子。那么因为我这个unique id是以用户id为前缀的。所以可以很快获取所有的该用户的帖子。
帖子和用户的关系也就不必再写入关系型数据库了。查找某个用户的帖子可以query（比如：用户id+周一，用户id周三）。


cache可以用来存储高频阅读的帖子，比如网红贴。

对于timeline的生成，
当用户发帖的时候，unique id就会生成，那么一方面写入存储媒体，一方面把帖子的id写入follow我的用户的timeline中。
我认为应该区分网红（follower超过某个数）和非网红。非网红的可以使用fan-out-on-write。而网红的贴采用fan-out-on-read。在用户读取的时候做aggregation处理。

retweet也是帖子。只不过其中的body是个ref-id的chain。比如A retweet B, but B was retweeting from C, C was retweeting from D. 这样读完这个ref-id的chain后可以deduce出ABCD的用户名，另外只要去取帖子D的内容就可以了。

estimation没有做，明白了以上的设计，加上一些常数和估计数字，estimation应该没有问题。
