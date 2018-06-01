# Chapter 1: Function Call

```bash
$ go version
go version go1.10.1 linux/amd64
```

### 1. go汇编代码基本要素

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
