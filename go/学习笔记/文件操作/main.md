## 文件操作

### 1、细节
* 打开的文件一定要手动关闭，系统不会自动关闭。

### 2、判断文件是否存在

```go
func fileExists(fileName string) (bool, error) {
  _, err := os.Stat(fileName)
  if err == nil {
    return true, nil
  }
  if os.IsNotExist(err) { // 返回一个布尔值说明该错误是否表示一个文件或目录不存在。
    return false, nil
  }

  return false, err
  //这样最靠谱，因为发送错误时，不一定是文件不存在的错误，这里需要判断下错误类型
  //也可以用IsExist()
}
```