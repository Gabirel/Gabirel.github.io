---
layout: post
lang: zh
title: golang 中把 <code>io.TeeReader</code> 和 <code>io.Pipe</code> 结合使用
categories: golang
tags: golang TeeReader pipe
date: 2020-08-05
---

<p class="intro"><span class="dropcap">这</span>篇文章主要解释和展示怎么在golang当中如何把 <code>io.TeeReader()</code> and <code>io.Pipe</code>结合使用。</p>

你可以轻松地根据[官方样例](https://pkg.go.dev/io?tab=doc#TeeReader)来使用`bytes.Buffer`和`io.TeeReader()`。

但是，如果你去使用`io.Pipe`的时候你会遇到一些问题。所有有了这篇文章。

**注意：这篇文章主要是面向初学者，而不是经验丰富的Golang使用者。**

-----
Table of Contents

* TOC
{:toc}

-----

------------------

# 初始修改

作为一个初学者，你可能轻松想到以下的修改方案。

```go
func main() {
   r := strings.NewReader("some io.Reader stream to be read\n")

   pr, pw := io.Pipe()
   tee := io.TeeReader(r, pw)

   // create channel to synchronize
   done := make(chan bool)
   defer close(done)

   printall := func(r io.Reader) {
      b, err := ioutil.ReadAll(r)
      if err != nil {
         log.Fatal(err)
      }

      fmt.Printf("%+v: %s", reflect.TypeOf(r), b)
   }

   defer pw.Close()
   printall(tee)
   printall(pr)
}
```

如果你运行这段代码，你会得到如下的错误信息：

```go
fatal error: all goroutines are asleep - deadlock!
```

这个错误来愿意你尝试在一个[相同的线程中去做这样的事](https://rodaine.com/2015/04/async-split-io-reader-in-golang/)，这个就会在你的脑袋上敲出一个panic。



# 修复Bug！

你可以轻松地通过一些微小的修改和增加同步来修复它。

```go
func main() {
	r := strings.NewReader("some io.Reader stream to be read\n")

	pr, pw := io.Pipe()
	tee := io.TeeReader(r, pw)

	// create channel to synchronize
	done := make(chan bool)
	defer close(done)

	printall := func(r io.Reader) {
		b, err := ioutil.ReadAll(r)
		if err != nil {
			log.Fatal(err)
		}

		fmt.Printf("%+v: %s", reflect.TypeOf(r), b)
	}
	go func() {
		printall(pr)
		done <- true
	}()

	go func() {
		defer pw.Close()

		printall(tee)
		done <- true
	}()


	// wait until both are done
	for c := 0; c < 2; c++ {
		<-done
	}
}
```

- `done` chan 被添加以此来确保同步能够正确地发生，即在main线程退出之前。
- `pw.Close()`必须要和`printall(tr)`放在另一个goroutine内部。这是因为`TeeReader`会返回一个将 r 中的数据读出后同时写入 w 的Reader中。
- 由于协程的存在，你现在可以更改两个独立的`go func(){}`的次序，不像官方例子`io.TeeReader()`提供的那样。



希望这篇文章对你有用。



<center><blockquote>The End</blockquote></center>

