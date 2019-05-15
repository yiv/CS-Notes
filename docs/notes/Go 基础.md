# 数据类型

## 基本类型

## 高级类型

### struct

### array

### slice

* slice 的零值是 nil

### map

### channel

#### 已关闭 channel 

* 读

  * 读无缓存已关闭 channel，读出的值为 0
  * 读有缓存已关闭 channel， 缓存值读完后，读出的值为 0
* 写
  * 读已关闭channel，panic: send on closed channel

#### 空 channel

* 读写空 channel 会阻塞
* 关闭空 channel 会 panic

### interface

# make 和 new

* new(T)  返回 T 的指针 *T 并指向 T 的零值，返回指针类型
* make(T)  返回的是经过初始化的后T的引用，返回引用类型，只能用于 slice，map，channel

# defer

## defer 执行顺序
```go
package main

import (
	"fmt"
)

func main() {
	defer_call()
}

func defer_call() {
	defer func() { fmt.Println("打印前") }()
	defer func() { fmt.Println("打印中") }()
	defer func() { fmt.Println("打印后") }()

	panic("触发异常")
} 
```

结果

```
打印后
打印中
打印前
panic: 触发异常
```

**defer**函数属延迟执行，延迟到调用者函数执行 return 命令前被执行。多个defer之间按LIFO先进后出顺序执行

## defer 与 return

函数的`return value` 不是原子操作.而是在编译器中分解为两部分：返回值赋值 和 return 。而defer刚好被插入到末尾的return前执行。故可以在derfer函数中修改返回值

```go
package main

import (
	"fmt"
)

func main() {
	fmt.Println(doubleScore(0))    //0
	fmt.Println(doubleScore(20.0)) //40
	fmt.Println(doubleScore(50.0)) //50
}
func doubleScore(source float32) (score float32) {
	defer func() {
		if score < 1 || score >= 100 {
			//将影响返回值
			score = source
		}
	}()
	score = source * 2
	return

	//或者
	//return source * 2
}
```

# for

## for-range 中的临时变量指针不变

```go
package main

import (
	"fmt"
)

type student struct {
	Name string
	Age  int
}

func pase_student() map[string]*student {
	m := make(map[string]*student)
	stus := []student{
		{Name: "zhou", Age: 24},
		{Name: "li", Age: 23},
		{Name: "wang", Age: 22},
	}
	for _, stu := range stus {
		m[stu.Name] = &stu
	}
	return m
}
func main() {
	students := pase_student()
	for k, v := range students {
		fmt.Printf("key=%s,value=%v \n", k, v)
	}
}
```

结果

```
输出的均是相同的值：&{wang 22}
```

因为for遍历时，变量`stu`指针不变，每次遍历仅进行struct值拷贝，故`m[stu.Name]=&stu`实际上一致指向同一个指针，最终该指针的值为遍历的最后一个struct的值拷贝