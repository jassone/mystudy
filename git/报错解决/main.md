## main

### 1. 不能提交

```
##问题
报不能使用用户名密码的验证方式提交

##解决
git remote set-url origin git@github.com:jassone/mystudy.git 
 

具体来说：
•  git remote set-url - 修改远程仓库的 URL
•  origin - 远程仓库的名称（通常默认是 origin）
•  git@github.com:jassone/mystudy.git - 新的远程地址

改变的内容：
•  之前：https://github.com/jassone/mystudy.git (HTTPS 协议)
•  之后：git@github.com:jassone/mystudy.git (SSH 协议)

为什么要改：
•  HTTPS 方式需要每次输入用户名和个人访问令牌（token），GitHub 已不支持密码认证
•  SSH 方式使用密钥对认证，配置好后不需要每次输入凭据，更方便也更安全

你可以用 git remote -v 查看当前配置的远程地址。
```

