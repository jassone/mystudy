## ss

ss是Socket Statistics的缩写。顾名思义，ss命令可以用来获取socket统计信息，它可以显示和netstat类似的内容。但ss的优势在于它能够显示更多更详细的有关TCP和连接状态的信息，而且比netstat更快速更高效。

```
ss -ntlp | grep nginx //  查看某个服务的连接信息

ss -tuln | grep 20001
```



## 相关文档

-  Linux命令·ss https://blog.csdn.net/m0_64560763/article/details/130566993