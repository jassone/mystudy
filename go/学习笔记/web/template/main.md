创建模板并将字符解析进去 Must用于检测模板是否有问题，有问题panic

```	
t := template.Must(template.New("range").Parse(rangeTemplate))
```
