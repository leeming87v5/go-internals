# Chapter 1: Function Call

```bash
$ go version
go version go1.10.1 linux/amd64
```
---

### golang汇编代码基本要素

考虑以下代码片段：
```go
// simplefunc.go
package main

//go:noinline
func simpleFunc(in int) int {
	in++
	return in
}

func main() {
	in := 0
	in = simpleFunc(in)
}
```

先来编译成汇编
```bash
go tool compile -S simplefunc.go
```

```asm
"".simpleFunc STEXT nosplit size=14 args=0x10 locals=0x0
	0x0000 00000 (simplefunc.go:5)	TEXT	"".simpleFunc(SB), NOSPLIT, $0-16
	0x0000 00000 (simplefunc.go:5)	FUNCDATA	$0, gclocals·f207267fbf96a0178e8758c6e3e0ce28(SB)
	0x0000 00000 (simplefunc.go:5)	FUNCDATA	$1, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
	0x0000 00000 (simplefunc.go:5)	MOVQ	"".in+8(SP), AX
	0x0005 00005 (simplefunc.go:6)	INCQ	AX
	0x0008 00008 (simplefunc.go:7)	MOVQ	AX, "".~r1+16(SP)
	0x000d 00013 (simplefunc.go:7)	RET
"".main STEXT size=59 args=0x0 locals=0x18
	0x0000 00000 (simplefunc.go:10)	TEXT	"".main(SB), $24-0
	;; 栈处理代码，暂不考虑
	0x000f 00015 (simplefunc.go:10)	SUBQ	$24, SP
	0x0013 00019 (simplefunc.go:10)	MOVQ	BP, 16(SP)
	0x0018 00024 (simplefunc.go:10)	LEAQ	16(SP), BP
	0x001d 00029 (simplefunc.go:10)	FUNCDATA	$0, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
	0x001d 00029 (simplefunc.go:10)	FUNCDATA	$1, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
	0x001d 00029 (simplefunc.go:12)	MOVQ	$0, (SP)
	0x0025 00037 (simplefunc.go:12)	PCDATA	$0, $0
	0x0025 00037 (simplefunc.go:12)	CALL	"".simpleFunc(SB)
	0x002a 00042 (simplefunc.go:13)	MOVQ	16(SP), BP
	0x002f 00047 (simplefunc.go:13)	ADDQ	$24, SP
	0x0033 00051 (simplefunc.go:13)	RET
	;; 栈处理代码，暂不考虑
```
#### simpleFunc要素分解

```asm
0x0000 00000 (simplefunc.go:5)	TEXT	"".simpleFunc(SB), NOSPLIT, $0-16
0x0000 00000 (simplefunc.go:5)	FUNCDATA	$0, gclocals·f207267fbf96a0178e8758c6e3e0ce28(SB)
0x0000 00000 (simplefunc.go:5)	FUNCDATA	$1, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
0x0000 00000 (simplefunc.go:5)	MOVQ	"".in+8(SP), AX
0x0005 00005 (simplefunc.go:6)	INCQ	AX
0x0008 00008 (simplefunc.go:7)	MOVQ	AX, "".~r1+16(SP)
0x000d 00013 (simplefunc.go:7)	RET
```
- `0x0000 00000`：指示代码的相对函数起始位置的偏移量，前半部分是16进制表示，后半部分是10进制表示；
- `(simplefunc.go:5)`：源代码所在文件以及行号；
- `TEXT	"".simpleFunc(SB)`：指令 `TEXT` 将 `"".simpleFunc(SB)` 符号声明在 `.text` 段，并且下面紧跟的就是函数的实现；
- `NOSPLIT`：这个是 golang 特有的指令，用来提示编译器无需为此函数处理栈增长逻辑；
这里需要解析下，golang采用了特有的“分离栈”的栈管理逻辑，可以在运行时动态地为函数分配更多的栈空间，以满足需求；
“分离栈”之所以可以动态增长，主要得益于编译器在函数的入口处加入额外的代码，用来检查本函数的栈使用量是否会超过当前栈的容量，并且依据结果动态增减栈空间；
具体的设计可以参考[这篇文档](https://docs.google.com/document/d/1wAaf1rYoM4S4gtnPh0zOlGzWtrZFQ5suE8qr2sD8uWQ/pub)。
- `$0-16`：前面的0表示本函数所需的栈帧大小，后面的16表示函数的参数所占空间（包括调用参数和返回参数）；
这里需要明确的一点是，golang函数调用的所有参数都是通过栈进行传递的，所以也就必须在栈中开辟相应的空间来存储它们；
这部分参数由函数的调用方(caller)在调用前分配，被调用方(callee)无需关心；
- `FUNCDATA`：接下来的2个`FUNCDATA`命令是与golang GC相关的，可以暂不考虑，后面的分析会直接忽略这部分处理；
- `MOVQ	"".in+8(SP), AX`：`8(SP)`表示取SP+8地址的内容，`"".in+8(SP)`表示将刚才的地址区域命名为`in`，也就是我们的入参名字；
整个命令的意思就是将`8(SP)`的内容命名为`"".in`，并且将这个值载入到`AX`寄存器中备用；
要注意，这里`8(SP)`是由函数的调用方(main)分配并且初始化的，而被调用方(simpleFunc)只需按照约定操作即可；
- `INCQ	AX`：将`AX`的值递增；
- `MOVQ	AX, "".~r1+16(SP)`：将`AX`的值赋给`"".~r1+16(SP)`，其中~r1是一个编译器临时取的名字（由于我们没有指定返回参数的名字），而16(SP)也是由调用方分配的空间；

...未完待续
