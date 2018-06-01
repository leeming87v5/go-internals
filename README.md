# go-internals
---
### 目录
* [Chapter 0: Before Set-off](https://github.com/leeming87v5/go-internals/blob/master/0.Before%20set-off/0.0%20common%20tools.md)

### what for?

对于 golang 各种内部实现的一些剖解，会深入到源码级别进行详细分析；
其中会涉及到 [Plan9汇编](http://doc.cat-v.org/plan_9/4th_edition/papers/asm)，其语法与 AT&T 大致相同，但是会加入了一些特有的操作；
不熟悉的可以到 Go 官网参考下[这篇文章](https://golang.org/doc/asm)（虽然也讲的比较浅）。

### which go?
分析全部基于 [go1.10.1](https://github.com/golang/go/tree/dev.boringcrypto.go1.10)，选用的平台为 linux/amd64，其他版本和平台暂不涉及；

### others
鉴于本人水平所限，如有错漏，恳请各路大神不吝赐教，共同进步，感激不尽；
