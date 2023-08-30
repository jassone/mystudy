## 一、go tool compile

runtime.newobject表示在堆上生成了新对象

```sh
go tool compile -S main.go | grep new
        0x0020 00032 (main.go:8)        CALL    runtime.newobject(SB)
        rel 33+4 t=7 runtime.newobject+0
        
go tool trace和 GODEBUG 可查看运行时调度信息。
```

