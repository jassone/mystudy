### 启动
两步
1. HandleFunc() 注册路由，这一步需要放在最开始。
2. ListenAndServer() 开启对客户端的监控

```	
http.HandleFunc("/hello", helloWorld)
if err := http.ListenAndServe(":8081", nil); err != nil {
		log.Fatal(err)
}
```

### 路由
```	
mux.HandleFunc("/", indexHandler) // 不精准
mux.HandleFunc("/hi", hiHandler) // 精准

http://localhost/hi/ 找不到，则向上找到/
```
###  输出

```
//该请求的来源地址,在服务端打印
fmt.Println("RemoteAddr", r.RemoteAddr)

//这个写入到w的是输出到客户端的
fmt.Fprintf(w, "hello, let's go!") 
```







