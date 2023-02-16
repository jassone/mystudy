## 单元测试

## 一、单元测试编写

### 1、准备

准备好调用所需的外部环境，如数据临时变量等。

### 2、调用

实际调用需要测试的测试方法、函数或者流程

### 3、断言

判断调用部分的返回结果是否符合预期

## 二、单元测试编写原则

- 编写快
- 相互隔离
- 一致性，幂等
- 自动化，不需要人工介入
- 及时写，及早写

## 三、单元测试遇到的一些问题

### 1、外部调用依赖问题

有时候外部接口不稳定或者很慢，可以使用mock数据。

##### a) mock库monkey

##### b) 需要单测特殊的外部存储依赖交互

比如mysql,redis,这时候可以用Docker-Compose，搭建真实的server。再配合monkey。

## 四、CI集成

- CI集成配置, 如gitlab ci。
- CI覆盖率卡点，codecov。

## 五、最佳实践

### 1、TestMain

写测试时，有时需要在测试之前或之后进行额外的**设置（setup）或拆卸（teardown）**；有时，测试还需要控制在主线程上运行的代码。为了支持这些需求，`testing` 包提供了 `TestMain` 函数 :

```go
func TestMain(m *testing.M) {
   fmt.Println("test begin")
   r := m.Run()
   fmt.Println("test end")
   os.Exit(r) // 注意这里要用m.Run()返回值
}
```

如果测试文件中包含该函数，那么生成的测试将调用 `TestMain(m)`，而不是直接运行测试。

## 六、单元测试框架选择

![20221123181002.jpg](https://pic.imgdb.cn/item/637df24a16f2c2beb122260d.jpg)



todo
https://www.topgoer.com/%E5%87%BD%E6%95%B0/%E5%8D%95%E5%85%83%E6%B5%8B%E8%AF%95.html

