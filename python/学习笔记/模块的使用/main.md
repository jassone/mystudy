## main

## 一、基础

```
导入整个模块
import os
使用：os.getcwd()

导入模块的特定方法
form os import chdir，getcwd
然后就能在代码中直接使用getcwd()了

```



## 二、包

多个模块放在一个文件夹中，该文件夹称为包

```
import 包  // 很少用，因为还需要再次定义下
form 包 import 模块

import numpy as np // 为导入的模块指定别名
form os import * // 不建议使用*导入模块所有函数

建议调用方式： 包.模块.方法
```

### 1、导入其他类库

```
from dir1.dir2.other_class import OtherClass
如果在同一个目录，则可以省略目录

自定义模块多次导入，模块中的代码只能被执行一次


```

### 2、区分在当前文件中执行还是被其他人引用时执行

```
print(__name__)  // 在当然文件中打印
当前文件中 : __main__
被其他人引用时执行：test444 // 既真实文件名
```

## 三、导入第三方包

```
pip install XXX
python -m pip install XXX
```





