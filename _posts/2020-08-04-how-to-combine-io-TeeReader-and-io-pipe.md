---
layout: post
lang: en
title: How to Combine <code>io.TeeReader</code> and <code>io.Pipe</code>
categories: golang
tags: golang pipe
date: 2020-08-04
---

<p class="intro"><span class="dropcap">T</span>his post can explain and show you how to combine <code>io.TeeReader()</code> and <code>io.Pipe</code> together in Golang. You can easily use <code>bytes.Buffer</code> with <code>io.TeeReader()</code> easily according to the official documents in <a href="https://pkg.go.dev/io?tab=doc#TeeReader">here</a>.</p>

But you will notice that you may encounter some issues if you choose to use `io.Pipe` . Here I am.

**Notice: This post is for the beginners, not the well-knowledged Golang programmer.**

-----
Table of Contents

* TOC
{:toc}

-----

# Initial modification based on its example

As a beginner, you may easily come up with these kind of modifications below.

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

If you run the code, you can get error message:

```go
fatal error: all goroutines are asleep - deadlock!
```

This comes from that you are [attempting this on the same thread](https://rodaine.com/2015/04/async-split-io-reader-in-golang/), which will give you a panic hammer on your head.



# Fix the bug!

You can easily fix it via small modifications and add synchronizations.

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

- `done` chan is added to make sure synchronizations happens correctly before main thread exits.

- `pw.Close()` must be performed inside another goroutine with <code>printall(tr)</code>. This is because <code>TeeReader</code> returns a Reader that writes to w what it reads from r.

- Due to the existence of goroutine, you can change the order of two separate `go func(){}` unlike the example provided by `io.TeeReader()`

  

Hope this post does help you in some way.



<center><blockquote>The End</blockquote></center>