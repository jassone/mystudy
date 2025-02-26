## main

### 1、前端访问报错： strict-origin-when-cross-origin

后端需要放开跨域请求，比如

```
package main
 
import (
    "github.com/gin-gonic/gin"
)
 
func main() {
    router := gin.Default()
 
    // 允许跨域请求，允许所有源
    router.Use(func(c *gin.Context) {
        c.Writer.Header().Set("Access-Control-Allow-Origin", "*")
        c.Writer.Header().Set("Access-Control-Allow-Methods", "POST, GET, OPTIONS")
        c.Writer.Header().Set("Access-Control-Allow-Headers", "Content-Type, Content-Length, Accept-Encoding, X-CSRF-Token, Authorization, accept, origin, Cache-Control, X-Requested-With")
 
        if c.Request.Method == "OPTIONS" {
            c.AbortWithStatus(200)
        }
 
        c.Next()
    })
 
    // 你的路由和处理函数
    router.GET("/someEndpoint", func(c *gin.Context) {
        // 处理请求
    })
 
    router.Run(":8080")
}
```

