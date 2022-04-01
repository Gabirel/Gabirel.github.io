---
layout: post
lang: zh
title: Golang 中 <coder>TeeReader</coder> 的 <coder>Writer</coder> 输出为空的问题小记
categories: golang
tags: golang teeReader writer
date: 2020-08-01 20:48:52
---

<p class="intro"><span class="dropcap">这</span>是一个在工作的时候遇到的 <code>TeeReader</code> 的 <code>Writer</code> 为空的问题。花了一点时间解决。并且根据官方示例代码做出一定的调整，发现了一个比较有意思的现象。但是没有什么人说到这个问题。</p>

并且，由于对 Golang 的流式处理不熟悉，途中遇到了很多的问题。

因此，来对记录一下 `TeeReader` 的解读。

-----
Table of Contents

* TOC
{:toc}

-----

# TeeReader

## TeeReader 的描述

首先放出 `TeeReader` 的源代码。代码比较简单，如下：

```go
func TeeReader(r Reader, w Writer) Reader {
	return &teeReader{r, w}
}

type teeReader struct {
	r Reader
	w Writer
}

func (t *teeReader) Read(p []byte) (n int, err error) {
	n, err = t.r.Read(p)
	if n > 0 {
		if n, err := t.w.Write(p[:n]); err != nil {
			return n, err
		}
	}
	return
}
```

官方的英文解释如下：

> TeeReader returns a Reader that writes to w what it reads from r. 
>
> All reads from r performed through it are matched with corresponding writes to w. There is no internal buffering - the write must complete before the read completes. 
>
> Any error encountered while writing is reported as a read error.

第一句话对于中文语言者可能没有那么好理解，它的意思其实非常简单：

<center><blockquote> 就是将 r 中的数据读出后同时写入 w 中，然后 `teeReader` 本身也是可以读取数据的，这样，就相当于复制出了一条数据流。</blockquote></center>

## TeeReader 的用法

官方的源代码示例如下：

```go
r := strings.NewReader("some io.Reader stream to be read\n")
var buf bytes.Buffer
tee := io.TeeReader(r, &buf)

printall := func(r io.Reader) {
	b, err := ioutil.ReadAll(r)
	if err != nil {
		log.Fatal(err)
	}

	fmt.Printf("%s", b)
}

printall(tee)
printall(&buf)
```

其输出结果可得：

```shell
some io.Reader stream to be read # <= 这个是*io.teeReader
some io.Reader stream to be read # <= 这个是*bytes.Buffer
```

## TeeReader 为什么 Writer 为空？

`TeeReader` 的参数是一个 reader 和一个 writer。但是为什么有时候 writer 会是空的？

这也是我在实际工作中遇到的一个问题。

这里，我们来把源代码示例的 `printall` 的输出顺序调整一下，如下：

```go
//修改前的
printall(tee)
printall(&buf)

//修改过后的
printall(&buf)
printall(tee)
```

这个时候输出即可看到，一个输出结果，也就是：

```go
some io.Reader stream to be read
```

这个时候如果使用 `reflect.typeOf()` 去看他们的输出类型，输出结果为：

```go
*bytes.Buffer: *io.teeReader: some io.Reader stream to be read
```

因此，可以看得到这里的 `bytes.Buffer` 输出内容为空，但是 `teeReader` 是不空的。

**那么为什么会这样呢？**

***A：*** 其实，这是由于 `teeReader` 必须要完成了过后，才会写入到 `buf` 当中。因此，提前写诗没有任何数据的。

这个也符合官方原文的英文解释：

> TeeReader returns a Reader that writes to w what it reads from r. 

希望对你有用。
