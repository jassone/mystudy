## Redis-Cell
redis4.0以后开始支持扩展模块，redis-cell是一个用rust语言编写的基于漏斗算法/令牌桶算法的的限流模块，提供`原子性`的限流功能，并允许突发流量，可以很方便的应用于分布式环境中。

### 相关wiki

* redis分布式限流之redis-cell http://researchlab.github.io/2018/10/05/redis-08-redis-cell/

