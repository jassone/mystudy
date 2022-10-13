## http/template

Golang内置`text/template`和`html/template`两个模板库，`html/template`库为HTML提供了完整的支持，包括普通变量渲染、列表渲染、对象渲染等。

**text/template是Golang标准库，实现数据驱动模板以生成文本输出，可理解为一组文本按照特定格式嵌套动态嵌入另一组文本中。可用来开发代码生成器。**

`text/template`包是数据驱动的文本输出模板，即在编写好的模板中填充数据。一般而言，模板使用流程分为三个步骤：定义模板、解析模板、数据驱动模板。

## 相关wiki

- Go语言标准库之http/template https://www.liwenzhou.com/posts/Go/go_template