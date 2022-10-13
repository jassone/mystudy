## [gorilla/mux](https://github.com/gorilla/mux) 

golang自带路由库 http.ServerMux ，实际上是一个 map[string][Handler，是请求的url路径和该url路径对于的一个处理函数的映射关系。这个实现比较简单，有一些缺点：

1. 不支持参数设定，例如/user/:uid 这种泛型类型匹配
2. 无法很友好的支持REST模式，无法限制访问方法（POST，GET等）
3. 也不支持正则

上面所指出来的glang自带路由的缺点，[gorilla/mux](https://github.com/gorilla/mux) 都具备，而且还兼容 http.ServerMux。除了支持路径正则，命名路由，还支持中间件等等功能。所以mux是一个短小精悍，功能很全的路由。

todo



https://blog.csdn.net/linzhongyilisha/article/details/112717527

https://blog.csdn.net/darjun/article/details/118948667



