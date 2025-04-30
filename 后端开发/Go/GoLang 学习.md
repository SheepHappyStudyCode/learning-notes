# GoLang 学习

## 泛型

使用泛型实现一个链表

```go
package main

import "fmt"

// List 表示一个可以保存任何类型的值的单链表。
// T any 表示这是一个泛型结构体，可以存储任意类型的数据
type List[T any] struct {
	next *List[T] // 指向下一个节点的指针
	val  T        // 当前节点存储的值，类型为 T
}

// NewList 创建一个新的链表节点
func NewList[T any](val T) *List[T] {
	return &List[T]{
		val:  val,
		next: nil,
	}
}

// Add 向链表末尾添加一个新节点
func (l *List[T]) Add(val T) {
	// 如果当前节点的下一个节点为空，则直接添加
	if l.next == nil {
		l.next = NewList(val)
		return
	}
	
	// 否则递归地向下一个节点添加
	l.next.Add(val)
}

// Print 打印链表中的所有元素
func (l *List[T]) Print() {
	// 打印当前节点的值
	fmt.Print(l.val, " ")
	
	// 如果还有下一个节点，继续打印
	if l.next != nil {
		l.next.Print()
	} else {
		// 打印完最后一个节点后换行
		fmt.Println()
	}
}

func main() {
	// 创建一个存储整数的链表
	intList := NewList(1)
	intList.Add(2)
	intList.Add(3)
	intList.Add(4)
	
	fmt.Println("整数链表:")
	intList.Print()
	
	// 创建一个存储字符串的链表
	strList := NewList("Go")
	strList.Add("语言")
	strList.Add("泛型")
	strList.Add("示例")
	
	fmt.Println("字符串链表:")
	strList.Print()
}
```

## Web 爬虫示例

```go
package main

import (
	"fmt"
	"sync"
)

type Fetcher interface {
	// Fetch 返回 URL 所指向页面的 body 内容，
	// 并将该页面上找到的所有 URL 放到一个切片中。
	Fetch(url string) (body string, urls []string, err error)
}

type set struct {
	m  map[string]bool
	mu sync.Mutex
}

func (p *set) add(url string) {
	p.mu.Lock()
	defer p.mu.Unlock()
	p.m[url] = true
}

func (p *set) contains(url string) bool {
	p.mu.Lock()
	defer p.mu.Unlock()
	_, ok := p.m[url]
	return ok
}

// Crawl 用 fetcher 从某个 URL 开始递归的爬取页面，直到达到最大深度。
func Crawl(url string, depth int, fetcher Fetcher, s *set, quit chan int) {
    // 用来标志本方法执行完毕
	defer func() {
		quit <- 0
	}()

	// 下面并没有实现上面两种情况：
	if depth <= 0 {
		return
	}

	if s.contains(url) {
		return
	}
	s.add(url)

	body, urls, err := fetcher.Fetch(url)
	if err != nil {
		fmt.Println(err)
		return
	}
	var chs []chan int
	fmt.Printf("found: %s %q\n", url, body)
	for _, u := range urls {
		ch := make(chan int, 1)
		go Crawl(u, depth-1, fetcher, s, ch)
		chs = append(chs, ch)
	}

	for _, ch := range chs {
		<-ch
	}
	return
}

func main() {
	s := &set{m: make(map[string]bool)}
	quit := make(chan int, 1)
	go Crawl("https://golang.org/", 4, fetcher, s, quit)
	<-quit
}

// fakeFetcher 是待填充结果的 Fetcher。
type fakeFetcher map[string]*fakeResult

type fakeResult struct {
	body string
	urls []string
}

func (f fakeFetcher) Fetch(url string) (string, []string, error) {
	if res, ok := f[url]; ok {
		return res.body, res.urls, nil
	}
	return "", nil, fmt.Errorf("not found: %s", url)
}

// fetcher 是填充后的 fakeFetcher。
var fetcher = fakeFetcher{
	"https://golang.org/": &fakeResult{
		"The Go Programming Language",
		[]string{
			"https://golang.org/pkg/",
			"https://golang.org/cmd/",
		},
	},
	"https://golang.org/pkg/": &fakeResult{
		"Packages",
		[]string{
			"https://golang.org/",
			"https://golang.org/cmd/",
			"https://golang.org/pkg/fmt/",
			"https://golang.org/pkg/os/",
		},
	},
	"https://golang.org/pkg/fmt/": &fakeResult{
		"Package fmt",
		[]string{
			"https://golang.org/",
			"https://golang.org/pkg/",
		},
	},
	"https://golang.org/pkg/os/": &fakeResult{
		"Package os",
		[]string{
			"https://golang.org/",
			"https://golang.org/pkg/",
		},
	},
}

```

