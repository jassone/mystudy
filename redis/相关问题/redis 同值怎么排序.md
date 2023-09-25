## redis 同值怎么排序

todo



当score相同时候，会按照插入的field的字典序排序，即**abc**比**acd**靠前，因为第一个字母都是a，但是b比c靠前。

