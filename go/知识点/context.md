## context
todo***

![20220410212123.jpg](https://pic.imgdb.cn/item/6252d9d2239250f7c53a0250.jpg)

### 总结
* Context仅仅是一个接口定义，根据实现的不同，可以衍生出不同的context类型；
* cancelCtx实现了Context接口，通过WithCancel()创建cancelCtx实例；
* timerCtx实现了Context接口，通过WithDeadline()和WithTimeout()创建timerCtx实例；
* valueCtx实现了Context接口，通过WithValue()创建valueCtx实例；
* 三种context实例可互为父节点，从而可以组合成不同的应用形式；

## 相关wiki
* https://www.topgoer.cn/docs/gozhuanjia/chapter055.3-context
