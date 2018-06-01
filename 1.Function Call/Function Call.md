# Chapter 1: Function Call

```bash
$ go version
go version go1.10.1 linux/amd64
```

考虑以下代码片段：
```go
// simple_func.go
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
go tool compile -S simple_func.go
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
	0x0000 48 8b 44 24 08 48 ff c0 48 89 44 24 10 c3        H.D$.H..H.D$..
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
"".init STEXT size=79 args=0x0 locals=0x8
	0x0000 00000 (<autogenerated>:1)	TEXT	"".init(SB), $8-0
	;; 栈处理代码，暂不考虑
	0x000f 00015 (<autogenerated>:1)	SUBQ	$8, SP
	0x0013 00019 (<autogenerated>:1)	MOVQ	BP, (SP)
	0x0017 00023 (<autogenerated>:1)	LEAQ	(SP), BP
	0x001b 00027 (<autogenerated>:1)	FUNCDATA	$0, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
	0x001b 00027 (<autogenerated>:1)	FUNCDATA	$1, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
	0x001b 00027 (<autogenerated>:1)	MOVBLZX	"".initdone·(SB), AX
	0x0022 00034 (<autogenerated>:1)	CMPB	AL, $1
	0x0024 00036 (<autogenerated>:1)	JLS	47
	0x0026 00038 (<autogenerated>:1)	MOVQ	(SP), BP
	0x002a 00042 (<autogenerated>:1)	ADDQ	$8, SP
	0x002e 00046 (<autogenerated>:1)	RET
	0x002f 00047 (<autogenerated>:1)	JNE	56
	0x0031 00049 (<autogenerated>:1)	PCDATA	$0, $0
	0x0031 00049 (<autogenerated>:1)	CALL	runtime.throwinit(SB)
	0x0036 00054 (<autogenerated>:1)	UNDEF
	0x0038 00056 (<autogenerated>:1)	MOVB	$2, "".initdone·(SB)
	0x003f 00063 (<autogenerated>:1)	MOVQ	(SP), BP
	0x0043 00067 (<autogenerated>:1)	ADDQ	$8, SP
	0x0047 00071 (<autogenerated>:1)	RET
	;; 栈处理代码，暂不考虑

```