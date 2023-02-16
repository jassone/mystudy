## monkey

## 一、基础知识

- 虽然 Go 是静态[编译语言](https://zh.wikipedia.org/wiki/編譯語言)，Mockey Patch 的作用域在 Runtime，但是通过 Go 的 `unsafe` 包，能够将内存中函数的地址替换为运行时函数的地址。具体的原理和实现方式参考 => [Monkey Ptching in Go](https://bou.ke/blog/monkey-patching-in-go/)。
- 因为 `unsafe`操作是不安全的，**绕过了 Go 的内存安全原则**，所以应该在测试环境中使用 Monkey Patch，并且只在需要的时候使用，确保真正需要 Mocking 的 testing 函数只使用这种方式。
- 支持数据/函数打桩

## 相关wiki

- Go monkey patch 的原理及应用【Go 夜读】https://www.bilibili.com/video/av676173977/  （todo）