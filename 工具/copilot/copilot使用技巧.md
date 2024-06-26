## copilot使用技巧

## 一、copilot的特性

### 1、单一职责

它最擅长干单一简单的事情，而不是一个复杂的事情。

### 2、具体的提示词

提示词越具体，越有助于copilot理解编程者的意图

- 计算机只能做你告诉他们做的事
- 算法只是一系列指令(既提示词)

### 3、短响应

它不善于返回大段的响应

## 二、基础概念

### 1、哪些可以提供提示词/上下文(任何东西都可以作为上下文)

- 打开的文件：基于当前文件和打开的文件

  在编辑器中打开相关的文件，关闭无关联的文件。copilot可以读取当前的文件以及在 IDE 中打开的任何文件

- import 语句：适当的代码引入

  比如go中 import 语句

- 粘贴来的外部文档

- 有意义的函数/参数名称/变量命名: 命名需要有有意义，对描述其用途的变量和函数使用一致的、特定的命名约定，使用具体的带有含义名称，比如变量名+s/List(表示数组), getXX等。比如

  ```
  func GetUserName(userId int) (userName string) {
  	...
  }
  ```

- 数据类型: 比如确定的数据类型，而不是go中interface类型

- 各种注释：具体且范围明确的功能注释，比如

  ```
  /****好的注释****/ 
  // item.rightAnswer是一个字符串，将其按字符切分转换为[]string，并赋值给question.Answers
  for _, v := range item.RightAnswer {
    question.Answers = append(question.Answers, string(v))
  }
  
  /****不好的注释****/ 
  // 根据二级科目id获取教材列表，并获取每个教材下所关联的题目数量
  // 上面的注释不具体，且范围不明确，应该提供更多的信息，或者将上述注释拆分为二个(既两个动作)
  ```

  文件顶部增加说明，有助于理解背景，比如

  ```
  // Package papers
  // Description: 试卷模型
  package papers
  ...
  ```

- 示例代码：用示例代码让copilot做好准备 ，可以先贴一段代码过来，生成好代码后再删除示例代码。比如

  ```
  /***********以下代码为从其他文件中copy过来的，为示例代码*************/
  // InsertBatch	批量插入
  func (m *LearningQuestions) InsertBatch(l []LearningQuestions) (err error) {
  	db := database.DB.Model(m).Create(&l)
  	return db.Error
  }
  
  /***********以下代码为先输入注释，由copilot生成的*************/
  // InsertBatch	批量插入
  func (m *LearningPapers) InsertBatch(l []LearningPapers) (err error) {
  	db := database.DB.Model(m).Create(&l)
  	return db.Error
  }
  ```

### 2、需要可预测性

- 机构化的软件架构

  比如mvc模式

- 描述文件用途及其与项目其余部分的关系的高级文件注释、函数级注释、类级注释和行注释

  最有帮助的注释是那些能够澄清代码中潜在不明确方面的注释

## 三、最佳实践

### 1、你还是你

Copilot 的中文翻译是副驾驶，所以它只能作为辅助工具来用，你才是主驾驶。所以在编程中，你之前是怎么写代码的现在还是怎么写代码，只需要在用Copilot来写代码时，就发送指令(既写注释)给Copilot。

千万不要把副驾驶当主驾驶。

### 2、你的要求要简单而具体

copilot 需要具体的分批次的步骤说明，他们更擅长一步一步接受指令。既**让 copilot 在生成一段代码后再给他下一个指令**，而不是用一大堆指令要求它一次性生成一堆代码。

### 3、信任它，但需要再次验证

Copilot 生成的代码可能有错误，需要我们再次验证

### 4、提示词可以是任何东西

你描述问题的质量决定了AI给出答案的质量

### 5、不停迭代

重试！对于你和 Copilot 来说，这是一个学习过程。你使用它的次数越多，你与 GitHub Copilot 的沟通就越顺畅。

### 6、让它帮你生成提示词

输入部分提示词，后续可以用补全方式丰富提示词

### 7、提前为copilot做好一切准备

比如提前手动导入需要的包，提前定义好数据结构，提取把最基础的函数方法(比如数据库表查询函数)写出来等，这样Copilot 才能有一个整体的认知，然后将各个函数方法组织起来。

既先从最小部分/最基础部分开始，定义好一切。这点会和我们之前的编程习惯有点相反，我们之前可能会直接写函数，然后再定义函数中用到的数据类型；或者先写controller层，后写model层。

### 8、尝试

先不写注释，看看还行，不行再写注释

### 9、Copilot 不好使了怎么办

如果Copilot 给出的代码有问题，这里分两种情况

- 部分能用

  - 自己手动修改下
  - 增加提示词，让Copilot 来修复

- 完全不能用，则不采纳，继续自己手动写代码

  有时候当你写了一个关键字(比如 for )或者一行代码后，Copilot 才能真正理解你的意图，并给出正确的代码
