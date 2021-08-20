1. 非常重要：确保QueryRow之后调用Scan方法，否则持有的数据库链接不会被释放
```	
err := db.QueryRow("select uid,name from `user` where uid=?", 1).Scan(&u.Uid, &u.Name)
if err != nil {
	fmt.Printf("scan failed, err:%v\n", err)
	return
}
fmt.Printf("uid:%d name:%s", u.Uid, u.Name)
```

1. mysql 用完记得关闭，rows.Close()，stmt.Close()
 