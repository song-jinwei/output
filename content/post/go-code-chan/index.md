---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Go 源码系列 chan"
subtitle: ""
summary: ""
authors: []
tags: ["Golang","源码"]
categories: []
date: 2020-07-25T23:28:35+08:00
lastmod: 2020-07-25T23:28:35+08:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

```go
type hchan struct {
	qcount   uint           // total data in the queue
	dataqsiz uint           // size of the circular queue
	buf      unsafe.Pointer // points to an array of dataqsiz elements
	elemsize uint16
	closed   uint32
	elemtype *_type // element type
	sendx    uint   // send index
	recvx    uint   // receive index
	recvq    waitq  // list of recv waiters
	sendq    waitq  // list of send waiters

	// lock protects all fields in hchan, as well as several
	// fields in sudogs blocked on this channel.
	//
	// Do not change another G's status while holding this lock
	// (in particular, do not ready a G), as this can deadlock
	// with stack shrinking.
	lock mutex
}
```

* qcount 当前缓存大小
*  dataqsiz  chan缓冲大小，也就是 make(chan int, n)，的大小
*  buf 指向缓存数组的地址
*  elemsize  chan存储元素所占大小
*  elemtype  chan存储的元素类型
*  closed     chan 是不是已经关闭
*  sendx     指向buf 的下标，就是下次接受到的数据放的数组位置
*  recvx      也是指向数组的下标，只不过是发送数据从对应下标取数据
*  recvq  双向链表，当buf中没有数据时，
*  sendq 


```go
type waitq struct {
	first *sudog
	last  *sudog
}

type sudog struct {
	// The following fields are protected by the hchan.lock of the
	// channel this sudog is blocking on. shrinkstack depends on
	// this for sudogs involved in channel ops.

	g *g

	next *sudog
	prev *sudog
	elem unsafe.Pointer // data element (may point to stack)

	// The following fields are never accessed concurrently.
	// For channels, waitlink is only accessed by g.
	// For semaphores, all fields (including the ones above)
	// are only accessed when holding a semaRoot lock.

	acquiretime int64
	releasetime int64
	ticket      uint32

	// isSelect indicates g is participating in a select, so
	// g.selectDone must be CAS'd to win the wake-up race.
	isSelect bool

	parent   *sudog // semaRoot binary tree
	waitlink *sudog // g.waiting list or semaRoot
	waittail *sudog // semaRoot
	c        *hchan // channel
}
```


###  创建
chan 具体可分为两种，无缓冲和有缓冲。举个例子来说：

无缓冲：快递员送快递，
有缓冲：

```go
// 创建无缓冲chan
ch := make(chan <- int)
// 创建有缓冲chan
// n == 0, 创建的是无缓冲chan
ch := make(chan <- int, n)
```
当我们make(chan int, n),最终调用是makechan方法

```go
func makechan(t *chantype, size int) *hchan {
	elem := t.elem
	// 这块的话主要是做一些安全检查。
	// compiler checks this but be safe.
	if elem.size >= 1<<16 {
		throw("makechan: invalid channel element type")
	}
	if hchanSize%maxAlign != 0 || elem.align > maxAlign {
		throw("makechan: bad alignment")
	}
	// 计算元素占用总内存大小
	mem, overflow := math.MulUintptr(elem.size, uintptr(size))
	if overflow || mem > maxAlloc-hchanSize || size < 0 {
		panic(plainError("makechan: size out of range"))
	}

	// Hchan does not contain pointers interesting for GC when elements stored in buf do not contain pointers.
	// buf points into the same allocation, elemtype is persistent.
	// SudoG's are referenced from their owning thread so they can't be collected.
	// TODO(dvyukov,rlh): Rethink when collector can move allocated objects.
	var c *hchan
	switch {
	case mem == 0:
		// Queue or element size is zero.
		//  例如make(chan struct, n)  空结构体不占用内存 
	      //  或者是创建的无缓冲通道，这里就不到buf
		c = (*hchan)(mallocgc(hchanSize, nil, true))
		// Race detector uses this location for synchronization.
		c.buf = c.raceaddr()
	case elem.ptrdata == 0:
		// Elements do not contain pointers.
		// Allocate hchan and buf in one call.
		// chan 里面存储的不是指针类型
		c = (*hchan)(mallocgc(hchanSize+mem, nil, true))
		c.buf = add(unsafe.Pointer(c), hchanSize)
	default:
		// Elements contain pointers.
		c = new(hchan)
		c.buf = mallocgc(mem, elem, true)
	}

	c.elemsize = uint16(elem.size)
	c.elemtype = elem
	c.dataqsiz = uint(size)
	lockInit(&c.lock, lockRankHchan)

	if debugChan {
		print("makechan: chan=", c, "; elemsize=", elem.size, "; dataqsiz=", size, "\n")
	}
	return c
}
```

```go
// 判断chan 是否已满
func full(c *hchan) bool {
	if c.dataqsiz == 0 {
		return c.recvq.first == nil
	}
	return c.qcount == c.dataqsiz
}
// 判断chan是否为空
func empty(c *hchan) bool {
	if c.dataqsiz == 0 {
		return atomic.Loadp(unsafe.Pointer(&c.sendq.first)) == nil
	}
	return atomic.Loaduint(&c.qcount) == 0
}
```

```go
// block 是不是阻塞
func chansend(c *hchan, ep unsafe.Pointer, block bool, callerpc uintptr) bool {
	if c == nil {
		if !block {
			return false
		}
		// 挂起协程
		gopark(nil, nil, waitReasonChanSendNilChan, traceEvGoStop, 2)
		throw("unreachable")
	}
	....  省略 debug 检查
	
	//  对于不阻塞的写入chan
	//  判断chan是否可以写入
	//  有缓冲时，buf是否已满
	//  无缓冲时，是否有阻塞的接受者
	if !block && c.closed == 0 && full(c) {
		return false
	}

	var t0 int64
	if blockprofilerate > 0 {
		t0 = cputicks()
	}
	// 加锁 并发安全
	lock(&c.lock)
	// 判断chan是否已经关闭，关闭直接panic
	if c.closed != 0 {
		unlock(&c.lock)
		panic(plainError("send on closed channel"))
	}
	// 直接发送给阻塞的接受者
	if sg := c.recvq.dequeue(); sg != nil {
		// Found a waiting receiver. We pass the value we want to send
		// directly to the receiver, bypassing the channel buffer (if any).
		send(c, sg, ep, func() { unlock(&c.lock) }, 3)
		return true
	}
	// 有缓冲通道，buf 未满的情况
	if c.qcount < c.dataqsiz {
		// Space is available in the channel buffer. Enqueue the element to send.
		qp := chanbuf(c, c.sendx)
		if raceenabled {
			raceacquire(qp)
			racerelease(qp)
		}
		typedmemmove(c.elemtype, qp, ep)
		c.sendx++
		if c.sendx == c.dataqsiz {
			c.sendx = 0
		}
		c.qcount++
		unlock(&c.lock)
		return true
	}

	if !block {
		unlock(&c.lock)
		return false
	}
	// 获取当前goruntinue指针
	gp := getg()
	// 创建一个sudog
	mysg := acquireSudog()
	mysg.releasetime = 0
	if t0 != 0 {
		mysg.releasetime = -1
	}
	mysg.elem = ep
	mysg.waitlink = nil
	mysg.g = gp
	mysg.isSelect = false
	mysg.c = c
	gp.waiting = mysg
	gp.param = nil
	// 放入chan sendq 双向链表
	c.sendq.enqueue(mysg)
	// 挂起协程
	gopark(chanparkcommit, unsafe.Pointer(&c.lock), waitReasonChanSend, traceEvGoBlockSend, 2)
	// Ensure the value being sent is kept alive until the
	// receiver copies it out. The sudog has a pointer to the
	// stack object, but sudogs aren't considered as roots of the
	// stack tracer.
	KeepAlive(ep)

	// 协程被唤醒
	if mysg != gp.waiting {
		throw("G waiting list is corrupted")
	}
	gp.waiting = nil
	gp.activeStackChans = false
	if gp.param == nil {
		if c.closed == 0 {
			throw("chansend: spurious wakeup")
		}
		// 唤醒后发现协程已经被关闭
		panic(plainError("send on closed channel"))
	}
	gp.param = nil
	if mysg.releasetime > 0 {
		blockevent(mysg.releasetime-t0, 2)
	}
	// 取消sulog 与 chan 的关联
	mysg.c = nil
	releaseSudog(mysg)
	return true
}
```
```go
func send(c *hchan, sg *sudog, ep unsafe.Pointer, unlockf func(), skip int) {
	.... 省略 `debug` 代码
	if sg.elem != nil {
	      // 内存拷贝
		sendDirect(c.elemtype, sg, ep)
		sg.elem = nil
	}
	// 获取 sulog 上绑定的 goruntinue
	gp := sg.g
	unlockf()
	gp.param = unsafe.Pointer(sg)
	if sg.releasetime != 0 {
		sg.releasetime = cputicks()
	}
	// 唤醒 sulog 上的 goruntinue, 
	// 数据已经 copy 过去，唤醒其处理后续逻辑
	goready(gp, skip+1)
}
```

```go
// chanrecv在通道c上接收并将接收到的数据写入ep。
// ep可能为nil，在这种情况下，接收到的数据将被忽略。
// 如果block == false并且没有可用元素，则返回（false，false）。
// 否则，如果c关闭，则* ep为零并返回（true，false）。
// 否则，用一个元素填充* ep并返回（true，true）。
// 非nil必须指向堆或调用者的堆栈。
func chanrecv(c *hchan, ep unsafe.Pointer, block bool) (selected, received bool) {
	if c == nil {
		if !block {
			return
		}
		gopark(nil, nil, waitReasonChanReceiveNilChan, traceEvGoStop, 2)
		throw("unreachable")
	}

	if !block && empty(c) {
		if atomic.Load(&c.closed) == 0 {
			return
		}
		if empty(c) {
			if raceenabled {
				raceacquire(c.raceaddr())
			}
			if ep != nil {
				typedmemclr(c.elemtype, ep)
			}
			return true, false
		}
	}

	var t0 int64
	if blockprofilerate > 0 {
		t0 = cputicks()
	}
	// 加锁 并发安全
	lock(&c.lock)
	// chan 已经关闭 并且buf 为空
	//   无缓冲通道关闭 或者是 有缓冲通道，但是buf为空 
	if c.closed != 0 && c.qcount == 0 {
		.... 省略 `debug` 代码
		unlock(&c.lock)
		if ep != nil {
			// 设置ep 为对应类型的零值
			typedmemclr(c.elemtype, ep)
		}
		return true, false
	}
	
	if sg := c.sendq.dequeue(); sg != nil {
		recv(c, sg, ep, func() { unlock(&c.lock) }, 3)
		return true, true
	}
	// 缓冲型通道  正常接收
	if c.qcount > 0 {
		// Receive directly from queue
		qp := chanbuf(c, c.recvx)
		if ep != nil {
			typedmemmove(c.elemtype, ep, qp)
		}
		typedmemclr(c.elemtype, qp)
		c.recvx++
		if c.recvx == c.dataqsiz {
			c.recvx = 0
		}
		c.qcount--
		unlock(&c.lock)
		return true, true
	}

	if !block {
		unlock(&c.lock)
		return false, false
	}

	// chan 阻塞接收 
	gp := getg()
	// 创建sulog 放入到 `recvq` 队列中，等待唤醒
	mysg := acquireSudog()
	mysg.releasetime = 0
	if t0 != 0 {
		mysg.releasetime = -1
	}
	mysg.elem = ep
	mysg.waitlink = nil
	gp.waiting = mysg
	mysg.g = gp
	mysg.isSelect = false
	mysg.c = c
	gp.param = nil
	c.recvq.enqueue(mysg)
	gopark(chanparkcommit, unsafe.Pointer(&c.lock), waitReasonChanReceive, traceEvGoBlockRecv, 2)

	// 唤醒之后 处理后续逻辑
	//  有两种唤醒方式，1.收到数据，2. chan被关闭
	if mysg != gp.waiting {
		throw("G waiting list is corrupted")
	}
	gp.waiting = nil
	gp.activeStackChans = false
	if mysg.releasetime > 0 {
		blockevent(mysg.releasetime-t0, 2)
	}
	closed := gp.param == nil
	gp.param = nil
	mysg.c = nil
	releaseSudog(mysg)
	return true, !closed
}
```

```go
func recv(c *hchan, sg *sudog, ep unsafe.Pointer, unlockf func(), skip int) {
	if c.dataqsiz == 0 {
		if ep != nil {
			// 如果是无缓冲，直接从 sender 复制数据
			recvDirect(c.elemtype, sg, ep)
		}
	} else {
		//buf 是满的，那么从buf 中接收游标处取元素，并且从阻塞的发送者队列中第一个元素放入buf
		qp := chanbuf(c, c.recvx)
		// 从 buf 队列中取元素给 ep
		if ep != nil {
			typedmemmove(c.elemtype, ep, qp)
		}
		// 从阻塞的发送者队列中取数据放入buf
		typedmemmove(c.elemtype, qp, sg.elem)
		c.recvx++
		if c.recvx == c.dataqsiz {
			c.recvx = 0
		}
		c.sendx = c.recvx // c.sendx = (c.sendx+1) % c.dataqsiz
	}
	sg.elem = nil
	gp := sg.g
	unlockf()
	gp.param = unsafe.Pointer(sg)
	if sg.releasetime != 0 {
		sg.releasetime = cputicks()
	}
	唤醒发送的 goruntinue
	goready(gp, skip+1)
}
```
```go
func closechan(c *hchan) {
    // var ch chan int
    // close(ch)
	if c == nil {
		panic(plainError("close of nil channel"))
	}

	lock(&c.lock)
	if c.closed != 0 {
		unlock(&c.lock)
		// 已经关闭的 chan 不能重复关闭
		panic(plainError("close of closed channel"))
	}
	// 设置关闭参数
	c.closed = 1
	// 带唤醒的协程队列
	var glist gList
	//释放所有的 recvq
	for {
		sg := c.recvq.dequeue()
		if sg == nil {
			break
		}
		if sg.elem != nil {
			typedmemclr(c.elemtype, sg.elem)
			sg.elem = nil
		}
		if sg.releasetime != 0 {
			sg.releasetime = cputicks()
		}
		gp := sg.g
		gp.param = nil
		// 入队列
		glist.push(gp)
	}

	// 释放所有的 sendq， 将会panic
	for {
		sg := c.sendq.dequeue()
		if sg == nil {
			break
		}
		sg.elem = nil
		if sg.releasetime != 0 {
			sg.releasetime = cputicks()
		}
		gp := sg.g
		gp.param = nil
		glist.push(gp)
	}
	unlock(&c.lock)
	// 遍历协程队列 从队尾依次唤醒
	for !glist.empty() {
		gp := glist.pop()
		gp.schedlink = 0
		goready(gp, 3)
	}
}
```