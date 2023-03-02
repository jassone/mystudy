## go-redis

如果当前服务是只写模式，则当写入时，写入同时就抛出panic，不会返回err

```go
rdb := redis.NewClient(&redis.Options{
   Addr:     "localhost:6379",
   Password: "", // no password set
   DB:       0,  // use default DB
})

d,err := rdb.Set("name", "jack2",  time.Duration(60)).Result()
if err == redis.Nil{
} else if err != nil {
} 
```