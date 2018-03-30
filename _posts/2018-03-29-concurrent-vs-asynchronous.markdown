---
layout: post
title: Concurrent vs. Asynchronous
date: 2018-03-29 14:21:20 -0800
description: Defining terminology
tags: [Programming]
---

Once upon a time, most computers had one CPU containing one core. Parallelism was often an afterthought on programs and programming languages. Nowadays, it seems that everyone's computer and toaster has multiple cores. As a consequence, parallel programming has become more of a necessity rather than a nice-to-have.

When it comes to parallel programming, some people confuse asynchronism with concurrency. It's not that they don't know the difference - it's more likely that they haven't mapped the terminology to their respective concepts.

---

## Asynchronism

Consider this code: [try running it](https://play.golang.org/p/ZRSMt6RYVH-)

{% highlight go %}
import "fmt"

func main() {
    for i := 0; i < 10; i++ {
        fmt.Printf("%d ", i)
    }
    fmt.Println("Done!")
}
{% endhighlight %}

This simple program would print `0 1 2 3 4 5 6 7 8 9 Done!`. This program would execute <i>synchronously</i> because each execution is done at a time. Every `fmt.Printf()` must be finished, before printing the next number.

As a contrast, consider this code: [try running it](https://play.golang.org/p/fgaPedmFLeV)

{% highlight go %}
import "time"
import "fmt"

func main() {
    go printTheNumbers()
    time.Sleep(time.Second/10) // give some time for program to finish
    fmt.Println("Done!")
}

func printTheNumbers() {
    for i := 0; i < 10; i++ {
        fmt.Printf("%d ", i)
    }
}
{% endhighlight %}

This time, printing 0 through 9 is done on a separate thread using `go <function>`. Basically, this is the main thread saying:
> Do this separately, on your own time. I don't care about the return value of this function.

And that's the entire premise of asynchronous programming. It involves a thread calling a function without waiting for it to return, and proceeding to do other things.

This program still outputs numbers 0 through 9 consecutively. This is because the second thread<sup>1</sup> still prints the numbers in order in the for loop.

---

## Concurrency

Concurrency is when multiple things can be done at the same time.

Consider the following: [try running it](https://play.golang.org/p/9LZWSDm_uoi)

{% highlight go %}
import "time"
import "fmt"
import "math/rand"

var seed = 43
var random = rand.New(rand.NewSource(int64(seed)))

func main() {
    for i := 0; i < 10; i++ {
        go printThisNumber(i)
    }
    time.Sleep(time.Second/10) // give some time for program to finish
    fmt.Println("Done!")
}

func printThisNumber(i int) {
    // sleep for a random small amount of time
    time.Sleep(time.Duration(random.Intn(5)) * time.Millisecond)
    fmt.Printf("%d ", i)
}
{% endhighlight %}

In this example, the main program creates 10 threads. Each thread sleeps for a random amount of time, before printing its assigned number. Therefore, when you change the random seed, you change the amount of time each thread sleeps and the order of the output.

In this program, all 10 threads are doing their own work. These 10 threads might be running in parallel. I say <i>might</i> because assigning threads to processor cores are not always done on a 1:1 ratio. It's possible that all threads are assigned to one core, in which case there is no parallelism. But because this program <b>can</b> be executed out of order, it can be said that concurrency is possible.

To summarize:
1. A function call is asynchronous when it is performed without waiting for the function to return.
2. A concurrent program has multiple parts that can be executed in random order, without affecting the final result<sup>3</sup>.

---

<sup>1</sup> Technically, these are goroutines, which act <i>like</i> threads, but are not threads. For the purposes of this article, they are effectively the same.

<sup>2</sup> It can be argued that with new CPU instructions, a single core can do multiple things in a single instruction. However, for the general audience, it is simpler to describe concurrent execution as multiple cores doing things at once.

<sup>3</sup> If the sample program's purpose was to output numbers 0 through 9 in any order, then the final result is valid even if it is out of order.
