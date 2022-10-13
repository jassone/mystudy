## time

### 1、正确使用time.Duration类型

当类型是time.Duration时

- 传入int类型，或者time.Duration(60)，则抛出 specified duration is 60ns, but minimal supported value is 1ms
- 正确做法：time.Minute * time.Duration(60)， 表示一分钟的纳秒数