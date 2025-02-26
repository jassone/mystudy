## main

### 如何判断命令执行完毕

- **`.Run()`** 会阻塞当前 Goroutine，直到命令执行完毕。
- **`.Output()`** 同样会阻塞当前 Goroutine，直到命令执行完毕，并返回输出。

你可以通过检查返回的 `err` 值来判断命令是否执行完毕并成功。如果 `err` 为 `nil`，则表示命令成功完成。

```
package main

import (
    "fmt"
    "os/exec"
)

func main() {
    // 创建一个命令，比如 ls
    cmd := exec.Command("ls", "-l")

    // 运行命令并等待其完成
    err := cmd.Run()

    // 检查命令是否执行成功
    if err != nil {
        fmt.Printf("命令执行出错: %v\n", err)
    } else {
        fmt.Println("命令执行成功!")
    }
}
```



```
package main

import (
    "fmt"
    "os/exec"
)

func main() {
    // 创建一个命令，比如 ls
    cmd := exec.Command("ls", "-l")

    // 运行命令并获取输出
    output, err := cmd.Output()

    if err != nil {
        fmt.Printf("命令执行出错: %v\n", err)
    } else {
        fmt.Printf("命令输出: %s\n", output)
    }
}
```

